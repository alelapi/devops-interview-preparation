# Configure IPv4 and IPv6 Networking and Hostname Resolution

## Overview
This guide covers essential commands and configurations for managing network interfaces, IP addressing (both IPv4 and IPv6), and hostname resolution in Linux systems.

---

## Network Interface Management

### `ip` Command
The modern standard for network configuration in Linux, replacing older tools like `ifconfig`.

#### Display Network Interfaces
```bash
# Show all network interfaces
ip link show

# Show specific interface
ip link show eth0

# Show interface statistics
ip -s link show eth0
```

#### Enable/Disable Interfaces
```bash
# Bring interface up
ip link set eth0 up

# Bring interface down
ip link set eth0 down
```

---

## IPv4 Configuration

### Viewing IPv4 Addresses
```bash
# Show all IPv4 addresses
ip -4 addr show

# Show IPv4 for specific interface
ip addr show eth0

# Brief output
ip -br addr show
```

### Adding/Removing IPv4 Addresses
```bash
# Add IPv4 address
ip addr add 192.168.1.100/24 dev eth0

# Add with broadcast
ip addr add 192.168.1.100/24 broadcast 192.168.1.255 dev eth0

# Remove IPv4 address
ip addr del 192.168.1.100/24 dev eth0

# Flush all addresses from interface
ip addr flush dev eth0
```

### Default Gateway
```bash
# Show routing table
ip route show

# Add default gateway
ip route add default via 192.168.1.1

# Delete default gateway
ip route del default via 192.168.1.1

# Add route to specific network
ip route add 10.0.0.0/8 via 192.168.1.254
```

---

## IPv6 Configuration

### Viewing IPv6 Addresses
```bash
# Show all IPv6 addresses
ip -6 addr show

# Show IPv6 only for specific interface
ip -6 addr show eth0

# Show IPv6 routing table
ip -6 route show
```

### Adding/Removing IPv6 Addresses
```bash
# Add IPv6 address
ip -6 addr add 2001:db8::1/64 dev eth0

# Remove IPv6 address
ip -6 addr del 2001:db8::1/64 dev eth0

# Add IPv6 default gateway
ip -6 route add default via 2001:db8::ff

# Delete IPv6 default gateway
ip -6 route del default via 2001:db8::ff
```

### IPv6 Link-Local Addresses
```bash
# View link-local addresses (fe80::/10)
ip -6 addr show scope link

# Ping using link-local (must specify interface)
ping6 fe80::a00:27ff:fe4e:66a1%eth0
```

---

## Network Configuration Files

### NetworkManager (RHEL/CentOS/Fedora)

#### Using `nmcli` Command
```bash
# Show all connections
nmcli connection show

# Show active connections
nmcli connection show --active

# Show device status
nmcli device status

# Create new connection with static IPv4
nmcli connection add \
    type ethernet \
    con-name eth0-static \
    ifname eth0 \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns "8.8.8.8 8.8.4.4" \
    ipv4.method manual

# Create DHCP connection
nmcli connection add \
    type ethernet \
    con-name eth0-dhcp \
    ifname eth0 \
    ipv4.method auto

# Modify existing connection
nmcli connection modify eth0-static ipv4.addresses 192.168.1.101/24

# Add secondary IP address
nmcli connection modify eth0-static +ipv4.addresses 192.168.1.102/24

# Configure IPv6
nmcli connection modify eth0-static \
    ipv6.addresses 2001:db8::100/64 \
    ipv6.gateway 2001:db8::1 \
    ipv6.method manual

# Enable/disable IPv6
nmcli connection modify eth0-static ipv6.method auto
nmcli connection modify eth0-static ipv6.method ignore

# Activate connection
nmcli connection up eth0-static

# Deactivate connection
nmcli connection down eth0-static

# Delete connection
nmcli connection delete eth0-static

# Reload configuration
nmcli connection reload
```

### Configuration Files (RHEL/CentOS)
Location: `/etc/sysconfig/network-scripts/ifcfg-<interface>`

Example static IPv4 configuration:
```bash
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

Example DHCP configuration:
```bash
TYPE=Ethernet
BOOTPROTO=dhcp
NAME=eth0
DEVICE=eth0
ONBOOT=yes
```

Example IPv6 configuration:
```bash
IPV6INIT=yes
IPV6ADDR=2001:db8::100/64
IPV6_DEFAULTGW=2001:db8::1
```

### Netplan (Ubuntu/Debian)
Location: `/etc/netplan/*.yaml`

Example static configuration:
```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
        - 2001:db8::100/64
      gateway4: 192.168.1.1
      gateway6: 2001:db8::1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
          - 2001:4860:4860::8888
```

Example DHCP configuration:
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: true
```

Apply Netplan configuration:
```bash
# Test configuration
netplan try

# Apply configuration
netplan apply

# Generate backend configuration
netplan generate
```

---

## Hostname Configuration

### Viewing Hostname
```bash
# Show hostname
hostname

# Show FQDN (Fully Qualified Domain Name)
hostname -f

# Show short hostname
hostname -s

# Show all hostname information
hostnamectl
```

### Setting Hostname

#### Using `hostnamectl` (systemd systems)
```bash
# Set hostname
hostnamectl set-hostname server1.example.com

# Set static hostname only
hostnamectl set-hostname server1 --static

# Set pretty hostname (descriptive name)
hostnamectl set-hostname "Production Web Server" --pretty

# Set transient hostname (temporary)
hostnamectl set-hostname temp-host --transient
```

#### Manual Configuration Files
```bash
# RHEL/CentOS/Fedora
echo "server1.example.com" > /etc/hostname

# Also update /etc/hosts
echo "192.168.1.100 server1.example.com server1" >> /etc/hosts
```

---

## DNS and Name Resolution

### DNS Configuration File
Location: `/etc/resolv.conf`

```bash
# Configure DNS servers
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 2001:4860:4860::8888

# Search domains
search example.com local.domain

# Options
options timeout:2
options attempts:3
```

### `/etc/hosts` File
Static hostname to IP mapping:

```bash
# IPv4
127.0.0.1       localhost localhost.localdomain
192.168.1.100   server1.example.com server1
192.168.1.101   server2.example.com server2

# IPv6
::1             localhost localhost.localdomain
2001:db8::100   server1.example.com server1
```

### `/etc/nsswitch.conf`
Controls the order of name resolution sources:

```bash
# Important line for hostname resolution
hosts: files dns myhostname

# This means: check /etc/hosts first, then DNS, then systemd hostname
```

### DNS Testing Commands

#### `nslookup`
```bash
# Query DNS for domain
nslookup example.com

# Query specific DNS server
nslookup example.com 8.8.8.8

# Reverse lookup
nslookup 8.8.8.8
```

#### `dig`
```bash
# Basic query
dig example.com

# Query specific record type
dig example.com A      # IPv4
dig example.com AAAA   # IPv6
dig example.com MX     # Mail exchange
dig example.com NS     # Name servers

# Query specific DNS server
dig @8.8.8.8 example.com

# Short output
dig +short example.com

# Reverse lookup
dig -x 8.8.8.8

# Trace DNS resolution path
dig +trace example.com
```

#### `host`
```bash
# Simple lookup
host example.com

# Reverse lookup
host 8.8.8.8

# Query specific record type
host -t MX example.com
host -t AAAA example.com

# Use specific DNS server
host example.com 8.8.8.8
```

#### `getent`
```bash
# Query using system resolution (respects /etc/nsswitch.conf)
getent hosts example.com

# This uses the configured resolution order (files, DNS, etc.)
getent ahosts example.com  # All addresses
```

---

## Network Manager Text UI

### `nmtui`
Interactive text-based interface for NetworkManager:

```bash
# Launch network configuration UI
nmtui

# Directly launch specific function
nmtui edit       # Edit connection
nmtui connect    # Activate connection
nmtui hostname   # Set hostname
```

---

## SystemD Network Configuration

### `systemd-networkd`
Alternative to NetworkManager on systemd systems.

Configuration files: `/etc/systemd/network/*.network`

Example configuration:
```ini
# /etc/systemd/network/20-wired.network
[Match]
Name=eth0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=8.8.4.4
```

Enable and start:
```bash
systemctl enable systemd-networkd
systemctl start systemd-networkd
systemctl status systemd-networkd
```

---

## Useful Network Information Commands

### Show Network Configuration Summary
```bash
# Modern way
ip addr show

# Show only IPv4
ip -4 addr

# Show only IPv6
ip -6 addr

# Show with colors
ip -c addr

# Brief output
ip -br addr show
```

### Check Connectivity
```bash
# Ping IPv4
ping -c 4 192.168.1.1

# Ping IPv6
ping6 -c 4 2001:db8::1

# Ping with specific interface
ping -I eth0 192.168.1.1
```

---

## Persistent vs Temporary Configuration

**Temporary (runtime only):**
```bash
ip addr add 192.168.1.100/24 dev eth0
# Lost after reboot
```

**Persistent (survives reboot):**
- Use `nmcli` commands
- Edit configuration files in `/etc/sysconfig/network-scripts/` (RHEL)
- Edit Netplan files `/etc/netplan/` (Ubuntu)
- Use `systemd-networkd` configuration files

---

## Common Troubleshooting Steps

1. **Check interface status:**
   ```bash
   ip link show
   ip addr show
   ```

2. **Check routing:**
   ```bash
   ip route show
   ip -6 route show
   ```

3. **Test connectivity:**
   ```bash
   ping -c 4 8.8.8.8
   ping6 -c 4 2001:4860:4860::8888
   ```

4. **Check DNS resolution:**
   ```bash
   dig example.com
   nslookup example.com
   getent hosts example.com
   ```

5. **Verify configuration files:**
   ```bash
   cat /etc/resolv.conf
   cat /etc/hosts
   cat /etc/nsswitch.conf
   ```

6. **Check NetworkManager/networkd status:**
   ```bash
   systemctl status NetworkManager
   systemctl status systemd-networkd
   ```

---

## Key Exam Tips

- Know both temporary (`ip` commands) and persistent (config files, `nmcli`) methods
- Understand IPv4 and IPv6 addressing and configuration
- Be comfortable with CIDR notation (e.g., /24, /64)
- Practice DNS configuration and testing
- Understand the hostname resolution order in `/etc/nsswitch.conf`
- Know how to configure both DHCP and static IP addresses
- Remember to restart/reload network services after configuration changes
