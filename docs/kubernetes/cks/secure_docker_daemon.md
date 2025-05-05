# Secure Docker Daemon

The Docker daemon is a critical component that manages and coordinates all Docker operations. Since it often runs with elevated privileges, securing it properly is essential to protect your containerized applications and host systems from potential security threats.

## Understanding Docker Daemon Security Risks

Before implementing security measures, it's important to understand the potential risks:

1. **Privilege Escalation**: The Docker daemon typically runs with root privileges, making it a prime target for attackers seeking to gain elevated access to your system.

2. **Unauthorized Access**: An unsecured Docker daemon can be accessed by unauthorized users, potentially leading to data breaches or system compromise.

3. **Container Breakouts**: Vulnerabilities in the Docker daemon can lead to container breakouts, where attackers escape container isolation and access the host system.

4. **API Exposure**: The Docker API, if exposed unsafely, can be exploited for unauthorized operations.

## Configuring and Securing Communication Sockets

Docker daemon can communicate through different types of sockets. Understanding and properly configuring these communication channels is essential for security.

### 1. Unix Socket (Default and Recommended)

By default, Docker runs through a non-networked UNIX socket. It can also optionally communicate using SSH or a TLS (HTTPS) socket. The Unix socket is located at `/var/run/docker.sock` and is the most secure option for local communications.

```bash
# The default configuration in daemon.json (often doesn't need to be specified)
{
  "hosts": ["unix:///var/run/docker.sock"]
}
```

Unix sockets are more secure than TCP sockets because:
- They're not exposed to the network
- They use standard Unix file permissions for access control
- They're not prone to cross-site request forgery attacks that can happen with TCP sockets

### 2. TCP Socket (For Remote Access)

If you need remote access to the Docker daemon, you can configure it to listen on a TCP socket, but this should always be protected with TLS.

When using a TCP socket, the Docker daemon provides un-encrypted and un-authenticated direct access to the Docker daemon by default. You should secure the daemon either using the built in HTTPS encrypted socket, or by putting a secure web proxy in front of it.

```bash
# Example daemon.json with both Unix socket and secure TCP socket
{
  "hosts": [
    "unix:///var/run/docker.sock",
    "tcp://0.0.0.0:2376"
  ],
  "tls": true,
  "tlscacert": "/path/to/ca.pem",
  "tlscert": "/path/to/server-cert.pem",
  "tlskey": "/path/to/server-key.pem",
  "tlsverify": true
}
```

**Important**: When using the TCP socket:
- **Never** expose the Docker daemon to the internet without TLS encryption
- Docker over TLS should run on TCP port 2376 (not 2375, which is unencrypted)
- Use client certificate authentication
- Implement proper network firewall rules

### 3. Setting Up TLS for Docker Daemon

To set up TLS for secure remote access:

1. **Generate CA, server, and client certificates**

```bash
# Create a CA key and certificate
openssl genrsa -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem

# Create a server key and certificate signing request (CSR)
# Set $HOST to the DNS name or IP of your Docker host
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr

# Sign the server certificate
# Include IP addresses and DNS names that will be used to connect to your Docker host
echo "subjectAltName = DNS:$HOST,IP:127.0.0.1,IP:$PUBLIC_IP" >> extfile.cnf
echo "extendedKeyUsage = serverAuth" >> extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -extfile extfile.cnf

# Create a client key and certificate signing request
openssl genrsa -out client-key.pem 4096
openssl req -subj '/CN=client' -new -key client-key.pem -out client.csr

# Sign the client certificate
echo "extendedKeyUsage = clientAuth" > extfile-client.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out client-cert.pem -extfile extfile-client.cnf
```

2. **Configure the Docker daemon to use TLS**

   Update the `/etc/docker/daemon.json` file:

```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/path/to/ca.pem",
  "tlscert": "/path/to/server-cert.pem",
  "tlskey": "/path/to/server-key.pem"
}
```

3. **Configure your client to use TLS**

Replace all instances of $HOST in the following example with the DNS name of your Docker daemon's host.

```bash
# When connecting, specify the TLS certificates
docker --tlsverify \
  --tlscacert=ca.pem \
  --tlscert=client-cert.pem \
  --tlskey=client-key.pem \
  -H=$HOST:2376 version

# Or set environment variables
export DOCKER_HOST=tcp://$HOST:2376
export DOCKER_TLS_VERIFY=1
export DOCKER_CERT_PATH=/path/to/client/certificates
```

### 4. Using systemd Configuration (Alternative Method)

If you're using systemd, you can also configure the Docker daemon through the service file:

```bash
# Create a systemd override file
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/override.conf

# Edit the override file
sudo nano /etc/systemd/system/docker.service.d/override.conf
```

Add the following content to the `override.conf` file:

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/server-cert.pem --tlskey=/path/to/server-key.pem
```

Reload systemd and restart Docker:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 5. Verifying Socket Configuration

After configuring your Docker daemon, verify that it's listening on the specified sockets:

```bash
# Check for Unix socket
ls -la /var/run/docker.sock

# Check for TCP socket
sudo netstat -tulpn | grep dockerd
```

### 6. Listening on Both Unix and TCP Sockets

When configuring Docker to listen on both types of sockets, be aware that the "hosts" option overrides the Docker default value rather than appends to it. You must specify both the Unix socket and TCP socket in your configuration.

```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"]
}
```

You can also verify the configuration is working with:

```bash
sudo netstat -lntp | grep dockerd
```

If properly configured, you should see Docker listening on both the Unix socket and the specified TCP port.

### 3. Implement Rootless Mode

Rootless mode allows running the Docker daemon and containers as a non-root user to mitigate potential vulnerabilities in the daemon and the container runtime.

Rootless mode ensures that the Docker daemon and containers are running as an unprivileged user, which means that even if an attacker breaks out of the container, they will not have root privileges on the host, which in turn substantially limits the attack surface.

```bash
# Set up rootless mode
dockerd-rootless-setuptool.sh install
```

### 4. Secure the Docker Socket

Docker communicates with a UNIX domain socket called /var/run/docker.sock. This is the main entry point for the Docker API. Anyone who has access to the Docker daemon socket also has unrestricted root access.

Best practices for socket security:

1. Never expose the Docker socket to containers unless absolutely necessary
2. Use proper Unix file permissions to restrict access
3. If socket sharing is required, consider using a proxy like Docker Socket Proxy

```bash
# Set proper permissions for the Docker socket
sudo chmod 660 /var/run/docker.sock
sudo chown root:docker /var/run/docker.sock
```

### 5. Configure User Namespace Remapping

User namespace remapping is a Docker feature that converts host UIDs to a different unprivileged range inside your containers. This helps to prevent privilege escalation attacks, where a process running in a container gains the same privileges as its UID has on your host.

```bash
# Configure user namespace remapping in daemon.json
{
  "userns-remap": "default"
}
```

### 6. Restrict Inter-Container Communication

Docker normally allows arbitrary communication between the containers running on your host. Each new container is automatically added to the docker0 bridge network, which allows it to discover and contact its peers. Keeping inter-container communication (ICC) enabled is risky because it could permit a malicious process to launch an attack against neighboring containers.

```bash
# Disable inter-container communication in daemon.json
{
  "icc": false
}
```

### 7. Implement Resource Limitations

Set appropriate resource constraints to prevent denial-of-service attacks:

```bash
# Configure default resource constraints in daemon.json
{
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  },
  "default-shm-size": "64M",
  "default-memory-swap": "-1",
  "default-memory": "1G",
  "default-cpu-shares": 1024
}
```

### 8. Enable Content Trust

Docker Engine can be configured to run only signed images, enhancing security through image signature verification. This feature, set up in the Docker configuration file (daemon.json), gives you control over enforcing security policies related to image usage.

```bash
# Enable Docker Content Trust in daemon.json
{
  "content-trust": {
    "trust-pinning": {
      "official-library-images": true
    }
  }
}
```

### 9. Enable Logging and Monitoring

Collecting Docker daemon logs is instrumental in identifying and responding to security incidents. These logs offer comprehensive insights into Docker's system-level operations, encompassing critical aspects such as container lifecycle events, network configurations, image management, and incoming API requests.

```bash
# Configure logging in daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "log-level": "info"
}
```

### 10. Use Security Enhanced Systems

Ensuring OS-level security systems are active helps defend against malicious activity originating inside containers and the Docker daemon. Docker supports policies for SELinux, Seccomp, and AppArmor; keeping them enabled ensures sane defaults are applied to your containers, including restrictions for dangerous system calls.

```bash
# Don't disable security-enhanced systems
# Incorrect configuration (do not use):
# {
#   "selinux-enabled": false,
#   "apparmor-enabled": false
# }

# Correct configuration:
{
  "selinux-enabled": true,
  "apparmor-enabled": true
}
```

## Comprehensive `/etc/docker/daemon.json` Example

Here's a comprehensive example of a secure `daemon.json` configuration that incorporates many of the best practices:

```json
{
  "icc": false,
  "log-level": "info",
  "iptables": true,
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true,
  "userns-remap": "default",
  "storage-driver": "overlay2",
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  },
  "selinux-enabled": true,
  "apparmor-enabled": true,
  "default-runtime": "runc",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "content-trust": {
    "trust-pinning": {
      "official-library-images": true
    }
  }
}
```

