# Configure Packet Filtering, Port Redirection, and NAT

## Overview
This guide covers firewall configuration using firewalld and iptables, including packet filtering, port forwarding (redirection), and Network Address Translation (NAT) in Linux.

---

## Firewall Concepts

### Key Terms
- **Packet Filtering**: Controlling network traffic based on rules
- **Stateful Firewall**: Tracks connection state
- **Stateless Firewall**: Examines packets individually
- **NAT**: Network Address Translation (translates IP addresses)
- **Port Forwarding**: Redirects traffic from one port to another
- **Masquerading**: Dynamic source NAT (SNAT) for outbound traffic
- **DNAT**: Destination NAT (port forwarding, load balancing)
- **SNAT**: Source NAT (masquerading, explicit source translation)

### Netfilter/iptables Tables
- **filter**: Default table for packet filtering (INPUT, OUTPUT, FORWARD)
- **nat**: Network address translation (PREROUTING, POSTROUTING, OUTPUT)
- **mangle**: Packet alteration (all chains)
- **raw**: Connection tracking exemption (PREROUTING, OUTPUT)

### iptables Chains
- **INPUT**: Incoming packets destined for local system
- **OUTPUT**: Outgoing packets from local system
- **FORWARD**: Packets being routed through the system
- **PREROUTING**: Packets before routing decision (DNAT)
- **POSTROUTING**: Packets after routing decision (SNAT/Masquerading)

---

## firewalld (Modern Firewall Management)

### Overview
firewalld is a dynamic firewall daemon that uses zones and services for easier management.

### Installation and Service Management
```bash
# Install firewalld
dnf install firewalld

# Start service
systemctl start firewalld

# Enable at boot
systemctl enable firewalld

# Check status
systemctl status firewalld
firewall-cmd --state

# Stop firewall
systemctl stop firewalld

# Disable at boot
systemctl disable firewalld
```

### Zones Concept
Zones define trust levels for network connections.

```bash
# List all zones
firewall-cmd --get-zones

# List active zones
firewall-cmd --get-active-zones

# Get default zone
firewall-cmd --get-default-zone

# Set default zone
firewall-cmd --set-default-zone=public

# List configuration for zone
firewall-cmd --zone=public --list-all

# List configuration for all zones
firewall-cmd --list-all-zones
```

Common zones:
- **drop**: Drop all incoming, allow outgoing
- **block**: Reject all incoming, allow outgoing
- **public**: Public networks (default)
- **external**: External networks with masquerading
- **dmz**: DMZ (limited access)
- **work**: Work networks
- **home**: Home networks
- **internal**: Internal networks
- **trusted**: Trust all connections

### Basic firewall-cmd Operations

#### View Current Configuration
```bash
# Show all settings
firewall-cmd --list-all

# Show settings for specific zone
firewall-cmd --zone=public --list-all

# List services
firewall-cmd --list-services

# List ports
firewall-cmd --list-ports

# List rich rules
firewall-cmd --list-rich-rules

# List all
firewall-cmd --list-all-zones
```

#### Runtime vs Permanent Changes
```bash
# Runtime change (lost on reload/reboot)
firewall-cmd --add-service=http

# Permanent change (survives reload/reboot)
firewall-cmd --permanent --add-service=http

# Apply both runtime and permanent
firewall-cmd --add-service=http
firewall-cmd --permanent --add-service=http

# Reload to apply permanent changes
firewall-cmd --reload

# Complete restart
firewall-cmd --complete-reload
```

### Managing Services

#### Predefined Services
```bash
# List available services
firewall-cmd --get-services

# Add service
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ssh

# Add multiple services
firewall-cmd --permanent --add-service={http,https,ssh}

# Remove service
firewall-cmd --permanent --remove-service=http

# Query service
firewall-cmd --query-service=http
```

#### Custom Services
Service definitions: `/etc/firewalld/services/`

```bash
# Create custom service file
# /etc/firewalld/services/myapp.xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>MyApp</short>
  <description>My Application Service</description>
  <port protocol="tcp" port="8080"/>
  <port protocol="udp" port="8080"/>
</service>

# Reload firewalld
firewall-cmd --reload

# Add custom service
firewall-cmd --permanent --add-service=myapp
firewall-cmd --reload
```

### Managing Ports

```bash
# Add single port
firewall-cmd --permanent --add-port=8080/tcp

# Add port range
firewall-cmd --permanent --add-port=8000-8100/tcp

# Add multiple ports
firewall-cmd --permanent --add-port={80/tcp,443/tcp,8080/tcp}

# Remove port
firewall-cmd --permanent --remove-port=8080/tcp

# Query port
firewall-cmd --query-port=8080/tcp

# Add UDP port
firewall-cmd --permanent --add-port=53/udp
```

### Source-Based Filtering

```bash
# Allow traffic from specific source
firewall-cmd --permanent --add-source=192.168.1.0/24

# Add source to specific zone
firewall-cmd --permanent --zone=trusted --add-source=10.0.0.0/8

# Remove source
firewall-cmd --permanent --remove-source=192.168.1.100

# Query source
firewall-cmd --query-source=192.168.1.0/24

# Change zone for interface
firewall-cmd --zone=public --change-interface=eth0
```

### Rich Rules
More complex firewall rules with additional options.

```bash
# Allow SSH from specific IP
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" service name="ssh" accept'

# Allow HTTP from subnet
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="http" accept'

# Reject traffic from IP
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" reject'

# Drop traffic from IP
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" drop'

# Log accepted packets
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" log prefix="SSH-ACCESS" level="info" accept'

# Rate limiting (prevent DoS)
firewall-cmd --permanent --add-rich-rule='rule service name="ssh" accept limit value="3/m"'

# Port-based rule
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port port="8080" protocol="tcp" accept'

# Time-based rule (not commonly supported)
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="http" accept'

# Remove rich rule
firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.1.100" service name="ssh" accept'

# List rich rules
firewall-cmd --list-rich-rules
```

### Port Forwarding (Port Redirection)

#### Local Port Forwarding
```bash
# Forward port 80 to 8080 on same system
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080

# Forward to different host
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.100:toport=8080

# Forward with source filtering
firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" forward-port port="80" protocol="tcp" to-port="8080"'

# Remove port forward
firewall-cmd --permanent --remove-forward-port=port=80:proto=tcp:toport=8080
```

### Masquerading (NAT)

```bash
# Enable masquerading (SNAT for outbound traffic)
firewall-cmd --permanent --zone=public --add-masquerade

# Disable masquerading
firewall-cmd --permanent --zone=public --remove-masquerade

# Query masquerading
firewall-cmd --zone=public --query-masquerade

# Common use case: Internet gateway
# External interface (public zone) with masquerading enabled
firewall-cmd --permanent --zone=public --add-interface=eth0
firewall-cmd --permanent --zone=public --add-masquerade

# Internal interface (internal zone)
firewall-cmd --permanent --zone=internal --add-interface=eth1
```

### Direct Rules
Raw iptables rules within firewalld.

```bash
# Add direct rule
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -p tcp --dport 9000 -j ACCEPT

# Remove direct rule
firewall-cmd --permanent --direct --remove-rule ipv4 filter INPUT 0 -p tcp --dport 9000 -j ACCEPT

# List direct rules
firewall-cmd --direct --get-all-rules
```

### Interface Management

```bash
# Add interface to zone
firewall-cmd --permanent --zone=public --add-interface=eth0

# Change interface zone
firewall-cmd --zone=internal --change-interface=eth1

# Remove interface from zone
firewall-cmd --permanent --zone=public --remove-interface=eth0

# Query interface
firewall-cmd --get-zone-of-interface=eth0
```

### Panic Mode
Emergency mode that blocks all traffic.

```bash
# Enable panic mode
firewall-cmd --panic-on

# Disable panic mode
firewall-cmd --panic-off

# Query panic mode
firewall-cmd --query-panic
```

---

## iptables (Traditional Firewall)

### Installation
```bash
# Install iptables
dnf install iptables iptables-services

# Stop firewalld (conflicts with iptables)
systemctl stop firewalld
systemctl disable firewalld
systemctl mask firewalld

# Enable iptables
systemctl start iptables
systemctl enable iptables
```

### Basic iptables Syntax
```bash
iptables [-t table] COMMAND CHAIN PARAMETERS -j TARGET
```

### Viewing Rules

```bash
# List all rules
iptables -L

# List with line numbers
iptables -L --line-numbers

# List with verbose output
iptables -L -v

# List with numeric output (no DNS)
iptables -L -n

# List specific chain
iptables -L INPUT

# List specific table
iptables -t nat -L

# List all with packet counts
iptables -L -v -n

# Show rules as commands
iptables-save
```

### Basic Packet Filtering

#### Allow/Deny Traffic
```bash
# Accept all incoming HTTP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Accept all incoming HTTPS
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Accept SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Accept from specific IP
iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Accept from subnet
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

# Drop from specific IP
iptables -A INPUT -s 10.0.0.5 -j DROP

# Reject from specific IP
iptables -A INPUT -s 10.0.0.5 -j REJECT

# Accept on specific interface
iptables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT

# Accept established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Accept loopback
iptables -A INPUT -i lo -j ACCEPT
```

#### Default Policies
```bash
# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# View policies
iptables -L | grep policy
```

#### Common Rule Patterns
```bash
# Allow SSH with rate limiting
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow ICMP (ping)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Allow DNS
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# Allow established and related
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Log dropped packets
iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROPPED: " --log-level 4
iptables -A INPUT -j DROP
```

### Deleting Rules

```bash
# Delete by specification
iptables -D INPUT -p tcp --dport 80 -j ACCEPT

# Delete by line number
iptables -D INPUT 5

# Delete all rules in chain
iptables -F INPUT

# Delete all rules in all chains
iptables -F

# Delete all rules in specific table
iptables -t nat -F
```

### Inserting Rules

```bash
# Insert at specific position
iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT

# Replace rule at position
iptables -R INPUT 1 -p tcp --dport 2222 -j ACCEPT
```

### NAT with iptables

#### Masquerading (SNAT)
```bash
# Enable IP forwarding (required for NAT)
echo 1 > /proc/sys/net/ipv4/ip_forward

# Make permanent
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Masquerade outgoing traffic
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Masquerade specific subnet
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# SNAT (explicit source translation)
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.5
```

#### DNAT (Port Forwarding)
```bash
# Forward external port to internal server
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80

# Forward to different port
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

# Forward with source restriction
iptables -t nat -A PREROUTING -s 10.0.0.0/8 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80

# Port range forwarding
iptables -t nat -A PREROUTING -p tcp --dport 8000:8100 -j DNAT --to-destination 192.168.1.100

# Allow forwarded traffic
iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

#### Complete NAT Router Example
```bash
# Enable IP forwarding
sysctl -w net.ipv4.ip_forward=1

# INPUT chain (for router itself)
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP

# FORWARD chain (for routed traffic)
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -j DROP

# NAT (masquerade internal network)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding (external 80 to internal 192.168.1.100:80)
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80
iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT
```

### Advanced iptables Features

#### Connection Tracking
```bash
# Match connection states
iptables -A INPUT -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m state --state INVALID -j DROP

# Match conntrack states
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

#### Rate Limiting
```bash
# Limit new connections
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

# Limit ICMP
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second -j ACCEPT

# Recent module for rate limiting
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 --rttl --name SSH -j DROP
```

#### Multiple Ports
```bash
# Match multiple ports
iptables -A INPUT -p tcp -m multiport --dports 80,443,8080 -j ACCEPT

# Match port ranges
iptables -A INPUT -p tcp --dport 8000:8100 -j ACCEPT
```

#### MAC Address Filtering
```bash
# Allow specific MAC address
iptables -A INPUT -m mac --mac-source 00:11:22:33:44:55 -j ACCEPT
```

#### Time-Based Rules
```bash
# Allow during specific hours
iptables -A INPUT -p tcp --dport 80 -m time --timestart 09:00 --timestop 18:00 -j ACCEPT

# Allow on specific days
iptables -A INPUT -p tcp --dport 80 -m time --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
```

#### String Matching
```bash
# Block packets containing string
iptables -A FORWARD -m string --string "badstring" --algo bm -j DROP
```

### Saving and Restoring Rules

```bash
# Save rules
iptables-save > /etc/sysconfig/iptables

# Restore rules
iptables-restore < /etc/sysconfig/iptables

# Save via service (RHEL/CentOS)
service iptables save

# Persistent rules across reboots
systemctl enable iptables
```

---

## nftables (Modern Replacement for iptables)

### Overview
nftables is the modern successor to iptables, offering better performance and syntax.

### Installation
```bash
# Install nftables
dnf install nftables

# Enable service
systemctl enable nftables
systemctl start nftables
```

### Basic Commands
```bash
# List ruleset
nft list ruleset

# Add table
nft add table inet filter

# Add chain
nft add chain inet filter input { type filter hook input priority 0 \; }

# Add rule
nft add rule inet filter input tcp dport 22 accept

# Delete rule by handle
nft -a list ruleset  # Show handles
nft delete rule inet filter input handle 5

# Flush rules
nft flush ruleset

# Save configuration
nft list ruleset > /etc/nftables.conf

# Load configuration
nft -f /etc/nftables.conf
```

---

## IP Forwarding and Routing

### Enable IP Forwarding

```bash
# Temporary
echo 1 > /proc/sys/net/ipv4/ip_forward

# Check status
cat /proc/sys/net/ipv4/ip_forward

# Permanent
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# For IPv6
echo "net.ipv6.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
```

### Connection Tracking

```bash
# View connection tracking table
conntrack -L

# Count connections
conntrack -C

# Delete specific connection
conntrack -D -p tcp --dport 80

# View statistics
cat /proc/net/nf_conntrack
```

---

## Common Firewall Scenarios

### Scenario 1: Basic Web Server
```bash
# Using firewalld
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload

# Using iptables
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -j DROP
iptables-save > /etc/sysconfig/iptables
```

### Scenario 2: NAT Gateway
```bash
# Enable forwarding
sysctl -w net.ipv4.ip_forward=1

# Using firewalld
firewall-cmd --permanent --zone=external --add-interface=eth0
firewall-cmd --permanent --zone=external --add-masquerade
firewall-cmd --permanent --zone=internal --add-interface=eth1
firewall-cmd --reload

# Using iptables
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

### Scenario 3: Port Forwarding (DMZ)
```bash
# Forward external 80 to DMZ server
# Using firewalld
firewall-cmd --permanent --zone=public --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.100:toport=80
firewall-cmd --reload

# Using iptables
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80
iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
```

### Scenario 4: Restrict SSH Access
```bash
# Allow SSH only from specific network
# Using firewalld
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.0/8" service name="ssh" accept'
firewall-cmd --reload

# Using iptables
iptables -A INPUT -p tcp -s 10.0.0.0/8 --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

---

## Troubleshooting

### Debug firewalld
```bash
# Check service status
firewall-cmd --state
systemctl status firewalld

# View logs
journalctl -u firewalld -f

# Test rules
firewall-cmd --direct --get-all-rules
firewall-cmd --list-all

# Reload
firewall-cmd --reload
```

### Debug iptables
```bash
# View packet counts
iptables -L -v -n

# Watch packets in real-time
watch -n 1 'iptables -L -v -n'

# Enable logging
iptables -A INPUT -j LOG --log-prefix "IPTABLES: "
tail -f /var/log/messages

# Trace packets
iptables -t raw -A PREROUTING -p tcp --dport 80 -j TRACE
```

### Test Connectivity
```bash
# Test from external
telnet server_ip 80
nc -vz server_ip 80

# Test locally
curl localhost:80
ss -tlnp | grep :80
```

---

## Quick Reference

### firewalld
```bash
firewall-cmd --list-all                                    # View configuration
firewall-cmd --permanent --add-service=http                # Add service
firewall-cmd --permanent --add-port=8080/tcp               # Add port
firewall-cmd --permanent --add-source=192.168.1.0/24       # Add source
firewall-cmd --permanent --add-masquerade                  # Enable NAT
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080  # Port forward
firewall-cmd --reload                                      # Apply changes
```

### iptables
```bash
iptables -L -n -v                                         # List rules
iptables -A INPUT -p tcp --dport 80 -j ACCEPT            # Add rule
iptables -D INPUT 5                                       # Delete rule
iptables -F                                               # Flush all
iptables-save > /etc/sysconfig/iptables                  # Save rules
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE     # Masquerade
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to 192.168.1.100  # Port forward
```

---

## Exam Tips

- Know both firewalld and iptables basics
- Understand the difference between runtime and permanent changes
- Practice NAT and port forwarding scenarios
- Remember to enable IP forwarding for routing/NAT
- Know how to troubleshoot with logs and packet counts
- Understand firewalld zones concept
- Be comfortable with rich rules in firewalld
- Know the iptables table/chain structure
- Practice saving and restoring firewall rules
- Understand connection tracking and stateful filtering
