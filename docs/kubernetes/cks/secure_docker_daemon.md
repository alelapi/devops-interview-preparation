# Securing Docker Daemon

The Docker daemon is a critical component that manages and coordinates all Docker operations. Since it often runs with elevated privileges, securing it properly is essential to protect your containerized applications and host systems from potential security threats.

## Understanding Docker Daemon Security Risks

Before implementing security measures, it's important to understand the potential risks:

1. **Privilege Escalation**: The Docker daemon typically runs with root privileges, making it a prime target for attackers seeking to gain elevated access to your system.

2. **Unauthorized Access**: An unsecured Docker daemon can be accessed by unauthorized users, potentially leading to data breaches or system compromise.

3. **Container Breakouts**: Vulnerabilities in the Docker daemon can lead to container breakouts, where attackers escape container isolation and access the host system.

4. **API Exposure**: The Docker API, if exposed unsafely, can be exploited for unauthorized operations.

## Essential Security Practices for Docker Daemon

### 1. Use Secure Communication

The REST API endpoint changed in Docker 0.5.2, and now uses a Unix socket instead of a TCP socket bound on 127.0.0.1. You can then use traditional Unix permission checks to limit access to the control socket.

If you need remote access to the Docker daemon:

1. Configure TLS encryption for the Docker daemon API
2. Use client certificate authentication
3. Implement proper access controls

```bash
# Example daemon.json configuration for TLS
{
  "tls": true,
  "tlscacert": "/path/to/ca.pem",
  "tlscert": "/path/to/server-cert.pem",
  "tlskey": "/path/to/server-key.pem",
  "tlsverify": true
}
```

### 2. Secure the Docker Socket

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

### 3. Configure User Namespace Remapping

User namespace remapping is a Docker feature that converts host UIDs to a different unprivileged range inside your containers. This helps to prevent privilege escalation attacks, where a process running in a container gains the same privileges as its UID has on your host.

```bash
# Configure user namespace remapping in daemon.json
{
  "userns-remap": "default"
}
```

### 4. Restrict Inter-Container Communication

Docker normally allows arbitrary communication between the containers running on your host. Each new container is automatically added to the docker0 bridge network, which allows it to discover and contact its peers. Keeping inter-container communication (ICC) enabled is risky because it could permit a malicious process to launch an attack against neighboring containers.

```bash
# Disable inter-container communication in daemon.json
{
  "icc": false
}
```

### 5. Implement Resource Limitations

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

### 6. Enable Content Trust

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

### 7. Enable Logging and Monitoring

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

### 8. Use Security Enhanced Systems

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


