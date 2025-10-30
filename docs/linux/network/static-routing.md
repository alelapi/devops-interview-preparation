# Configure Static Routing

## Overview
This guide covers static routing configuration, route management, policy-based routing, and advanced routing concepts in Linux.

---

## Routing Concepts

### Key Terms
- **Routing**: Process of forwarding packets between networks
- **Static Route**: Manually configured route (doesn't change automatically)
- **Dynamic Route**: Automatically learned route (via routing protocols)
- **Default Gateway**: Route used when no specific route matches
- **Metric**: Cost or priority of a route (lower is preferred)
- **Next Hop**: Next router/gateway to forward packets
- **Administrative Distance**: Route preference (lower is preferred)
- **Policy Routing**: Routing based on source, not just destination

### Routing Table Components
- **Destination**: Target network
- **Gateway**: Next hop IP address
- **Genmask/Prefix**: Network mask
- **Interface**: Outgoing network interface
- **Metric**: Route priority

---

## Viewing Routing Tables

### Using `ip route` Command
```bash
# Show routing table
ip route show
ip route list
ip r

# Show with details
ip route show table all

# Show specific table
ip route show table main
ip route show table local

# Show IPv6 routes
ip -6 route show

# Show route to specific destination
ip route get 8.8.8.8
ip route get 192.168.1.100

# Show cached routes (deprecated in newer kernels)
ip route show cache
```

### Using `route` Command (Legacy)
```bash
# Show routing table
route -n

# Show with hostname resolution
route

# Show IPv6 routes
route -A inet6 -n
```

### Using `netstat` Command
```bash
# Show routing table
netstat -r
netstat -rn

# Show IPv6 routes
netstat -rn -A inet6
```

### Routing Table Files
```bash
# View kernel routing table
cat /proc/net/route

# View IPv6 routing table
cat /proc/net/ipv6_route

# Format: destination, gateway, netmask, flags, metric, ref, use, interface
```

---

## Managing Static Routes

### Using `ip route` Command

#### Add Routes
```bash
# Add route to network via gateway
ip route add 10.0.0.0/8 via 192.168.1.1

# Add route via specific interface
ip route add 10.0.0.0/8 dev eth1

# Add route with both gateway and interface
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0

# Add default gateway
ip route add default via 192.168.1.1
ip route add 0.0.0.0/0 via 192.168.1.1

# Add route with metric
ip route add 10.0.0.0/8 via 192.168.1.1 metric 100

# Add host route (single IP)
ip route add 10.0.0.5/32 via 192.168.1.1

# Add IPv6 route
ip -6 route add 2001:db8::/32 via 2001:db8::1

# Add IPv6 default gateway
ip -6 route add default via 2001:db8::1
```

#### Delete Routes
```bash
# Delete specific route
ip route del 10.0.0.0/8 via 192.168.1.1

# Delete default gateway
ip route del default

# Delete IPv6 route
ip -6 route del 2001:db8::/32
```

#### Replace Routes
```bash
# Replace existing route
ip route replace 10.0.0.0/8 via 192.168.1.2

# Replace default gateway
ip route replace default via 192.168.1.254
```

#### Flush Routes
```bash
# Flush all routes
ip route flush table main

# Flush routes to specific network
ip route flush 10.0.0.0/8

# Flush cached routes
ip route flush cache
```

### Using `route` Command (Legacy)

```bash
# Add route
route add -net 10.0.0.0/8 gw 192.168.1.1

# Add default gateway
route add default gw 192.168.1.1

# Add host route
route add -host 10.0.0.5 gw 192.168.1.1

# Delete route
route del -net 10.0.0.0/8

# Delete default gateway
route del default

# Add route via interface
route add -net 10.0.0.0/8 dev eth1
```

---

## Persistent Static Routes

### RHEL/CentOS/Fedora

#### Method 1: Network Scripts
Create route files: `/etc/sysconfig/network-scripts/route-<interface>`

```bash
# /etc/sysconfig/network-scripts/route-eth0
10.0.0.0/8 via 192.168.1.1
172.16.0.0/12 via 192.168.1.1 dev eth0
default via 192.168.1.254

# Format:
# network/prefix via gateway [dev interface] [metric N]
```

Alternative format:
```bash
# /etc/sysconfig/network-scripts/route-eth0
ADDRESS0=10.0.0.0
NETMASK0=255.0.0.0
GATEWAY0=192.168.1.1

ADDRESS1=172.16.0.0
NETMASK1=255.240.0.0
GATEWAY1=192.168.1.1
```

#### Method 2: NetworkManager

Using `nmcli`:
```bash
# Add static route to connection
nmcli connection modify eth0 +ipv4.routes "10.0.0.0/8 192.168.1.1"

# Add multiple routes
nmcli connection modify eth0 +ipv4.routes "10.0.0.0/8 192.168.1.1, 172.16.0.0/12 192.168.1.1"

# Add route with metric
nmcli connection modify eth0 +ipv4.routes "10.0.0.0/8 192.168.1.1 100"

# Remove route
nmcli connection modify eth0 -ipv4.routes "10.0.0.0/8 192.168.1.1"

# View routes
nmcli connection show eth0 | grep ipv4.routes

# Apply changes
nmcli connection up eth0

# IPv6 routes
nmcli connection modify eth0 +ipv6.routes "2001:db8::/32 2001:db8::1"
```

#### Method 3: NetworkManager Configuration Files
Edit: `/etc/NetworkManager/system-connections/<connection>.nmconnection`

```ini
[ipv4]
method=manual
address1=192.168.1.100/24,192.168.1.1
route1=10.0.0.0/8,192.168.1.1
route2=172.16.0.0/12,192.168.1.1,100
dns=8.8.8.8;8.8.4.4;

[ipv6]
method=manual
address1=2001:db8::100/64,2001:db8::1
route1=2001:db8:1::/48,2001:db8::1
```

Reload NetworkManager:
```bash
nmcli connection reload
nmcli connection up eth0
```

### Ubuntu/Debian (Netplan)

Edit: `/etc/netplan/01-netcfg.yaml`

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      routes:
        - to: 10.0.0.0/8
          via: 192.168.1.1
          metric: 100
        - to: 172.16.0.0/12
          via: 192.168.1.1
        - to: 0.0.0.0/0
          via: 192.168.1.254
          metric: 200
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

IPv6 example:
```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 2001:db8::100/64
      gateway6: 2001:db8::1
      routes:
        - to: 2001:db8:1::/48
          via: 2001:db8::1
```

Apply configuration:
```bash
netplan try      # Test configuration
netplan apply    # Apply configuration
```

### systemd-networkd

Configuration: `/etc/systemd/network/`

```ini
# /etc/systemd/network/20-wired.network
[Match]
Name=eth0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1

[Route]
Destination=10.0.0.0/8
Gateway=192.168.1.1
Metric=100

[Route]
Destination=172.16.0.0/12
Gateway=192.168.1.1

[Route]
Destination=0.0.0.0/0
Gateway=192.168.1.254
Metric=200
```

Restart service:
```bash
systemctl restart systemd-networkd
```

---

## Multiple Default Gateways

### Metric-Based Selection
```bash
# Primary default gateway (lower metric)
ip route add default via 192.168.1.1 metric 100

# Backup default gateway (higher metric)
ip route add default via 192.168.2.1 metric 200

# View
ip route show
```

### Multiple Gateways (Load Balancing)
```bash
# Equal-cost multi-path routing
ip route add default \
    nexthop via 192.168.1.1 dev eth0 weight 1 \
    nexthop via 192.168.2.1 dev eth1 weight 1
```

---

## Policy-Based Routing

### Routing Tables

#### View Routing Tables
```bash
# List all tables
cat /etc/iproute2/rt_tables

# Default tables:
# 0     unspec
# 253   default
# 254   main
# 255   local

# Add custom table
echo "100 custom" >> /etc/iproute2/rt_tables
```

#### Manage Routes in Custom Tables
```bash
# Add route to custom table
ip route add 10.0.0.0/8 via 192.168.1.1 table custom

# Add default gateway to custom table
ip route add default via 192.168.1.1 table custom

# Show routes in custom table
ip route show table custom

# Delete route from table
ip route del 10.0.0.0/8 table custom
```

### Routing Rules (Policy Routing)

#### View Rules
```bash
# Show routing rules
ip rule show
ip rule list

# Default rules:
# 0: from all lookup local
# 32766: from all lookup main
# 32767: from all lookup default
```

#### Add Rules

**Source-based routing:**
```bash
# Route traffic from specific source via custom table
ip rule add from 192.168.1.100 table custom priority 100

# Route traffic from network
ip rule add from 192.168.1.0/24 table custom priority 100
```

**Destination-based routing:**
```bash
# Route traffic to specific destination via custom table
ip rule add to 10.0.0.0/8 table custom priority 100
```

**Interface-based routing:**
```bash
# Route traffic arriving on interface
ip rule add iif eth0 table custom priority 100

# Route traffic leaving on interface
ip rule add oif eth1 table custom priority 100
```

**TOS-based routing:**
```bash
# Route based on Type of Service
ip rule add tos 0x10 table custom priority 100
```

**Fwmark-based routing:**
```bash
# Route based on firewall mark
ip rule add fwmark 1 table custom priority 100
```

**Combined rules:**
```bash
# Complex rule
ip rule add from 192.168.1.0/24 to 10.0.0.0/8 table custom priority 100
```

#### Delete Rules
```bash
# Delete by specification
ip rule del from 192.168.1.100 table custom

# Delete by priority
ip rule del priority 100

# Flush all rules (dangerous!)
ip rule flush
```

### Complete Policy Routing Example

**Scenario: Route traffic from different networks through different gateways**

```bash
# Create custom routing tables
echo "100 isp1" >> /etc/iproute2/rt_tables
echo "200 isp2" >> /etc/iproute2/rt_tables

# Add routes to tables
ip route add default via 10.0.1.1 table isp1
ip route add default via 10.0.2.1 table isp2

# Add routing rules
ip rule add from 192.168.1.0/24 table isp1 priority 100
ip rule add from 192.168.2.0/24 table isp2 priority 200

# Add routes for local networks in both tables
ip route add 192.168.1.0/24 dev eth1 table isp1
ip route add 192.168.2.0/24 dev eth2 table isp2

# Flush routing cache
ip route flush cache
```

---

## Source-Based Routing Example

**Route different users through different gateways:**

```bash
# Setup
echo "100 admin_table" >> /etc/iproute2/rt_tables

# Add default gateway for admin table
ip route add default via 192.168.1.1 table admin_table

# Add local network routes
ip route add 192.168.1.0/24 dev eth0 table admin_table

# Route admin user (192.168.1.100) through specific gateway
ip rule add from 192.168.1.100 table admin_table priority 100

# Verify
ip rule show
ip route show table admin_table
```

---

## Equal-Cost Multi-Path (ECMP) Routing

### Load Balancing Between Gateways
```bash
# Add multi-path default route
ip route add default \
    nexthop via 192.168.1.1 dev eth0 weight 1 \
    nexthop via 192.168.2.1 dev eth1 weight 1

# Unequal weight distribution (2:1 ratio)
ip route add default \
    nexthop via 192.168.1.1 dev eth0 weight 2 \
    nexthop via 192.168.2.1 dev eth1 weight 1

# View
ip route show
```

---

## Reverse Path Filtering

### Configure rp_filter
```bash
# Check current settings
cat /proc/sys/net/ipv4/conf/all/rp_filter
cat /proc/sys/net/ipv4/conf/eth0/rp_filter

# Values:
# 0 = No source validation
# 1 = Strict mode (recommended)
# 2 = Loose mode

# Set temporarily
echo 1 > /proc/sys/net/ipv4/conf/all/rp_filter

# Set permanently
echo "net.ipv4.conf.all.rp_filter = 1" >> /etc/sysctl.conf
echo "net.ipv4.conf.default.rp_filter = 1" >> /etc/sysctl.conf
sysctl -p
```

---

## Advanced Routing Features

### Nexthop Objects (Newer Kernels)
```bash
# Create nexthop object
ip nexthop add id 1 via 192.168.1.1 dev eth0
ip nexthop add id 2 via 192.168.2.1 dev eth1

# Create nexthop group
ip nexthop add id 10 group 1/2

# Use nexthop in route
ip route add 10.0.0.0/8 nhid 10

# View nexthops
ip nexthop show
```

### Route Metrics and Preferences
```bash
# Lower metric is preferred
ip route add 10.0.0.0/8 via 192.168.1.1 metric 10
ip route add 10.0.0.0/8 via 192.168.2.1 metric 20

# With both routes present, 192.168.1.1 is preferred
```

### Administrative Distance
Not directly configurable in Linux, but protocols have default preferences:
- Connected: 0
- Static: 1
- OSPF: 110
- RIP: 120

### Route Attributes
```bash
# Add route with specific attributes
ip route add 10.0.0.0/8 via 192.168.1.1 \
    metric 100 \
    mtu 1400 \
    advmss 1360

# Show route with all attributes
ip route show 10.0.0.0/8
```

---

## Troubleshooting Routing

### Verify Routes
```bash
# Check routing table
ip route show

# Test route to destination
ip route get 8.8.8.8
ip route get 10.0.0.5

# Check specific table
ip route show table custom

# Check rules
ip rule show
```

### Trace Route Path
```bash
# Traceroute
traceroute 8.8.8.8
traceroute -n 8.8.8.8  # No DNS resolution

# MTR (better)
mtr 8.8.8.8
mtr -n 8.8.8.8
```

### Check IP Forwarding
```bash
# Check if enabled
cat /proc/sys/net/ipv4/ip_forward

# Enable temporarily
echo 1 > /proc/sys/net/ipv4/ip_forward

# Enable permanently
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

### Debug Routing Issues
```bash
# Check interface status
ip link show
ip addr show

# Check if gateway is reachable
ping -c 4 192.168.1.1

# Check ARP table
ip neigh show
arp -n

# Monitor routing changes
ip monitor route

# Check routing cache (older kernels)
ip route show cache
```

### Common Issues

**Issue: No route to host**
```bash
# Add missing route
ip route add 10.0.0.0/8 via 192.168.1.1

# Or add default gateway
ip route add default via 192.168.1.1
```

**Issue: Asymmetric routing**
```bash
# May need to disable rp_filter
echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter

# Or use policy routing
```

**Issue: Routes not persisting**
```bash
# Add to configuration files
# RHEL: /etc/sysconfig/network-scripts/route-*
# Ubuntu: /etc/netplan/*.yaml
# Or use NetworkManager
```

---

## Routing with Multiple Interfaces

### Setup Routing Between Interfaces
```bash
# Enable IP forwarding
sysctl -w net.ipv4.ip_forward=1

# Add routes
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0
ip route add 172.16.0.0/12 via 192.168.2.1 dev eth1

# Default route
ip route add default via 192.168.1.1 dev eth0
```

### Interface-Specific Routing
```bash
# Force traffic out specific interface
ip route add 10.0.0.0/8 dev eth1
ip route add 172.16.0.0/12 dev eth0

# Source-based interface selection
ip rule add from 192.168.1.0/24 oif eth0
ip rule add from 192.168.2.0/24 oif eth1
```

---

## Monitoring Routing

### Real-Time Monitoring
```bash
# Monitor route changes
ip monitor route

# Monitor all IP events
ip monitor

# Monitor specific table
ip monitor route table custom

# Watch routing table
watch -n 1 'ip route show'
```

### Routing Statistics
```bash
# View route cache statistics
ip -s route show cache

# Interface statistics
ip -s link show eth0

# Routing protocol statistics (if running dynamic routing)
vtysh -c "show ip route"
```

---

## Quick Reference Commands

### View Routes
```bash
ip route show                          # Show routing table
ip route show table all                # All tables
ip route get 8.8.8.8                  # Route to destination
ip rule show                          # Show routing rules
```

### Add Routes
```bash
ip route add 10.0.0.0/8 via 192.168.1.1                    # Basic route
ip route add default via 192.168.1.1                       # Default gateway
ip route add 10.0.0.0/8 via 192.168.1.1 metric 100        # With metric
ip route add 10.0.0.0/8 via 192.168.1.1 table custom      # Custom table
```

### Policy Routing
```bash
echo "100 custom" >> /etc/iproute2/rt_tables              # Add table
ip route add default via 192.168.1.1 table custom         # Add route to table
ip rule add from 192.168.1.0/24 table custom priority 100 # Add rule
```

### Delete Routes
```bash
ip route del 10.0.0.0/8               # Delete route
ip route del default                   # Delete default
ip rule del priority 100               # Delete rule
```

### Persistent Configuration
```bash
# RHEL/CentOS
vim /etc/sysconfig/network-scripts/route-eth0
nmcli connection modify eth0 +ipv4.routes "10.0.0.0/8 192.168.1.1"

# Ubuntu
vim /etc/netplan/01-netcfg.yaml
netplan apply
```

---

## Practical Examples

### Example 1: Multi-Homed Host
```bash
# Host with two network connections
# eth0: 192.168.1.100/24 (ISP1 - gateway 192.168.1.1)
# eth1: 192.168.2.100/24 (ISP2 - gateway 192.168.2.1)

# Setup tables
echo "100 isp1" >> /etc/iproute2/rt_tables
echo "200 isp2" >> /etc/iproute2/rt_tables

# Add routes
ip route add default via 192.168.1.1 table isp1
ip route add default via 192.168.2.1 table isp2
ip route add 192.168.1.0/24 dev eth0 table isp1
ip route add 192.168.2.0/24 dev eth1 table isp2

# Add rules
ip rule add from 192.168.1.100 table isp1
ip rule add from 192.168.2.100 table isp2

# Main table default (for locally generated traffic)
ip route add default via 192.168.1.1 metric 100
ip route add default via 192.168.2.1 metric 200
```

### Example 2: VPN Routing
```bash
# Route specific traffic through VPN
# VPN interface: tun0, VPN gateway: 10.8.0.1

# Add route for specific network through VPN
ip route add 10.0.0.0/8 via 10.8.0.1 dev tun0

# Or use policy routing
echo "100 vpn" >> /etc/iproute2/rt_tables
ip route add default via 10.8.0.1 dev tun0 table vpn
ip rule add from 192.168.1.100 table vpn
```

### Example 3: DMZ Routing
```bash
# Router with three interfaces
# eth0: WAN (Internet) - 203.0.113.5
# eth1: LAN - 192.168.1.1/24
# eth2: DMZ - 10.0.0.1/24

# Enable forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Routes
ip route add default via 203.0.113.1 dev eth0
ip route add 192.168.1.0/24 dev eth1
ip route add 10.0.0.0/24 dev eth2

# Allow LAN to DMZ
# (firewall rules needed too)
```

---

## Exam Tips

- Know how to add/delete/modify routes with `ip route`
- Understand routing tables and policy-based routing
- Be familiar with persistent route configuration
- Know how to troubleshoot with `ip route get`
- Understand metrics and route selection
- Practice multi-path routing scenarios
- Know the difference between runtime and persistent routes
- Be comfortable with both RHEL and Debian-based configurations
- Understand source-based routing concepts
- Know how to verify routing with traceroute/mtr
- Remember to enable IP forwarding for routing between interfaces
- Practice reading and understanding routing table output
