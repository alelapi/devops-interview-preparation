# Implement Reverse Proxies and Load Balancers

## Overview
This guide covers implementation of reverse proxies and load balancers using HAProxy, Nginx, and Apache in Linux systems.

---

## Concepts

### Reverse Proxy
A **reverse proxy** sits in front of web servers and forwards client requests to those servers. Benefits:
- SSL/TLS termination
- Caching
- Compression
- Security (hide backend servers)
- Single entry point

### Load Balancer
A **load balancer** distributes incoming traffic across multiple servers. Benefits:
- High availability
- Scalability
- Redundancy
- Performance optimization

### Key Terms
- **Frontend**: Entry point that receives client requests
- **Backend**: Pool of servers that handle requests
- **Health Check**: Monitoring to verify server availability
- **Session Persistence**: Maintaining user session on same server (sticky sessions)
- **Load Balancing Algorithm**: Method to distribute traffic

### Load Balancing Algorithms
- **Round Robin**: Sequential distribution
- **Least Connections**: Server with fewest active connections
- **IP Hash**: Based on client IP address
- **Weighted**: Servers have different capacities
- **Least Response Time**: Fastest responding server

---

## HAProxy (High Availability Proxy)

### Overview
HAProxy is a popular open-source load balancer and proxy server for TCP and HTTP-based applications.

### Installation
```bash
# RHEL/CentOS/Fedora
dnf install haproxy

# Ubuntu/Debian
apt install haproxy

# Start service
systemctl start haproxy
systemctl enable haproxy

# Check status
systemctl status haproxy
```

### Configuration File
Location: `/etc/haproxy/haproxy.cfg`

### Basic HAProxy Configuration Structure

```bash
# Global settings
global
    # Process management
    # Logging
    # Performance tuning

# Default settings for all sections
defaults
    # Timeouts
    # Mode (tcp/http)
    # Options

# Frontend - entry point
frontend <name>
    # Bind address and port
    # ACLs
    # Backend selection

# Backend - server pool
backend <name>
    # Load balancing algorithm
    # Health checks
    # Server definitions
```

### Simple HTTP Load Balancer Example

```bash
# /etc/haproxy/haproxy.cfg

global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

frontend http_front
    bind *:80
    stats uri /haproxy?stats
    default_backend http_back

backend http_back
    balance roundrobin
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
    server web3 192.168.1.12:80 check
```

Test and reload:
```bash
# Test configuration
haproxy -c -f /etc/haproxy/haproxy.cfg

# Reload without dropping connections
systemctl reload haproxy

# Restart
systemctl restart haproxy
```

### Advanced HAProxy Configuration

#### SSL/TLS Termination
```bash
frontend https_front
    bind *:443 ssl crt /etc/haproxy/certs/cert.pem
    bind *:80
    redirect scheme https code 301 if !{ ssl_fc }
    default_backend web_back

backend web_back
    balance roundrobin
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```

#### Health Checks
```bash
backend web_back
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    
    server web1 192.168.1.10:80 check inter 2000 rise 2 fall 3
    server web2 192.168.1.11:80 check inter 2000 rise 2 fall 3
    
# Parameters:
# check      - Enable health checks
# inter 2000 - Check interval (2 seconds)
# rise 2     - Consider server up after 2 successful checks
# fall 3     - Consider server down after 3 failed checks
```

#### Load Balancing Algorithms
```bash
backend web_back
    # Round robin (default)
    balance roundrobin
    
    # Or least connections
    # balance leastconn
    
    # Or source IP hash
    # balance source
    
    # Or URI hash
    # balance uri
    
    # Or URL parameter hash
    # balance url_param userid
    
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```

#### Weighted Load Balancing
```bash
backend web_back
    balance roundrobin
    server web1 192.168.1.10:80 check weight 100
    server web2 192.168.1.11:80 check weight 50
    server web3 192.168.1.12:80 check weight 150
    # web3 gets 1.5x more traffic than web1
```

#### Session Persistence (Sticky Sessions)
```bash
backend web_back
    balance roundrobin
    # Cookie-based persistence
    cookie SERVERID insert indirect nocache
    server web1 192.168.1.10:80 check cookie web1
    server web2 192.168.1.11:80 check cookie web2
    
    # Or source IP based
    # stick-table type ip size 200k expire 30m
    # stick on src
```

#### ACLs (Access Control Lists)
```bash
frontend http_front
    bind *:80
    
    # Define ACLs
    acl is_api path_beg /api
    acl is_admin path_beg /admin
    acl is_static path_end .jpg .png .css .js
    acl allowed_ips src 192.168.1.0/24 10.0.0.0/8
    
    # Use ACLs
    use_backend api_back if is_api
    use_backend admin_back if is_admin allowed_ips
    use_backend static_back if is_static
    default_backend web_back

backend api_back
    server api1 192.168.1.20:8080 check

backend admin_back
    server admin1 192.168.1.30:8080 check

backend static_back
    server static1 192.168.1.40:80 check

backend web_back
    server web1 192.168.1.10:80 check
```

#### HTTP Headers Manipulation
```bash
frontend http_front
    bind *:80
    # Add headers
    http-request add-header X-Forwarded-Proto https
    http-request set-header X-Client-IP %[src]
    default_backend web_back

backend web_back
    # Add backend headers
    http-request add-header X-Backend-Server %[srv_name]
    # Remove headers
    http-response del-header Server
    server web1 192.168.1.10:80 check
```

#### Rate Limiting
```bash
frontend http_front
    bind *:80
    # Track client requests
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    # Deny if more than 100 requests in 10 seconds
    http-request deny if { sc_http_req_rate(0) gt 100 }
    default_backend web_back
```

### HAProxy Statistics Page
```bash
frontend http_front
    bind *:80
    # Enable stats
    stats enable
    stats uri /haproxy-stats
    stats auth admin:password
    stats refresh 30s
    default_backend web_back
```

Access: http://your-server/haproxy-stats

### HAProxy Runtime API
```bash
# Connect to admin socket
echo "show stat" | socat stdio /run/haproxy/admin.sock

# Show servers
echo "show servers state" | socat stdio /run/haproxy/admin.sock

# Disable server
echo "disable server web_back/web1" | socat stdio /run/haproxy/admin.sock

# Enable server
echo "enable server web_back/web1" | socat stdio /run/haproxy/admin.sock

# Set server weight
echo "set weight web_back/web1 50" | socat stdio /run/haproxy/admin.sock
```

---

## Nginx as Reverse Proxy and Load Balancer

### Installation
```bash
# RHEL/CentOS/Fedora
dnf install nginx

# Ubuntu/Debian
apt install nginx

# Start service
systemctl start nginx
systemctl enable nginx
```

### Basic Reverse Proxy Configuration

```nginx
# /etc/nginx/nginx.conf or /etc/nginx/conf.d/proxy.conf

http {
    upstream backend {
        server 192.168.1.10:80;
        server 192.168.1.11:80;
        server 192.168.1.12:80;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

Test and reload:
```bash
# Test configuration
nginx -t

# Reload
systemctl reload nginx

# Restart
systemctl restart nginx
```

### Load Balancing Methods

```nginx
# Round robin (default)
upstream backend {
    server 192.168.1.10:80;
    server 192.168.1.11:80;
}

# Least connections
upstream backend {
    least_conn;
    server 192.168.1.10:80;
    server 192.168.1.11:80;
}

# IP hash (session persistence)
upstream backend {
    ip_hash;
    server 192.168.1.10:80;
    server 192.168.1.11:80;
}

# Weighted load balancing
upstream backend {
    server 192.168.1.10:80 weight=3;
    server 192.168.1.11:80 weight=1;
    server 192.168.1.12:80 weight=2;
}

# Hash-based (generic)
upstream backend {
    hash $request_uri consistent;
    server 192.168.1.10:80;
    server 192.168.1.11:80;
}
```

### Health Checks

```nginx
upstream backend {
    server 192.168.1.10:80 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:80 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:80 backup;  # Backup server
}

# max_fails: Number of failed attempts before marking down
# fail_timeout: Time to consider server down
# backup: Only used when other servers are down
```

### SSL/TLS Termination

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
```

### Caching

```nginx
# Define cache path
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;

server {
    listen 80;
    
    location / {
        proxy_cache my_cache;
        proxy_cache_valid 200 60m;
        proxy_cache_valid 404 10m;
        proxy_cache_key "$scheme$request_method$host$request_uri";
        add_header X-Cache-Status $upstream_cache_status;
        
        proxy_pass http://backend;
    }
}
```

### Advanced Configuration

```nginx
http {
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
    
    # Connection limiting
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    
    upstream backend {
        least_conn;
        server 192.168.1.10:80 weight=3 max_fails=2 fail_timeout=30s;
        server 192.168.1.11:80 weight=2 max_fails=2 fail_timeout=30s;
        server 192.168.1.12:80 backup;
        
        # Keepalive connections
        keepalive 32;
    }

    server {
        listen 80;
        server_name example.com;

        # Apply rate limit
        location / {
            limit_req zone=mylimit burst=20 nodelay;
            limit_conn addr 10;
            
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
        
        # Static content (bypass backend)
        location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
            root /var/www/static;
            expires 30d;
            access_log off;
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
        }
    }
}
```

---

## Apache as Reverse Proxy

### Installation and Modules
```bash
# RHEL/CentOS/Fedora
dnf install httpd

# Enable proxy modules
# Modules are typically enabled by default or in:
# /etc/httpd/conf.modules.d/00-proxy.conf

# Required modules:
# mod_proxy
# mod_proxy_http
# mod_proxy_balancer
# mod_lbmethod_byrequests

# Ubuntu/Debian
apt install apache2
a2enmod proxy
a2enmod proxy_http
a2enmod proxy_balancer
a2enmod lbmethod_byrequests

systemctl restart httpd  # or apache2
```

### Basic Reverse Proxy

```apache
# /etc/httpd/conf.d/proxy.conf

<VirtualHost *:80>
    ServerName example.com
    
    ProxyPreserveHost On
    ProxyPass / http://192.168.1.10/
    ProxyPassReverse / http://192.168.1.10/
    
    # Logging
    ErrorLog /var/log/httpd/proxy_error.log
    CustomLog /var/log/httpd/proxy_access.log combined
</VirtualHost>
```

### Load Balancer Configuration

```apache
<VirtualHost *:80>
    ServerName example.com
    
    # Load balancer
    <Proxy balancer://mycluster>
        BalancerMember http://192.168.1.10:80
        BalancerMember http://192.168.1.11:80
        BalancerMember http://192.168.1.12:80
        
        # Load balancing method
        ProxySet lbmethod=byrequests
        # Options: byrequests, bytraffic, bybusyness
    </Proxy>
    
    ProxyPreserveHost On
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```

### Advanced Apache Load Balancer

```apache
<VirtualHost *:80>
    ServerName example.com
    
    <Proxy balancer://mycluster>
        # Server definitions with parameters
        BalancerMember http://192.168.1.10:80 loadfactor=3 route=node1
        BalancerMember http://192.168.1.11:80 loadfactor=2 route=node2
        BalancerMember http://192.168.1.12:80 loadfactor=1 status=+H
        # status=+H means hot standby (backup)
        
        # Health check (requires mod_proxy_hcheck)
        # ProxySet lbmethod=bybusyness
        
        # Session stickiness
        ProxySet stickysession=ROUTEID
    </Proxy>
    
    # Headers
    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader set X-Forwarded-Port "80"
    
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
    
    # Manager interface
    <Location /balancer-manager>
        SetHandler balancer-manager
        Require ip 192.168.1.0/24
    </Location>
</VirtualHost>
```

### SSL Termination

```apache
<VirtualHost *:443>
    ServerName example.com
    
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/cert.pem
    SSLCertificateKeyFile /etc/pki/tls/private/key.pem
    
    <Proxy balancer://mycluster>
        BalancerMember http://192.168.1.10:80
        BalancerMember http://192.168.1.11:80
    </Proxy>
    
    RequestHeader set X-Forwarded-Proto "https"
    ProxyPreserveHost On
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>

# HTTP to HTTPS redirect
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

---

## Comparison: HAProxy vs Nginx vs Apache

| Feature | HAProxy | Nginx | Apache |
|---------|---------|-------|--------|
| **Primary Use** | Load balancing | Web server + proxy | Web server + proxy |
| **Performance** | Excellent | Excellent | Good |
| **Configuration** | Moderate complexity | Moderate | Complex |
| **Layer 4 LB** | Yes | Yes (stream) | Limited |
| **Layer 7 LB** | Yes | Yes | Yes |
| **Health Checks** | Advanced | Basic+ | Basic |
| **SSL Termination** | Yes | Yes | Yes |
| **Statistics** | Built-in | Third-party | mod_status |
| **Hot Reload** | Yes | Yes | Limited |
| **Ease of Use** | Moderate | Good | Moderate |

---

## Common Scenarios

### Scenario 1: Web Application Load Balancer
```nginx
# Nginx configuration
http {
    upstream webapp {
        least_conn;
        server 192.168.1.10:8080 weight=2;
        server 192.168.1.11:8080 weight=2;
        server 192.168.1.12:8080 weight=1;
        
        keepalive 32;
    }

    server {
        listen 80;
        server_name app.example.com;
        
        location / {
            proxy_pass http://webapp;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### Scenario 2: API Gateway with Rate Limiting
```nginx
http {
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/s;
    
    upstream api_backend {
        server 192.168.1.20:3000;
        server 192.168.1.21:3000;
    }

    server {
        listen 80;
        server_name api.example.com;
        
        location /api/ {
            limit_req zone=api_limit burst=50 nodelay;
            
            proxy_pass http://api_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

### Scenario 3: High Availability Setup with Health Checks
```bash
# HAProxy configuration
frontend http_front
    bind *:80
    default_backend web_servers

backend web_servers
    balance leastconn
    option httpchk GET /health HTTP/1.1\r\nHost:\ example.com
    http-check expect status 200
    
    server web1 192.168.1.10:80 check inter 5s fall 3 rise 2
    server web2 192.168.1.11:80 check inter 5s fall 3 rise 2
    server web3 192.168.1.12:80 check inter 5s fall 3 rise 2 backup
```

### Scenario 4: SSL Offloading
```nginx
server {
    listen 443 ssl http2;
    server_name secure.example.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## Monitoring and Logging

### HAProxy Stats
```bash
# Via stats page
frontend http_front
    stats enable
    stats uri /stats
    stats auth admin:password

# Via socket
echo "show stat" | socat stdio /run/haproxy/admin.sock

# Show info
echo "show info" | socat stdio /run/haproxy/admin.sock
```

### Nginx Status
```nginx
server {
    listen 80;
    
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

### Logs
```bash
# HAProxy logs
tail -f /var/log/haproxy.log

# Nginx logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Apache logs
tail -f /var/log/httpd/access_log
tail -f /var/log/httpd/error_log
```

---

## Troubleshooting

### Check Configuration
```bash
# HAProxy
haproxy -c -f /etc/haproxy/haproxy.cfg

# Nginx
nginx -t

# Apache
apachectl configtest
httpd -t
```

### Test Backend Connectivity
```bash
# From load balancer
curl -I http://192.168.1.10/
telnet 192.168.1.10 80
nc -vz 192.168.1.10 80
```

### Debug Issues
```bash
# Check if service is running
systemctl status haproxy
systemctl status nginx
systemctl status httpd

# Check listening ports
ss -tlnp | grep haproxy
ss -tlnp | grep nginx

# View connections
ss -tn | grep :80

# Check logs
journalctl -u haproxy -f
journalctl -u nginx -f
```

### Common Issues

**Backend servers not responding:**
```bash
# Check health
curl http://backend-ip/health

# Verify firewall
firewall-cmd --list-all

# Check backend logs
```

**SSL certificate errors:**
```bash
# Verify certificate
openssl x509 -in /path/to/cert.pem -text -noout

# Test SSL
openssl s_client -connect example.com:443
```

---

## Quick Reference

### HAProxy
```bash
haproxy -c -f /etc/haproxy/haproxy.cfg    # Test config
systemctl reload haproxy                   # Reload
echo "show stat" | socat stdio /run/haproxy/admin.sock  # Stats
```

### Nginx
```bash
nginx -t                                   # Test config
systemctl reload nginx                     # Reload
curl http://localhost/nginx_status         # Status
```

### Apache
```bash
apachectl configtest                       # Test config
systemctl reload httpd                     # Reload
curl http://localhost/server-status        # Status
```

---

## Exam Tips

- Know how to configure basic reverse proxy with Nginx or HAProxy
- Understand load balancing algorithms (round-robin, least-conn, etc.)
- Be familiar with health checks configuration
- Know how to implement SSL termination
- Understand session persistence mechanisms
- Practice testing configurations before applying
- Know how to view statistics and monitor
- Be comfortable with ACLs and routing rules
- Understand the difference between Layer 4 and Layer 7 load balancing
- Know how to troubleshoot backend connectivity issues
- Practice configuring multiple backends with different weights
- Understand when to use each solution (HAProxy vs Nginx vs Apache)
