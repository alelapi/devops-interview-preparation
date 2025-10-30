# Configure OpenSSH Server and Client

## Overview
This guide covers configuration, management, and security best practices for OpenSSH server (sshd) and client (ssh) on Linux systems.

---

## SSH Basics

SSH (Secure Shell) provides encrypted network communication for:
- Remote command execution
- Secure file transfer (SCP, SFTP)
- Port forwarding and tunneling
- X11 forwarding for GUI applications

Default port: **22/TCP**

---

## Installation

### Install OpenSSH
```bash
# RHEL/CentOS/Fedora
dnf install openssh-server openssh-clients

# Ubuntu/Debian
apt install openssh-server openssh-client

# Check if installed
rpm -qa | grep openssh          # RHEL
dpkg -l | grep openssh          # Debian
```

---

## SSH Server (sshd) Configuration

### Service Management
```bash
# Start SSH service
systemctl start sshd

# Stop SSH service
systemctl stop sshd

# Restart SSH service
systemctl restart sshd

# Reload configuration (no connection drop)
systemctl reload sshd

# Enable at boot
systemctl enable sshd

# Check status
systemctl status sshd

# Check if enabled
systemctl is-enabled sshd

# View SSH service logs
journalctl -u sshd
journalctl -u sshd -f          # Follow logs
```

### Main Configuration File
Location: `/etc/ssh/sshd_config`

**Important Configuration Directives:**

```bash
# Port and address binding
Port 22                        # Change default port
#Port 2222                     # Custom port
ListenAddress 0.0.0.0          # Listen on all IPv4
ListenAddress ::               # Listen on all IPv6
#ListenAddress 192.168.1.100   # Specific IP only

# Protocol version
Protocol 2                     # Only use SSH protocol 2

# Host keys
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Logging
SyslogFacility AUTH
LogLevel INFO                  # QUIET, FATAL, ERROR, INFO, VERBOSE, DEBUG

# Authentication
PermitRootLogin no             # Disable root login (RECOMMENDED)
#PermitRootLogin prohibit-password  # Allow only key-based root login
#PermitRootLogin yes            # Allow all root login (NOT RECOMMENDED)

PubkeyAuthentication yes       # Enable public key authentication
PasswordAuthentication yes     # Enable password authentication
PermitEmptyPasswords no        # Disable empty passwords
ChallengeResponseAuthentication no

# PAM authentication
UsePAM yes

# Kerberos authentication
KerberosAuthentication no
KerberosOrLocalPasswd yes

# GSSAPI authentication
GSSAPIAuthentication no

# Connection settings
X11Forwarding yes              # Enable X11 forwarding
X11UseLocalhost yes
PrintMotd no                   # Don't print /etc/motd
PrintLastLog yes
TCPKeepAlive yes
ClientAliveInterval 300        # Send keepalive every 300 seconds
ClientAliveCountMax 3          # Disconnect after 3 failed keepalives

# Login settings
MaxAuthTries 3                 # Maximum authentication attempts
MaxSessions 10                 # Maximum sessions per connection
LoginGraceTime 60              # Time to authenticate (seconds)

# Subsystem
Subsystem sftp /usr/libexec/openssh/sftp-server

# User/Group restrictions
AllowUsers user1 user2         # Only these users can connect
#DenyUsers baduser             # These users cannot connect
AllowGroups sshusers           # Only members of these groups
#DenyGroups restricted         # Members cannot connect

# Banner
Banner /etc/ssh/banner         # Display banner before login
```

### Example: Secure Configuration
```bash
# /etc/ssh/sshd_config - Hardened configuration
Port 2222
Protocol 2
ListenAddress 0.0.0.0

# Encryption
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# Authentication
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes

# Access control
AllowGroups sshusers
MaxAuthTries 3
MaxSessions 5
LoginGraceTime 30

# Connection security
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
PrintMotd no
TCPKeepAlive yes

# Logging
LogLevel VERBOSE
SyslogFacility AUTH
```

### Test Configuration
```bash
# Test configuration syntax
sshd -t

# Test with specific config file
sshd -t -f /etc/ssh/sshd_config

# Test and show configuration
sshd -T

# Show configuration for specific user
sshd -T -C user=john
```

### Reload Configuration
```bash
# Reload without dropping connections
systemctl reload sshd

# Or send HUP signal
kill -HUP $(cat /var/run/sshd.pid)
```

---

## SSH Client Configuration

### Per-User Configuration
Location: `~/.ssh/config`

```bash
# Global defaults
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    ForwardAgent no

# Specific host
Host webserver
    HostName web.example.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/web_rsa
    
Host db
    HostName 192.168.1.100
    User dbadmin
    Port 22
    IdentityFile ~/.ssh/db_key
    ForwardAgent yes
    
# Pattern matching
Host *.example.com
    User john
    IdentityFile ~/.ssh/example_rsa
    
# Jump host (bastion)
Host internal-server
    HostName 10.0.1.100
    User admin
    ProxyJump bastion.example.com
    
# Multiple jump hosts
Host deep-server
    HostName 10.0.2.100
    ProxyJump bastion1.example.com,bastion2.example.com
```

### System-Wide Configuration
Location: `/etc/ssh/ssh_config`

---

## SSH Key Management

### Generate SSH Key Pairs

```bash
# Generate RSA key (default, 3072 bits)
ssh-keygen

# Generate RSA with specific bits
ssh-keygen -t rsa -b 4096 -C "user@email.com"

# Generate Ed25519 key (recommended, more secure)
ssh-keygen -t ed25519 -C "user@email.com"

# Generate ECDSA key
ssh-keygen -t ecdsa -b 521

# Specify file location
ssh-keygen -t ed25519 -f ~/.ssh/custom_key

# Generate without passphrase (not recommended)
ssh-keygen -t ed25519 -N ""

# Change passphrase of existing key
ssh-keygen -p -f ~/.ssh/id_ed25519
```

### Key Files
```bash
~/.ssh/id_rsa          # Private key (RSA)
~/.ssh/id_rsa.pub      # Public key (RSA)
~/.ssh/id_ed25519      # Private key (Ed25519)
~/.ssh/id_ed25519.pub  # Public key (Ed25519)
~/.ssh/authorized_keys # Authorized public keys
~/.ssh/known_hosts     # Known host fingerprints
~/.ssh/config          # Client configuration
```

### Deploy Public Key

```bash
# Method 1: Using ssh-copy-id (recommended)
ssh-copy-id user@remote-host
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@remote-host
ssh-copy-id -p 2222 user@remote-host

# Method 2: Manual copy
cat ~/.ssh/id_ed25519.pub | ssh user@remote-host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Method 3: Direct append
ssh user@remote-host "echo '$(cat ~/.ssh/id_ed25519.pub)' >> ~/.ssh/authorized_keys"

# Method 4: SCP
scp ~/.ssh/id_ed25519.pub user@remote-host:~/key.pub
ssh user@remote-host "mkdir -p ~/.ssh && cat ~/key.pub >> ~/.ssh/authorized_keys && rm ~/key.pub"
```

### Correct Permissions
```bash
# On remote server
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/id_*.pub
chmod 644 ~/.ssh/known_hosts
chmod 600 ~/.ssh/config

# Fix all at once
chmod 700 ~/.ssh && chmod 600 ~/.ssh/* && chmod 644 ~/.ssh/*.pub
```

### Manage Authorized Keys
```bash
# Add key to authorized_keys
echo "ssh-ed25519 AAAA..." >> ~/.ssh/authorized_keys

# View authorized keys
cat ~/.ssh/authorized_keys

# Remove specific key (edit file)
vim ~/.ssh/authorized_keys

# Limit key usage (prepend to key in authorized_keys)
from="192.168.1.0/24" ssh-ed25519 AAAA...
command="/usr/local/bin/backup.sh" ssh-ed25519 AAAA...
no-port-forwarding,no-X11-forwarding ssh-ed25519 AAAA...
```

### SSH Agent
```bash
# Start SSH agent
eval $(ssh-agent)

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Add with timeout (seconds)
ssh-add -t 3600 ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l

# List loaded keys with full public key
ssh-add -L

# Remove specific key
ssh-add -d ~/.ssh/id_ed25519

# Remove all keys
ssh-add -D

# Kill agent
ssh-agent -k
```

---

## Connecting with SSH

### Basic Connection
```bash
# Connect to host
ssh user@hostname

# Connect to specific port
ssh -p 2222 user@hostname

# Connect using specific key
ssh -i ~/.ssh/custom_key user@hostname

# Connect with verbose output
ssh -v user@hostname          # Level 1
ssh -vv user@hostname         # Level 2
ssh -vvv user@hostname        # Level 3 (most verbose)

# Connect and execute command
ssh user@hostname "ls -la /tmp"

# Connect and execute multiple commands
ssh user@hostname "uptime; df -h; free -m"

# Using config alias
ssh webserver                 # Uses ~/.ssh/config
```

### Advanced Connection Options
```bash
# Disable strict host key checking (use carefully)
ssh -o StrictHostKeyChecking=no user@hostname

# Specify cipher
ssh -c aes256-gcm@openssh.com user@hostname

# Enable compression
ssh -C user@hostname

# Allocate pseudo-TTY
ssh -t user@hostname

# No pseudo-TTY
ssh -T user@hostname

# Run in background
ssh -f user@hostname "sleep 10; command"

# Keep connection alive
ssh -o ServerAliveInterval=60 user@hostname

# X11 forwarding
ssh -X user@hostname
ssh -Y user@hostname          # Trusted X11 forwarding
```

### Jump Hosts (Bastion)
```bash
# Connect through jump host
ssh -J jump-host user@target-host

# Multiple jump hosts
ssh -J jump1,jump2 user@target

# Alternative syntax
ssh -o ProxyJump=jump-host user@target

# With different users
ssh -J jumpuser@jump-host targetuser@target
```

---

## SSH Tunneling and Port Forwarding

### Local Port Forwarding
Forward local port to remote destination.

```bash
# Basic syntax: ssh -L local_port:destination:destination_port user@ssh_server

# Forward local port 8080 to remote localhost:80
ssh -L 8080:localhost:80 user@remote-host

# Access remote database locally
ssh -L 3306:localhost:3306 user@database-server
# Now connect to localhost:3306 locally

# Forward to third host through SSH server
ssh -L 8080:internal-web:80 user@gateway

# Multiple port forwards
ssh -L 8080:web:80 -L 3306:db:3306 user@gateway

# Bind to specific interface
ssh -L 192.168.1.100:8080:localhost:80 user@remote
```

### Remote Port Forwarding
Forward remote port to local destination.

```bash
# Basic syntax: ssh -R remote_port:destination:destination_port user@ssh_server

# Expose local web server to remote
ssh -R 8080:localhost:80 user@remote-host
# Remote host can access your local web server on port 8080

# Allow remote server to forward to internal network
ssh -R 3306:database:3306 user@remote

# Bind to all remote interfaces (requires GatewayPorts yes in sshd_config)
ssh -R 0.0.0.0:8080:localhost:80 user@remote
```

### Dynamic Port Forwarding (SOCKS Proxy)
Create a SOCKS proxy for all traffic.

```bash
# Create SOCKS proxy on local port 1080
ssh -D 1080 user@remote-host

# Use specific bind address
ssh -D 127.0.0.1:1080 user@remote

# Configure browser or application to use SOCKS proxy:
# Host: localhost, Port: 1080, SOCKS v5

# Test SOCKS proxy
curl --socks5 localhost:1080 http://example.com
```

### Combined Tunneling
```bash
# Local + Dynamic forwarding
ssh -L 8080:web:80 -D 1080 user@gateway

# Background with no shell
ssh -fNL 8080:localhost:80 user@remote

# Options explained:
# -f: Background
# -N: No command execution
# -L: Local forward
# -R: Remote forward
# -D: Dynamic forward
```

### SSH Tunnel as Daemon
```bash
# Create persistent tunnel
ssh -fNL 8080:localhost:80 user@remote

# With auto-reconnect in cron
*/5 * * * * pgrep -f "ssh -fNL" || ssh -fNL 8080:localhost:80 user@remote

# Using autossh (better for persistent tunnels)
autossh -M 0 -fNL 8080:localhost:80 user@remote
```

---

## File Transfer with SSH

### SCP (Secure Copy)
```bash
# Copy file to remote
scp file.txt user@remote:/path/to/destination

# Copy file from remote
scp user@remote:/path/to/file.txt /local/destination

# Copy directory recursively
scp -r /local/dir user@remote:/remote/dir

# Copy with specific port
scp -P 2222 file.txt user@remote:/path

# Copy with compression
scp -C large_file.txt user@remote:/path

# Preserve permissions and timestamps
scp -p file.txt user@remote:/path

# Copy between two remote hosts
scp user1@remote1:/file user2@remote2:/path

# Limit bandwidth (in Kbps)
scp -l 1000 large_file user@remote:/path

# Verbose output
scp -v file.txt user@remote:/path
```

### SFTP (SSH File Transfer Protocol)
```bash
# Connect to SFTP server
sftp user@remote-host

# Connect with specific port
sftp -P 2222 user@remote

# SFTP interactive commands
sftp> ls                      # List remote directory
sftp> lls                     # List local directory
sftp> pwd                     # Print remote working directory
sftp> lpwd                    # Print local working directory
sftp> cd /remote/path         # Change remote directory
sftp> lcd /local/path         # Change local directory
sftp> get file.txt            # Download file
sftp> get -r directory/       # Download directory
sftp> put file.txt            # Upload file
sftp> put -r directory/       # Upload directory
sftp> mkdir newdir            # Create remote directory
sftp> rmdir directory         # Remove remote directory
sftp> rm file.txt             # Delete remote file
sftp> rename old.txt new.txt  # Rename remote file
sftp> chmod 755 script.sh     # Change permissions
sftp> exit                    # Quit

# Batch mode with command file
echo "get file.txt" | sftp user@remote

# Execute single command
sftp user@remote <<< "get /remote/file.txt"
```

### rsync over SSH
```bash
# Basic sync
rsync -avz /local/path/ user@remote:/remote/path/

# Sync with specific SSH port
rsync -avz -e "ssh -p 2222" /local/ user@remote:/remote/

# Sync with progress
rsync -avz --progress /local/ user@remote:/remote/

# Dry run (test without changes)
rsync -avz --dry-run /local/ user@remote:/remote/

# Delete files in destination not in source
rsync -avz --delete /local/ user@remote:/remote/

# Exclude files
rsync -avz --exclude='*.log' /local/ user@remote:/remote/

# Options explained:
# -a: Archive mode (preserves permissions, times, etc.)
# -v: Verbose
# -z: Compress
# -e: Specify SSH command
```

---

## SSH Security Best Practices

### 1. Disable Root Login
```bash
# In /etc/ssh/sshd_config
PermitRootLogin no
```

### 2. Use Key-Based Authentication
```bash
# Generate strong key
ssh-keygen -t ed25519 -C "user@email.com"

# Disable password authentication after deploying keys
PasswordAuthentication no
```

### 3. Change Default Port
```bash
# In /etc/ssh/sshd_config
Port 2222

# Update firewall
firewall-cmd --permanent --add-port=2222/tcp
firewall-cmd --reload
```

### 4. Limit User Access
```bash
# In /etc/ssh/sshd_config
AllowUsers john jane admin
# Or use groups
AllowGroups sshusers
```

### 5. Use fail2ban
```bash
# Install fail2ban
dnf install fail2ban

# Configure for SSH
# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/secure
maxretry = 3
bantime = 3600
```

### 6. Two-Factor Authentication
```bash
# Install Google Authenticator PAM module
dnf install google-authenticator

# Configure PAM
# Add to /etc/pam.d/sshd
auth required pam_google_authenticator.so

# Configure sshd_config
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

### 7. Restrict SSH Protocol
```bash
# Only use protocol 2
Protocol 2
```

### 8. Set Login Grace Time
```bash
# Limit time to authenticate
LoginGraceTime 30
```

### 9. Use TCP Wrappers
```bash
# /etc/hosts.allow
sshd: 192.168.1.0/24

# /etc/hosts.deny
sshd: ALL
```

### 10. Configure Idle Timeout
```bash
# In /etc/ssh/sshd_config
ClientAliveInterval 300
ClientAliveCountMax 2
```

---

## Troubleshooting SSH

### Check SSH Service
```bash
# Service status
systemctl status sshd

# Check if listening
ss -tlnp | grep ssh
netstat -tlnp | grep ssh

# Test configuration
sshd -t

# View logs
journalctl -u sshd -f
tail -f /var/log/secure    # RHEL
tail -f /var/log/auth.log  # Ubuntu
```

### Verbose Connection Testing
```bash
# Client-side debugging
ssh -vvv user@host

# Look for:
# - Key exchange
# - Authentication attempts
# - Cipher negotiation
# - Connection errors
```

### Common Issues

#### Connection Refused
```bash
# Check if service running
systemctl status sshd

# Check firewall
firewall-cmd --list-all
iptables -L -n

# Check if listening on correct interface
ss -tlnp | grep :22
```

#### Permission Denied (publickey)
```bash
# Check key permissions
ls -la ~/.ssh/

# Should be:
# 700 for ~/.ssh/
# 600 for private keys
# 644 for public keys

# Check server logs
tail -f /var/log/secure

# Verify key is in authorized_keys
cat ~/.ssh/authorized_keys

# Test with password (if enabled)
ssh -o PubkeyAuthentication=no user@host
```

#### Host Key Verification Failed
```bash
# Remove old key
ssh-keygen -R hostname

# Or edit known_hosts
vim ~/.ssh/known_hosts

# Accept new key
ssh -o StrictHostKeyChecking=no user@host
```

#### Too Many Authentication Failures
```bash
# Limit keys offered
ssh -o IdentitiesOnly=yes -i ~/.ssh/specific_key user@host

# Or configure in ~/.ssh/config
Host problem-host
    IdentitiesOnly yes
    IdentityFile ~/.ssh/specific_key
```

---

## Firewall Configuration

### firewalld
```bash
# Allow SSH
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload

# Allow custom SSH port
firewall-cmd --permanent --add-port=2222/tcp
firewall-cmd --reload

# Allow from specific source
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'
firewall-cmd --reload
```

### iptables
```bash
# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow from specific source
iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT

# Save rules
iptables-save > /etc/sysconfig/iptables
```

---

## Quick Reference Commands

### Service Management
```bash
systemctl start/stop/restart sshd    # Manage service
systemctl enable sshd                 # Enable at boot
sshd -t                               # Test configuration
```

### Key Management
```bash
ssh-keygen -t ed25519                 # Generate key
ssh-copy-id user@host                 # Deploy key
ssh-add                               # Add to agent
```

### Connecting
```bash
ssh user@host                         # Basic connection
ssh -p 2222 user@host                 # Custom port
ssh -i key user@host                  # Specific key
ssh -J jump user@target               # Jump host
```

### Port Forwarding
```bash
ssh -L 8080:dest:80 user@host        # Local forward
ssh -R 8080:dest:80 user@host        # Remote forward
ssh -D 1080 user@host                 # SOCKS proxy
```

### File Transfer
```bash
scp file user@host:/path              # Copy file
sftp user@host                        # Interactive transfer
rsync -avz /src/ user@host:/dst/     # Sync directories
```

---

## Exam Tips

- Know both server (`sshd_config`) and client (`ssh_config`) configuration
- Understand key-based authentication setup and troubleshooting
- Practice port forwarding scenarios (local, remote, dynamic)
- Know how to secure SSH (disable root, change port, keys only)
- Understand file permissions for SSH directories and files
- Be comfortable with `ssh-keygen`, `ssh-copy-id`, and `ssh-agent`
- Know how to test SSH configuration without breaking access
- Practice troubleshooting with verbose output (`-vvv`)
- Understand TCP wrappers and firewall configuration for SSH
- Know the difference between SCP, SFTP, and rsync
