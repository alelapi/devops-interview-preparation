# Monitor and Troubleshoot Networking

## Overview
This guide covers essential tools and techniques for monitoring network performance, diagnosing connectivity issues, and troubleshooting network problems in Linux.

---

## Basic Connectivity Testing

### `ping` Command
Tests basic connectivity using ICMP Echo Request/Reply.

```bash
# Basic ping
ping 8.8.8.8

# Ping with count
ping -c 4 google.com

# Ping with specific interval (default 1 second)
ping -i 2 192.168.1.1

# Ping with specific packet size
ping -s 1000 google.com

# Ping with timeout
ping -W 2 192.168.1.1

# Ping IPv6
ping6 2001:4860:4860::8888
ping6 -c 4 google.com

# Flood ping (requires root)
ping -f 192.168.1.1

# Don't fragment packets
ping -M do -s 1472 192.168.1.1
```

Options explained:
- `-c`: Count (number of packets)
- `-i`: Interval between packets
- `-s`: Packet size
- `-W`: Timeout
- `-f`: Flood mode
- `-M`: MTU discovery strategy

---

## Network Interface Monitoring

### `ip` Command
```bash
# Show all interfaces with statistics
ip -s link show

# Show specific interface statistics
ip -s -s link show eth0

# Show interface errors
ip -s link show eth0 | grep -E "RX|TX|errors|dropped"

# Monitor in real-time
watch -n 1 'ip -s link show eth0'

# Show ARP cache
ip neighbour show
ip neigh show

# Flush ARP cache
ip neighbour flush all
```

### `ifconfig` (Legacy, but still useful)
```bash
# Show all interfaces
ifconfig -a

# Show specific interface
ifconfig eth0

# Show statistics
ifconfig eth0 | grep -E "RX|TX"
```

### Interface Statistics Files
```bash
# View interface statistics via /sys
cat /sys/class/net/eth0/statistics/rx_packets
cat /sys/class/net/eth0/statistics/tx_packets
cat /sys/class/net/eth0/statistics/rx_errors
cat /sys/class/net/eth0/statistics/tx_errors
cat /sys/class/net/eth0/statistics/collisions

# View all statistics for interface
ls /sys/class/net/eth0/statistics/
```

---

## Routing and Network Path

### `ip route` Command
```bash
# Show routing table
ip route show

# Show IPv6 routing table
ip -6 route show

# Show routing table with details
ip route show table all

# Show route to specific destination
ip route get 8.8.8.8
ip route get 2001:4860:4860::8888

# Show routing cache (deprecated in newer kernels)
ip route show cache
```

### `traceroute` Command
Shows the path packets take to reach destination.

```bash
# Basic traceroute
traceroute google.com

# Traceroute with no DNS resolution
traceroute -n 8.8.8.8

# Traceroute using ICMP instead of UDP
traceroute -I google.com

# Traceroute using TCP SYN
traceroute -T -p 80 google.com

# Set maximum hops
traceroute -m 20 google.com

# Set number of queries per hop
traceroute -q 3 google.com

# IPv6 traceroute
traceroute6 google.com

# MTU path discovery
traceroute --mtu google.com
```

### `tracepath` Command
Similar to traceroute but doesn't require root privileges.

```bash
# Basic tracepath
tracepath google.com

# IPv6 tracepath
tracepath6 google.com

# Set initial packet length
tracepath -l 1400 google.com
```

### `mtr` Command
Combines ping and traceroute functionality with real-time updates.

```bash
# Interactive mode
mtr google.com

# Report mode (non-interactive)
mtr -r -c 10 google.com

# No DNS resolution
mtr -n google.com

# Show both hostnames and IP addresses
mtr -b google.com

# Wide report mode
mtr -w google.com

# CSV output
mtr --csv google.com

# JSON output
mtr --json google.com

# Set packet size
mtr -s 1000 google.com
```

---

## Port and Service Connectivity

### `telnet` Command
Test TCP connectivity to specific ports.

```bash
# Test HTTP port
telnet google.com 80

# Test HTTPS port
telnet google.com 443

# Test SSH port
telnet 192.168.1.100 22

# Exit telnet: Ctrl+] then type 'quit'
```

### `nc` (netcat) Command
Swiss Army knife for network testing.

```bash
# Test TCP connection
nc -vz google.com 80

# Test UDP connection
nc -vzu 8.8.8.8 53

# Scan range of ports
nc -vz google.com 80-443

# Listen on a port
nc -l 8080

# Connect and send data
echo "GET / HTTP/1.0" | nc google.com 80

# Transfer file
# On receiver:
nc -l 9999 > received_file
# On sender:
nc receiver_ip 9999 < file_to_send

# Port scanning
nc -zv 192.168.1.1 20-80

# Test with timeout
nc -w 3 -vz google.com 80
```

### `nmap` Command
Powerful network scanning and port discovery tool.

```bash
# Scan single host
nmap 192.168.1.1

# Scan with service detection
nmap -sV 192.168.1.1

# Scan specific ports
nmap -p 22,80,443 192.168.1.1

# Scan port range
nmap -p 1-1000 192.168.1.1

# Scan all ports
nmap -p- 192.168.1.1

# Fast scan (top 100 ports)
nmap -F 192.168.1.1

# Scan subnet
nmap 192.168.1.0/24

# OS detection
nmap -O 192.168.1.1

# Aggressive scan
nmap -A 192.168.1.1

# TCP SYN scan (stealth)
nmap -sS 192.168.1.1

# UDP scan
nmap -sU 192.168.1.1

# Save output
nmap -oN output.txt 192.168.1.1
nmap -oX output.xml 192.168.1.1
```

---

## Network Statistics and Connections

### `ss` Command (Socket Statistics)
Modern replacement for netstat, showing socket information.

```bash
# Show all sockets
ss -a

# Show listening TCP sockets
ss -lt

# Show listening UDP sockets
ss -lu

# Show all TCP connections
ss -t

# Show all UDP connections
ss -u

# Show process using socket
ss -p

# Show summary statistics
ss -s

# Show sockets with numeric ports
ss -n

# Combine options
ss -tulpn

# Show TCP sockets in listening state
ss -tln

# Show established connections
ss -t state established

# Show connections to specific port
ss -tn sport = :80
ss -tn dport = :443

# Show connections to specific IP
ss dst 192.168.1.100

# Show socket memory usage
ss -m

# Show timer information
ss -o

# Extended socket info
ss -e

# Show both IPv4 and IPv6
ss -46tulpn
```

### `netstat` Command (Legacy)
```bash
# Show all listening ports
netstat -tuln

# Show all connections with process
netstat -tulpn

# Show routing table
netstat -r

# Show interface statistics
netstat -i

# Show network statistics
netstat -s

# Continuous monitoring
netstat -c

# Show only TCP
netstat -t

# Show only UDP
netstat -u

# Show listening sockets
netstat -l

# Show all (listening and non-listening)
netstat -a
```

---

## Packet Capture and Analysis

### `tcpdump` Command
Capture and analyze network packets.

```bash
# Capture on specific interface
tcpdump -i eth0

# Capture specific number of packets
tcpdump -c 100 -i eth0

# Capture and save to file
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Capture with verbose output
tcpdump -v -i eth0
tcpdump -vv -i eth0
tcpdump -vvv -i eth0

# Show packet contents in hex and ASCII
tcpdump -X -i eth0

# Capture specific host
tcpdump -i eth0 host 192.168.1.100

# Capture specific port
tcpdump -i eth0 port 80

# Capture specific protocol
tcpdump -i eth0 icmp
tcpdump -i eth0 tcp
tcpdump -i eth0 udp

# Capture source or destination
tcpdump -i eth0 src 192.168.1.100
tcpdump -i eth0 dst 192.168.1.100

# Capture network range
tcpdump -i eth0 net 192.168.1.0/24

# Complex filters
tcpdump -i eth0 'tcp port 80 and src 192.168.1.100'
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'

# Capture DNS queries
tcpdump -i eth0 -n port 53

# Capture HTTP traffic
tcpdump -i eth0 -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# Don't resolve hostnames
tcpdump -n -i eth0

# Don't resolve hostnames or ports
tcpdump -nn -i eth0

# Capture on all interfaces
tcpdump -i any

# Set snapshot length
tcpdump -s 0 -i eth0  # Full packet
tcpdump -s 96 -i eth0 # First 96 bytes

# Rotate capture files
tcpdump -i eth0 -w capture.pcap -C 100 -W 5
```

Common filters:
- `host X`: Traffic to/from host X
- `src X`: Traffic from X
- `dst X`: Traffic to X
- `net X`: Traffic to/from network X
- `port X`: Traffic on port X
- `portrange X-Y`: Traffic on port range
- `less/greater X`: Packet size less/greater than X
- `tcp/udp/icmp`: Specific protocol

---

## DNS Troubleshooting

### `dig` Command
```bash
# Basic query
dig example.com

# Query specific record type
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com NS
dig example.com TXT
dig example.com SOA

# Query specific DNS server
dig @8.8.8.8 example.com

# Short answer only
dig +short example.com

# Reverse DNS lookup
dig -x 8.8.8.8

# Trace DNS resolution
dig +trace example.com

# Show query time
dig example.com +stats

# Show all information
dig example.com ANY

# Disable recursion
dig +norecurse example.com
```

### `nslookup` Command
```bash
# Basic query
nslookup example.com

# Query specific server
nslookup example.com 8.8.8.8

# Reverse lookup
nslookup 8.8.8.8

# Interactive mode
nslookup
> server 8.8.8.8
> set type=MX
> example.com
> exit
```

### `host` Command
```bash
# Basic lookup
host example.com

# Specific record type
host -t A example.com
host -t MX example.com
host -t NS example.com

# Reverse lookup
host 8.8.8.8

# Verbose output
host -v example.com

# Query specific server
host example.com 8.8.8.8
```

---

## Network Performance Testing

### `iperf3` Command
Network bandwidth testing tool.

Server side:
```bash
# Start server
iperf3 -s

# Start server on specific port
iperf3 -s -p 5201

# Server with JSON output
iperf3 -s -J
```

Client side:
```bash
# Basic test
iperf3 -c server_ip

# Test with specific duration
iperf3 -c server_ip -t 30

# Test with specific bandwidth
iperf3 -c server_ip -b 100M

# Reverse mode (server sends)
iperf3 -c server_ip -R

# Bidirectional test
iperf3 -c server_ip --bidir

# UDP test
iperf3 -c server_ip -u

# Parallel streams
iperf3 -c server_ip -P 4

# JSON output
iperf3 -c server_ip -J

# Test specific port
iperf3 -c server_ip -p 5201
```

### `curl` Command for HTTP Testing
```bash
# Basic request with timing
curl -w "@-" -o /dev/null -s https://example.com << 'EOF'
    time_namelookup:  %{time_namelookup}\n
       time_connect:  %{time_connect}\n
    time_appconnect:  %{time_appconnect}\n
   time_pretransfer:  %{time_pretransfer}\n
      time_redirect:  %{time_redirect}\n
 time_starttransfer:  %{time_starttransfer}\n
                    ----------\n
         time_total:  %{time_total}\n
EOF

# Test connection speed
curl -o /dev/null https://example.com/file

# Show only headers
curl -I https://example.com

# Follow redirects with timing
curl -L -w "Total time: %{time_total}s\n" https://example.com

# Test with specific timeout
curl --connect-timeout 5 https://example.com
```

---

## Log Analysis for Network Issues

### System Logs
```bash
# General system logs
journalctl -xe

# Network-related logs
journalctl -u NetworkManager
journalctl -u systemd-networkd
journalctl -u chronyd

# Follow logs in real-time
journalctl -f

# Show kernel messages
dmesg | grep -i eth
dmesg | grep -i network
dmesg | tail -50

# Traditional log files
tail -f /var/log/messages
tail -f /var/log/syslog
grep -i network /var/log/messages
```

### Connection Tracking
```bash
# Show connection tracking table (conntrack)
conntrack -L

# Show statistics
conntrack -S

# Monitor new connections
conntrack -E

# Count connections
conntrack -C
```

---

## Bandwidth Monitoring

### `iftop` Command
Real-time bandwidth monitoring per connection.

```bash
# Monitor specific interface
iftop -i eth0

# Don't resolve hostnames
iftop -n

# Don't resolve port numbers
iftop -N

# Show ports
iftop -P

# Text mode (no curses)
iftop -t

# Filter by network
iftop -F 192.168.1.0/24
```

### `nethogs` Command
Bandwidth usage per process.

```bash
# Monitor all interfaces
nethogs

# Monitor specific interface
nethogs eth0

# Don't resolve hostnames
nethogs -v 0

# Trace mode (no curses)
nethogs -t
```

### `vnstat` Command
Network traffic logger and monitor.

```bash
# Show statistics for all interfaces
vnstat

# Show specific interface
vnstat -i eth0

# Live monitoring
vnstat -l -i eth0

# Show hourly stats
vnstat -h -i eth0

# Show daily stats
vnstat -d -i eth0

# Show monthly stats
vnstat -m -i eth0

# Show top days
vnstat -t -i eth0

# JSON output
vnstat --json

# Initialize database for interface
vnstat -u -i eth0
```

### `bmon` Command
Bandwidth monitoring with graphical output.

```bash
# Monitor all interfaces
bmon

# Monitor specific interface
bmon -p eth0

# Set update interval
bmon -r 1

# Show bits instead of bytes
bmon -b
```

---

## Network File Systems Monitoring

### `nfsstat` Command
```bash
# Show NFS statistics
nfsstat

# Show client statistics
nfsstat -c

# Show server statistics
nfsstat -s

# Show all statistics
nfsstat -a

# Show statistics with timestamps
watch -n 5 nfsstat
```

### `showmount` Command
```bash
# Show NFS exports
showmount -e nfs_server

# Show mounted directories
showmount -d nfs_server

# Show all mount points
showmount -a nfs_server
```

---

## Wireless Network Monitoring

### `iwconfig` Command
```bash
# Show wireless interfaces
iwconfig

# Show specific interface
iwconfig wlan0

# Set wireless parameters
iwconfig wlan0 essid "NetworkName"
iwconfig wlan0 key s:password
```

### `iw` Command (Modern)
```bash
# Show wireless devices
iw dev

# Show wireless info
iw dev wlan0 info

# Scan for networks
iw dev wlan0 scan

# Show link status
iw dev wlan0 link

# Show station info
iw dev wlan0 station dump
```

---

## Troubleshooting Workflow

### Step-by-Step Network Troubleshooting

1. **Check Physical Layer**
```bash
# Check if interface is up
ip link show eth0

# Check cable connection
ethtool eth0 | grep "Link detected"
```

2. **Check IP Configuration**
```bash
# Verify IP address
ip addr show eth0

# Check for DHCP lease (if using DHCP)
dhclient -v eth0
```

3. **Check Local Connectivity**
```bash
# Ping gateway
ping -c 4 192.168.1.1

# Check ARP resolution
ip neigh show
```

4. **Check DNS Resolution**
```bash
# Test DNS
dig google.com
nslookup google.com

# Check resolv.conf
cat /etc/resolv.conf
```

5. **Check Routing**
```bash
# Verify default gateway
ip route show

# Test route to destination
ip route get 8.8.8.8

# Traceroute to destination
traceroute 8.8.8.8
```

6. **Check Remote Connectivity**
```bash
# Ping external host
ping -c 4 8.8.8.8

# Test specific service
nc -vz google.com 443
```

7. **Check Firewall Rules**
```bash
# Check firewall status
firewall-cmd --list-all
iptables -L -n -v
```

8. **Check Services and Ports**
```bash
# Check listening ports
ss -tulpn

# Check specific service
systemctl status NetworkManager
```

---

## Common Network Issues and Solutions

### Issue: No Network Connectivity
```bash
# Check interface status
ip link show

# Bring interface up
ip link set eth0 up

# Restart NetworkManager
systemctl restart NetworkManager

# Check for DHCP
dhclient -v eth0
```

### Issue: DNS Not Resolving
```bash
# Check DNS servers
cat /etc/resolv.conf

# Test DNS manually
dig @8.8.8.8 google.com

# Flush DNS cache (systemd-resolved)
resolvectl flush-caches

# Restart DNS service
systemctl restart systemd-resolved
```

### Issue: Slow Network Performance
```bash
# Check interface errors
ip -s link show eth0

# Check bandwidth usage
iftop -i eth0

# Check MTU
ip link show eth0 | grep mtu

# Test bandwidth
iperf3 -c server_ip

# Check for packet loss
ping -c 100 8.8.8.8 | grep loss
```

### Issue: Intermittent Connectivity
```bash
# Monitor in real-time
mtr google.com

# Check for errors
dmesg | grep -i eth

# Monitor connections
watch -n 1 'ss -s'

# Check logs
journalctl -u NetworkManager -f
```

---

## Performance Metrics to Monitor

1. **Bandwidth**: Current usage vs. available capacity
2. **Latency**: Round-trip time (ping)
3. **Packet Loss**: Percentage of lost packets
4. **Throughput**: Actual data transfer rate
5. **Connection Count**: Number of active connections
6. **Errors**: Interface errors, collisions, drops
7. **DNS Resolution Time**: Time to resolve hostnames
8. **MTU**: Maximum transmission unit issues

---

## Quick Reference Commands

### Connectivity
```bash
ping -c 4 8.8.8.8             # Test connectivity
traceroute google.com         # Trace route
mtr google.com                # Combined ping/traceroute
```

### Interfaces and Routing
```bash
ip addr show                  # Show IP addresses
ip link show                  # Show interfaces
ip route show                 # Show routing table
ip route get 8.8.8.8         # Show route to destination
```

### Connections and Ports
```bash
ss -tulpn                     # Show listening ports
ss -t state established       # Show established TCP
nc -vz host 80                # Test port connectivity
nmap -p 80,443 host           # Scan specific ports
```

### Performance and Monitoring
```bash
iftop -i eth0                 # Real-time bandwidth
nethogs                       # Bandwidth per process
vnstat -l -i eth0             # Live traffic stats
iperf3 -c server              # Bandwidth test
```

### Packet Analysis
```bash
tcpdump -i eth0 port 80       # Capture HTTP traffic
tcpdump -nn -i eth0           # Capture without DNS lookup
tcpdump -r file.pcap          # Read capture file
```

### DNS
```bash
dig google.com                # DNS query
nslookup google.com           # Simple DNS lookup
host google.com               # Quick DNS lookup
```

---

## Exam Tips

- Know the difference between `ss` and `netstat` (prefer `ss`)
- Practice using `tcpdump` with various filters
- Understand how to read `mtr` output
- Be comfortable with both `ip` and legacy commands
- Know how to test connectivity at each OSI layer
- Practice troubleshooting methodology systematically
- Understand common port numbers (22, 80, 443, 53, etc.)
- Know how to interpret network statistics and errors
- Be able to identify bottlenecks and performance issues
- Practice reading and analyzing packet captures
