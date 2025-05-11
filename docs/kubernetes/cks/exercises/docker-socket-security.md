# Editing Docker Socket Service File to Change Group Ownership to Root

## Problem Statement

Edit the docker.socket service file to make the group owned by root instead of the default docker group.

## Solution

The Docker socket file (`docker.socket`) defines permissions for the Docker daemon socket. By default, this socket is owned by root:docker, which allows any user in the docker group to interact with the Docker daemon with essentially root privileges. Changing the group ownership to root restricts Docker access to only the root user, enhancing security.

### Step 1: Locate the docker.socket Service File

First, let's find the exact location of the docker.socket service file:

```bash
find /lib/systemd/system /usr/lib/systemd/system -name "docker.socket"
```

This typically returns:
```
/lib/systemd/system/docker.socket
```

### Step 2: Examine the Current Configuration

Before making changes, examine the current configuration:

```bash
cat /lib/systemd/system/docker.socket
```

The file should look something like this:

```ini
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
```

Notice that the `SocketGroup` is set to `docker`.

### Step 3: Edit the docker.socket File

Now edit the file to change the group ownership to root:

```bash
sudo vi /lib/systemd/system/docker.socket
```

Modify the `SocketGroup` line to use root instead of docker:

```ini
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=root

[Install]
WantedBy=sockets.target
```

Save and exit the editor.

### Step 4: Reload the Systemd Daemon

After editing the service file, reload the systemd daemon to recognize the changes:

```bash
sudo systemctl daemon-reload
```

### Step 5: Restart the Docker Socket and Service

Restart both the Docker socket and the Docker service to apply the changes:

```bash
sudo systemctl restart docker.socket
sudo systemctl restart docker
```

### Step 6: Verify the Change

Verify that the Docker socket is now owned by root:root:

```bash
ls -la /var/run/docker.sock
```

You should see output similar to this:

```
srw-rw---- 1 root root 0 May 11 10:24 /var/run/docker.sock
```

Notice that both the user and group are now "root" instead of "root docker".

### Step 7: Test Docker Functionality

Test that Docker still works when accessed with sudo:

```bash
sudo docker ps
```

This should execute successfully and list running containers if any.

Try to run Docker as a non-root user:

```bash
docker ps
```

This should fail with a permissions error, confirming that only root can now access Docker.

## Security Implications

1. **Improved Security**: Only the root user can now interact with the Docker daemon, reducing the attack surface.

2. **Restricted Access**: Non-root users (even those in the docker group) can no longer run Docker commands directly.

3. **Operational Impact**: Any applications or services that rely on docker group access will need to be reconfigured to use sudo or another privilege escalation method.

4. **Principle of Least Privilege**: This change implements the security principle of granting only the minimum necessary privileges.

## Conclusion

By changing the Docker socket group ownership from `docker` to `root`, we've significantly enhanced the security of the Docker daemon by restricting access to only the root user. This modification aligns with security best practices for systems where Docker access should be tightly controlled.

Remember that after this change, all Docker commands must be executed with sudo or by the root user, which might require adjustments to scripts, CI/CD pipelines, or user workflows that previously relied on docker group membership.
