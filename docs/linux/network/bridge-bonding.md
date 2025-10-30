# Configure Bridge and Bonding Devices

## Overview
This guide covers network bridge configuration (for virtualization and container networking) and network bonding/teaming (for redundancy and load balancing) in Linux.

---

## Network Bridges

### Bridge Concepts
A **network bridge** connects two or more network segments, operating at Layer 2 (Data Link layer). Common uses:
- Virtual machine networking
- Container networking (Docker, LXC)
- Connecting physical and virtual networks
- Network segmentation

**Key Points:**
- Bridges forward traffic based on MAC addresses
- All bridge members share the same broadcast domain
- Bridge itself can have an IP address
- Used extensively in virtualization (KVM, VirtualBox, etc.)

---

## Creating Network Bridges

### Method 1: Using `ip` Command (Temporary)

```bash
# Create bridge interface
ip link add name br0 type bridge

# Bring bridge up
ip link set br0 up

# Add interfaces to bridge
ip link set eth0 master br0
ip link set eth1 master br0

# Assign IP to bridge
ip addr add 192.168.1.100/24 dev br0

# View bridge
ip link show master br0    # Show bridge members
bridge link show           # Show bridge ports

# Remove interface from bridge
ip link set eth0 nomaster

# Delete bridge
ip link set br0 down
ip link delete br0
```

### Method 2: Using `brctl` Command (Legacy)

```bash
# Install bridge-utils
dnf install bridge-utils

# Create bridge
brctl addbr br0

# Add interfaces to bridge
brctl addif br0 eth0
brctl addif br0 eth1

# Remove interface
brctl delif br0 eth0

# Show bridge information
brctl show
brctl showmacs br0      # Show MAC addresses
brctl showstp br0       # Show STP info

# Delete bridge
brctl delbr br0
```

### Method 3: NetworkManager (Persistent)

```bash
# Create bridge connection
nmcli connection add type bridge \
    con-name br0 \
    ifname br0

# Configure bridge IP
nmcli connection modify br0 \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns 8.8.8.8 \
    ipv4.method manual

# Add slave interfaces
nmcli connection add type bridge-slave \
    con-name br0-slave-eth0 \
    ifname eth0 \
    master br0

nmcli connection add type bridge-slave \
    con-name br0-slave-eth1 \
    ifname eth1 \
    master br0

# Activate bridge
nmcli connection up br0
nmcli connection up br0-slave-eth0
nmcli connection up br0-slave-eth1

# View configuration
nmcli connection show br0
nmcli device status

# Modify bridge properties
nmcli connection modify br0 bridge.stp yes
nmcli connection modify br0 bridge.priority 32768
nmcli connection modify br0 bridge.forward-delay 15
nmcli connection modify br0 bridge.hello-time 2
nmcli connection modify br0 bridge.max-age 20

# Delete bridge
nmcli connection delete br0
nmcli connection delete br0-slave-eth0
```

### Method 4: Configuration Files (RHEL/CentOS)

Bridge interface: `/etc/sysconfig/network-scripts/ifcfg-br0`
```bash
DEVICE=br0
TYPE=Bridge
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
STP=yes
DELAY=0
```

Bridge member: `/etc/sysconfig/network-scripts/ifcfg-eth0`
```bash
DEVICE=eth0
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
BRIDGE=br0
```

Restart network:
```bash
systemctl restart NetworkManager
# or
nmcli connection reload
nmcli connection up br0
```

### Method 5: Netplan (Ubuntu/Debian)

Edit: `/etc/netplan/01-netcfg.yaml`

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
    eth1:
      dhcp4: no
  bridges:
    br0:
      interfaces:
        - eth0
        - eth1
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      parameters:
        stp: true
        forward-delay: 4
```

Apply:
```bash
netplan try
netplan apply
```

---

## Bridge Management and Monitoring

### View Bridge Information
```bash
# Using ip command
ip link show type bridge
bridge link show

# Show FDB (forwarding database)
bridge fdb show
bridge fdb show br br0

# Show VLAN info
bridge vlan show

# Using brctl
brctl show
brctl showmacs br0
brctl showstp br0
```

### Bridge Parameters

#### Spanning Tree Protocol (STP)
```bash
# Enable/disable STP (prevents loops)
ip link set br0 type bridge stp_state 1  # Enable
ip link set br0 type bridge stp_state 0  # Disable

# Using brctl
brctl stp br0 on
brctl stp br0 off

# Set bridge priority (lower = higher priority)
ip link set br0 type bridge priority 32768
brctl setbridgeprio br0 32768

# Set forward delay
ip link set br0 type bridge forward_delay 1500  # 15 seconds
brctl setfd br0 15

# Set hello time
ip link set br0 type bridge hello_time 200  # 2 seconds
brctl sethello br0 2

# Set max age
ip link set br0 type bridge max_age 2000  # 20 seconds
brctl setmaxage br0 20

# View STP info
bridge -d link show
```

#### Port Parameters
```bash
# Set port priority
bridge link set dev eth0 priority 10

# Set port path cost
bridge link set dev eth0 cost 100

# Disable learning on port
bridge link set dev eth0 learning off

# Disable flooding on port
bridge link set dev eth0 flood off
```

---

## Network Bonding (Link Aggregation)

### Bonding Concepts
**Network bonding** (also called NIC teaming) combines multiple network interfaces into a single logical interface for:
- **Redundancy**: Failover if one link fails
- **Load balancing**: Distribute traffic across links
- **Increased bandwidth**: Aggregate throughput (mode-dependent)

### Bonding Modes

**Mode 0 (balance-rr)**: Round-robin load balancing
- Packets transmitted sequentially on each slave
- Provides load balancing and fault tolerance
- Requires switch support (EtherChannel/LACP)

**Mode 1 (active-backup)**: Active-backup
- One slave active, others standby
- Provides fault tolerance only
- No switch configuration needed
- Most compatible

**Mode 2 (balance-xor)**: XOR load balancing
- Traffic distributed by source/destination MAC/IP
- Provides load balancing and fault tolerance
- May require switch configuration

**Mode 3 (broadcast)**: Broadcast
- All traffic transmitted on all slaves
- Provides fault tolerance
- Rarely used

**Mode 4 (802.3ad)**: IEEE 802.3ad LACP
- Dynamic link aggregation
- Requires switch support for LACP
- Provides load balancing and fault tolerance
- Recommended for most use cases

**Mode 5 (balance-tlb)**: Adaptive transmit load balancing
- Outgoing traffic balanced, incoming on one interface
- No switch configuration needed
- Provides load balancing and fault tolerance

**Mode 6 (balance-alb)**: Adaptive load balancing
- Both TX and RX load balancing
- No switch configuration needed
- Requires ethtool support

---

## Creating Network Bonds

### Method 1: Using `ip` Command (Temporary)

```bash
# Load bonding module
modprobe bonding

# Create bond interface
ip link add bond0 type bond mode active-backup

# Set bonding mode (if not set during creation)
ip link set bond0 type bond mode 802.3ad

# Add slaves
ip link set eth0 master bond0
ip link set eth1 master bond0

# Bring interfaces up
ip link set eth0 up
ip link set eth1 up
ip link set bond0 up

# Assign IP
ip addr add 192.168.1.100/24 dev bond0

# View bond info
cat /proc/net/bonding/bond0
ip link show bond0

# Remove slave
ip link set eth0 nomaster

# Delete bond
ip link set bond0 down
ip link delete bond0
```

### Method 2: NetworkManager (Persistent)

```bash
# Create bond connection
nmcli connection add type bond \
    con-name bond0 \
    ifname bond0 \
    mode active-backup

# Alternative modes:
# mode balance-rr (0)
# mode active-backup (1)
# mode balance-xor (2)
# mode broadcast (3)
# mode 802.3ad (4)
# mode balance-tlb (5)
# mode balance-alb (6)

# Configure IP
nmcli connection modify bond0 \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns 8.8.8.8 \
    ipv4.method manual

# Add slave interfaces
nmcli connection add type ethernet \
    con-name bond0-slave-eth0 \
    ifname eth0 \
    master bond0

nmcli connection add type ethernet \
    con-name bond0-slave-eth1 \
    ifname eth1 \
    master bond0

# Activate
nmcli connection up bond0
nmcli connection up bond0-slave-eth0
nmcli connection up bond0-slave-eth1

# View configuration
nmcli connection show bond0
nmcli device status

# Modify bonding parameters
nmcli connection modify bond0 \
    bond.options "mode=802.3ad,miimon=100,lacp_rate=fast"

# Common bonding options:
# miimon=100               # Link monitoring interval (ms)
# downdelay=200            # Delay before disabling slave
# updelay=200              # Delay before enabling slave
# lacp_rate=fast           # LACP rate (slow/fast)
# xmit_hash_policy=layer3+4  # Hash policy for mode 2/4
# primary=eth0             # Primary interface for mode 1
# arp_interval=250         # ARP monitoring interval
# arp_ip_target=192.168.1.1  # ARP target IP

# Delete bond
nmcli connection delete bond0
nmcli connection delete bond0-slave-eth0
nmcli connection delete bond0-slave-eth1
```

### Method 3: Configuration Files (RHEL/CentOS)

Bond interface: `/etc/sysconfig/network-scripts/ifcfg-bond0`
```bash
DEVICE=bond0
TYPE=Bond
BONDING_MASTER=yes
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
BONDING_OPTS="mode=active-backup miimon=100"
```

Slave interface: `/etc/sysconfig/network-scripts/ifcfg-eth0`
```bash
DEVICE=eth0
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

Slave interface: `/etc/sysconfig/network-scripts/ifcfg-eth1`
```bash
DEVICE=eth1
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

Load bonding module: `/etc/modprobe.d/bonding.conf`
```bash
alias bond0 bonding
options bonding mode=1 miimon=100
```

Restart network:
```bash
systemctl restart NetworkManager
```

### Method 4: Netplan (Ubuntu/Debian)

Edit: `/etc/netplan/01-netcfg.yaml`

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
    eth1:
      dhcp4: no
  bonds:
    bond0:
      interfaces:
        - eth0
        - eth1
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
      parameters:
        mode: active-backup
        mii-monitor-interval: 100
        # or for 802.3ad:
        # mode: 802.3ad
        # lacp-rate: fast
        # transmit-hash-policy: layer3+4
```

Apply:
```bash
netplan apply
```

---

## Bonding Configuration Options

### Common Bonding Parameters

```bash
# Bonding modes
mode=0              # balance-rr
mode=1              # active-backup
mode=2              # balance-xor
mode=4              # 802.3ad
mode=5              # balance-tlb
mode=6              # balance-alb

# Link monitoring
miimon=100          # MII link monitoring interval (ms)
arp_interval=250    # ARP monitoring interval (ms)
arp_ip_target=192.168.1.1  # Target for ARP monitoring

# Failover/failback timing
downdelay=200       # Delay before marking slave down (ms)
updelay=200         # Delay before marking slave up (ms)

# Mode-specific options
primary=eth0        # Primary interface (mode 1)
lacp_rate=fast      # LACP heartbeat rate (slow=30s, fast=1s)
xmit_hash_policy=layer3+4  # Hash policy (mode 2/4)
# Options: layer2, layer3+4, layer2+3, encap2+3, encap3+4

# Advanced
fail_over_mac=none  # MAC address handling
all_slaves_active=0 # Deliver packets to all slaves
```

### Set Bonding Options

```bash
# Using NetworkManager
nmcli connection modify bond0 \
    bond.options "mode=802.3ad,miimon=100,lacp_rate=fast,xmit_hash_policy=layer3+4"

# Using sysfs (temporary)
echo "fast" > /sys/class/net/bond0/bonding/lacp_rate
echo "layer3+4" > /sys/class/net/bond0/bonding/xmit_hash_policy
```

---

## Monitoring Bonds and Bridges

### Monitor Bonding Status

```bash
# View bond status
cat /proc/net/bonding/bond0

# View brief status
ip link show bond0

# Using nmcli
nmcli device show bond0

# Check slave status
cat /sys/class/net/bond0/bonding/slaves
cat /sys/class/net/bond0/bonding/active_slave

# Monitor in real-time
watch -n 1 'cat /proc/net/bonding/bond0'
```

### Monitor Bridge Status

```bash
# View bridge
bridge link show
ip link show master br0

# View MAC address table
bridge fdb show
bridge fdb show br br0

# View statistics
ip -s link show br0

# Monitor in real-time
watch -n 1 'bridge link show'
```

---

## Network Teaming (Alternative to Bonding)

### Teaming vs Bonding
- **teamd**: Modern alternative to bonding
- More flexible and feature-rich
- Better performance monitoring
- JSON configuration
- Active-backup, load balancing, LACP support

### Create Team with NetworkManager

```bash
# Create team
nmcli connection add type team \
    con-name team0 \
    ifname team0 \
    config '{"runner": {"name": "activebackup"}}'

# Alternative runners:
# "roundrobin"
# "activebackup"
# "loadbalance"
# "broadcast"
# "lacp"

# Configure IP
nmcli connection modify team0 \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.method manual

# Add team ports
nmcli connection add type team-slave \
    con-name team0-port1 \
    ifname eth0 \
    master team0

nmcli connection add type team-slave \
    con-name team0-port2 \
    ifname eth1 \
    master team0

# Activate
nmcli connection up team0

# View team status
teamdctl team0 state
```

### Team Configuration Examples

**Active-Backup:**
```json
{
    "runner": {
        "name": "activebackup"
    },
    "link_watch": {
        "name": "ethtool"
    }
}
```

**LACP:**
```json
{
    "runner": {
        "name": "lacp",
        "active": true,
        "fast_rate": true,
        "tx_hash": ["eth", "ipv4", "ipv6"]
    },
    "link_watch": {
        "name": "ethtool"
    }
}
```

**Load Balance:**
```json
{
    "runner": {
        "name": "loadbalance",
        "tx_hash": ["eth", "ipv4", "ipv6"]
    },
    "link_watch": {
        "name": "ethtool"
    }
}
```

---

## VLAN with Bridges and Bonds

### Bridge with VLAN
```bash
# Create VLAN interface
ip link add link eth0 name eth0.100 type vlan id 100

# Add to bridge
ip link set eth0.100 master br0

# Or using NetworkManager
nmcli connection add type vlan \
    con-name vlan100 \
    ifname eth0.100 \
    dev eth0 \
    id 100 \
    master br0
```

### Bond with VLAN
```bash
# Create VLAN on bond
ip link add link bond0 name bond0.100 type vlan id 100

# Configure VLAN
ip addr add 192.168.100.1/24 dev bond0.100
ip link set bond0.100 up

# Using NetworkManager
nmcli connection add type vlan \
    con-name vlan100 \
    ifname bond0.100 \
    dev bond0 \
    id 100
```

---

## Troubleshooting

### Bridge Troubleshooting

```bash
# Check bridge exists and is up
ip link show br0
brctl show

# Check interfaces in bridge
bridge link show
ip link show master br0

# Check MAC address table
bridge fdb show

# Check STP status
brctl showstp br0

# Verify IP configuration
ip addr show br0

# Check for errors
ip -s link show br0

# Test connectivity
ping -I br0 192.168.1.1
```

### Bond Troubleshooting

```bash
# Check bond status
cat /proc/net/bonding/bond0

# Verify mode
cat /sys/class/net/bond0/bonding/mode

# Check active slave
cat /sys/class/net/bond0/bonding/active_slave

# Check all slaves
cat /sys/class/net/bond0/bonding/slaves

# Check MII status
cat /sys/class/net/bond0/bonding/mii_status

# View statistics
ip -s link show bond0

# Check slave status individually
ethtool eth0
ethtool eth1

# Test failover (for testing only)
ip link set eth0 down
cat /proc/net/bonding/bond0
ip link set eth0 up
```

### Common Issues

**Bridge not forwarding traffic:**
```bash
# Enable forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Check iptables
iptables -L FORWARD

# Disable netfilter on bridge (if needed)
echo 0 > /proc/sys/net/bridge/bridge-nf-call-iptables
```

**Bond slaves not activating:**
```bash
# Check interfaces are up
ip link set eth0 up
ip link set eth1 up

# Verify no IP on slaves
ip addr flush dev eth0
ip addr flush dev eth1

# Check for conflicting configurations
nmcli connection show
```

**LACP not negotiating:**
```bash
# Verify switch configuration
# Check LACP is enabled on switch
# Verify correct mode (802.3ad)
cat /proc/net/bonding/bond0 | grep -A 10 "802.3ad"

# Check LACP rate
cat /sys/class/net/bond0/bonding/lacp_rate
```

---

## Practical Examples

### Example 1: KVM Bridge
```bash
# Create bridge for KVM VMs
nmcli connection add type bridge \
    con-name br0 \
    ifname br0 \
    ipv4.method disabled \
    ipv6.method disabled

nmcli connection add type bridge-slave \
    con-name br0-port1 \
    ifname eth0 \
    master br0

nmcli connection up br0
```

### Example 2: Redundant Server Network
```bash
# Active-backup bond with two NICs
nmcli connection add type bond \
    con-name bond0 \
    ifname bond0 \
    mode active-backup \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.method manual

nmcli connection add type ethernet \
    slave-type bond \
    con-name bond0-eth0 \
    ifname eth0 \
    master bond0

nmcli connection add type ethernet \
    slave-type bond \
    con-name bond0-eth1 \
    ifname eth1 \
    master bond0

nmcli connection modify bond0 \
    bond.options "mode=active-backup,miimon=100,primary=eth0"

nmcli connection up bond0
```

### Example 3: LACP Bond for High Bandwidth
```bash
# 802.3ad LACP bond
nmcli connection add type bond \
    con-name bond0 \
    ifname bond0 \
    mode 802.3ad \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.method manual

nmcli connection modify bond0 \
    bond.options "mode=802.3ad,miimon=100,lacp_rate=fast,xmit_hash_policy=layer3+4"

nmcli connection add type ethernet \
    slave-type bond \
    ifname eth0 \
    master bond0

nmcli connection add type ethernet \
    slave-type bond \
    ifname eth1 \
    master bond0

nmcli connection up bond0
```

---

## Quick Reference

### Bridge Commands
```bash
ip link add br0 type bridge                    # Create bridge
ip link set eth0 master br0                    # Add interface
bridge link show                               # Show bridge ports
brctl show                                     # Show bridges
ip link delete br0                             # Delete bridge
```

### Bond Commands
```bash
ip link add bond0 type bond mode active-backup # Create bond
ip link set eth0 master bond0                  # Add slave
cat /proc/net/bonding/bond0                    # Show status
ip link delete bond0                           # Delete bond
```

### NetworkManager
```bash
nmcli connection add type bridge con-name br0 ifname br0
nmcli connection add type bridge-slave ifname eth0 master br0
nmcli connection add type bond con-name bond0 mode active-backup
nmcli connection add type ethernet master bond0 ifname eth0
```

---

## Exam Tips

- Know how to create bridges and bonds with NetworkManager
- Understand different bonding modes and when to use each
- Be familiar with both temporary (`ip`) and persistent (nmcli/files) configuration
- Know how to troubleshoot bond and bridge issues
- Understand LACP requirements and configuration
- Practice viewing status with `/proc/net/bonding/` and `bridge` commands
- Know the difference between bonding and teaming
- Understand when bridges are needed (VMs, containers)
- Be comfortable with both RHEL and Ubuntu configurations
- Know how to verify link status and failover behavior
