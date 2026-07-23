# Month 1 — PART 2: NETWORKING FUNDAMENTALS

---

## 2.1 What is Networking?

Networking is how computers communicate with each other. Every application you deploy, every API call, every database query — all happen over a network. As a DevOps engineer, you troubleshoot network issues daily.

**Why DevOps Engineers need networking:**
- Troubleshoot "connection refused" / "timeout" errors
- Design cloud VPCs (Virtual Private Clouds)
- Configure load balancers and reverse proxies
- Set up Kubernetes networking (services, ingress)
- Implement network security (firewalls, ACLs)
- Debug microservice communication issues

---

## 2.2 The OSI Model

The OSI model describes how data travels from one application to another. Think of it as layers of an onion — each layer adds/removes headers.

```
Layer 7: Application   — What the user/app interacts with
                         Protocols: HTTP, HTTPS, DNS, FTP, SSH, SMTP, SNMP
                         DevOps: API calls, web servers, DNS resolution

Layer 6: Presentation  — Data format, encryption
                         Protocols: SSL/TLS, JPEG, ASCII
                         DevOps: TLS certificates, data encoding

Layer 5: Session       — Manage connections
                         Protocols: NetBIOS, RPC
                         DevOps: Session management, connection pooling

Layer 4: Transport     — How data is delivered (reliable vs fast)
                         Protocols: TCP (reliable), UDP (fast)
                         DevOps: Port numbers, load balancing (L4)

Layer 3: Network       — Routing between networks
                         Protocols: IP, ICMP, BGP, OSPF
                         DevOps: IP addressing, subnets, routing, VPCs

Layer 2: Data Link     — Communication within a network
                         Protocols: Ethernet, ARP, MAC addresses
                         DevOps: VLANs, switches, ARP issues

Layer 1: Physical      — Physical cables and signals
                         Hardware: Cables, NICs, hubs
                         DevOps: Rarely deal with this (cloud abstracts it)
```

**Memory Aid:** "Please Do Not Throw Sausage Pizza Away" (Physical → Application)

**Practical Rule:** When troubleshooting, start from the bottom:
1. Can you ping it? (Layer 3 — Network)
2. Can you reach the port? (Layer 4 — Transport)
3. Does the application respond? (Layer 7 — Application)

---

## 2.3 TCP vs UDP

```
TCP (Transmission Control Protocol):
┌──────────────────────────────────────────────┐
│ Connection-oriented (must establish first)     │
│ Reliable delivery (guarantees all data arrives)│
│ Ordered (packets arrive in sequence)           │
│ Error checking + retransmission               │
│ Slower (overhead from reliability)             │
│                                                │
│ 3-Way Handshake:                              │
│   Client → SYN → Server                      │
│   Client ← SYN-ACK ← Server                  │
│   Client → ACK → Server                      │
│   (Connection established)                     │
│                                                │
│ Used for: HTTP/S, SSH, FTP, SMTP, databases   │
│ "I need every packet to arrive correctly"      │
└──────────────────────────────────────────────┘

UDP (User Datagram Protocol):
┌──────────────────────────────────────────────┐
│ Connectionless (just send it)                  │
│ Unreliable (no guarantee of delivery)          │
│ Unordered (packets may arrive out of sequence) │
│ No retransmission                              │
│ Fast (minimal overhead)                        │
│                                                │
│ Used for: DNS, video streaming, gaming, VoIP   │
│ "Speed matters more than perfection"           │
└──────────────────────────────────────────────┘
```

---

## 2.4 IP Addressing & Subnetting

### IPv4 Addresses
```
Format: 4 octets, each 0-255
Example: 192.168.1.100

Private IP Ranges (not routable on internet):
  10.0.0.0/8       → 10.0.0.0 - 10.255.255.255      (16 million IPs)
  172.16.0.0/12    → 172.16.0.0 - 172.31.255.255     (1 million IPs)
  192.168.0.0/16   → 192.168.0.0 - 192.168.255.255   (65,000 IPs)

Special Addresses:
  127.0.0.1        → Localhost (loopback — yourself)
  0.0.0.0          → All interfaces / default route
  255.255.255.255  → Broadcast (everyone on network)
  169.254.x.x      → Link-local (DHCP failed)
```

### CIDR Notation (Classless Inter-Domain Routing)
```
CIDR tells you how many IPs are in a network.
Format: IP/prefix_length

/32 = 1 IP          (single host)    — 2^0 = 1
/31 = 2 IPs         (point-to-point)
/30 = 4 IPs         (2 usable)
/28 = 16 IPs        (14 usable)
/24 = 256 IPs       (254 usable)     — Most common subnet
/20 = 4,096 IPs
/16 = 65,536 IPs    (large network)
/8  = 16,777,216 IPs (massive)

Formula: 2^(32-prefix) = total IPs
         Usable = total - 2 (network address + broadcast)

Example: 10.0.1.0/24
  Network:   10.0.1.0
  First host: 10.0.1.1
  Last host:  10.0.1.254
  Broadcast:  10.0.1.255
  Subnet mask: 255.255.255.0
```

### Subnetting in Practice (Cloud VPC Design)
```
VPC CIDR: 10.0.0.0/16 (65,536 IPs total)

Subnet division:
  Public subnets (for load balancers, bastion):
    10.0.1.0/24    (AZ-a)  — 256 IPs
    10.0.2.0/24    (AZ-b)  — 256 IPs

  Private subnets (for applications):
    10.0.10.0/24   (AZ-a)  — 256 IPs
    10.0.20.0/24   (AZ-b)  — 256 IPs

  Database subnets (most restricted):
    10.0.100.0/24  (AZ-a)  — 256 IPs
    10.0.200.0/24  (AZ-b)  — 256 IPs
```

**TIP:** Use https://www.ipaddressguide.com/cidr for CIDR calculations!

---

## 2.5 DNS (Domain Name System)

DNS translates human-readable names (google.com) to IP addresses (142.250.185.46).

### How DNS Resolution Works
```
1. You type "www.example.com" in browser
2. Browser checks its cache → Not found
3. OS checks /etc/hosts → Not found
4. OS asks configured DNS resolver (from /etc/resolv.conf)
5. Resolver checks its cache → Not found
6. Resolver asks Root nameserver → "Ask .com TLD server"
7. Resolver asks .com TLD server → "Ask ns1.example.com"
8. Resolver asks ns1.example.com → "IP is 93.184.216.34"
9. Resolver caches result and returns to OS
10. Browser connects to 93.184.216.34
```

### DNS Record Types
```
A       → Maps domain to IPv4 address
           example.com → 93.184.216.34

AAAA    → Maps domain to IPv6 address
           example.com → 2606:2800:220:1:248:1893:25c8:1946

CNAME   → Alias (points to another domain name)
           www.example.com → example.com
           ⚠️ Cannot be used at zone apex (root domain)

MX      → Mail server (with priority)
           example.com MX 10 mail.example.com

NS      → Name servers for the domain
           example.com NS ns1.example.com

TXT     → Text records (SPF, DKIM, domain verification)
           example.com TXT "v=spf1 include:_spf.google.com ~all"

SRV     → Service locator (port + host)
           _sip._tcp.example.com SRV 10 60 5060 sip.example.com

PTR     → Reverse DNS (IP → domain name)
           34.216.184.93.in-addr.arpa PTR example.com

SOA     → Start of Authority (zone configuration)

TTL     → Time To Live (how long to cache)
           Low (60s): Quick propagation after changes
           High (86400s): Less DNS traffic, slower changes
```

### DNS Troubleshooting Commands
```bash
# Basic lookup
nslookup example.com                   # Simple DNS query
dig example.com                        # Detailed DNS query
dig +short example.com A               # Just the IP
dig example.com MX                     # Mail records
dig @8.8.8.8 example.com              # Query specific DNS server
host example.com                       # Simple lookup

# Trace full resolution path
dig +trace example.com                 # Shows every step of resolution

# Reverse DNS
dig -x 8.8.8.8                        # IP → domain name

# Check DNS configuration
cat /etc/resolv.conf                   # Your DNS servers
systemd-resolve --status               # DNS status on systemd systems

# Common DNS issues:
# "Could not resolve host" → DNS server unreachable or domain doesn't exist
# Slow resolution → DNS server slow, try 8.8.8.8 or 1.1.1.1
# Wrong IP returned → Stale cache (wait for TTL) or wrong DNS server
```

---

## 2.6 HTTP/HTTPS

```
HTTP Request/Response Cycle:
  Client → Request (method + URL + headers + body) → Server
  Client ← Response (status code + headers + body) ← Server

HTTP Methods:
  GET     → Retrieve data (no body)
  POST    → Submit/create data
  PUT     → Replace entire resource
  PATCH   → Partial update
  DELETE  → Remove resource
  HEAD    → Like GET but no body (check if exists)
  OPTIONS → What methods are supported (CORS preflight)

Status Codes (MEMORIZE THESE):
  2xx — Success:
    200 OK                  — Request succeeded
    201 Created             — Resource created (POST)
    204 No Content          — Success, no body to return

  3xx — Redirect:
    301 Moved Permanently   — URL changed forever
    302 Found               — Temporary redirect
    304 Not Modified        — Use cached version

  4xx — Client Error:
    400 Bad Request         — Malformed request
    401 Unauthorized        — Not authenticated (need to login)
    403 Forbidden           — Authenticated but not allowed
    404 Not Found           — Resource doesn't exist
    405 Method Not Allowed  — Wrong HTTP method
    408 Request Timeout     — Client too slow
    429 Too Many Requests   — Rate limited

  5xx — Server Error:
    500 Internal Server Error — Something broke on server
    502 Bad Gateway          — Upstream server returned invalid response
    503 Service Unavailable  — Server overloaded or in maintenance
    504 Gateway Timeout      — Upstream server didn't respond in time

HTTPS = HTTP + TLS (encrypted)
  TLS Handshake:
    Client → ClientHello (supported ciphers)
    Server ← ServerHello (chosen cipher + certificate)
    Client → Verify certificate, key exchange
    Both   → Derive session keys
    Both   → Encrypted communication begins
```

### HTTP Headers You Should Know
```
Request Headers:
  Host: example.com              — Which server (virtual hosting)
  Authorization: Bearer token    — Authentication
  Content-Type: application/json — Body format
  Accept: application/json       — Desired response format
  User-Agent: curl/7.64          — Client identification
  X-Forwarded-For: client-ip     — Original client IP (behind proxy)

Response Headers:
  Content-Type: application/json — Response format
  Cache-Control: max-age=3600    — Caching instructions
  Set-Cookie: session=abc123     — Set browser cookie
  X-Request-ID: uuid             — Request tracking
  Strict-Transport-Security      — Force HTTPS (HSTS)
```

---

## 2.7 Load Balancing

```
What is a Load Balancer?
  Distributes incoming traffic across multiple backend servers.
  Provides: High availability, scalability, health checking.

Types:
  Layer 4 (Transport) — TCP/UDP level
    - Routes based on IP + port
    - Cannot inspect HTTP content
    - Faster (less processing)
    - Example: AWS NLB, Azure LB

  Layer 7 (Application) — HTTP level
    - Routes based on URL path, headers, cookies
    - Can inspect and modify requests
    - More features (SSL termination, URL rewriting)
    - Example: AWS ALB, Nginx, HAProxy, Azure App Gateway

Algorithms:
  Round Robin         — Equal distribution, one by one
  Weighted Round Robin — More traffic to stronger servers
  Least Connections   — Send to server with fewest active connections
  IP Hash             — Same client always goes to same server (sticky)
  Random              — Random selection

Health Checks:
  TCP check    → Can we connect to the port?
  HTTP check   → Does GET /health return 200?
  Custom       → Run a script/command

Session Persistence (Sticky Sessions):
  Problem: User's session data is on Server A, next request goes to Server B
  Solutions:
    - Cookie-based routing (LB sets cookie with server ID)
    - IP affinity (same IP → same server)
    - External session store (Redis) — BEST APPROACH
```

---

## 2.8 Network Troubleshooting Toolkit

```bash
# === Layer 3: Can I reach the host? ===
ping google.com                        # Basic connectivity (ICMP)
ping -c 4 10.0.1.50                    # Send 4 pings only
traceroute google.com                  # Show network path (hops)
mtr google.com                         # Better traceroute (live, combines ping+trace)

# === Layer 4: Can I reach the port? ===
telnet host 80                         # Test TCP port connectivity
nc -zv host 443                        # Netcat port check (quick)
nc -zv host 1-1000                     # Scan port range
nmap -p 80,443 host                    # Port scanner

# === Layer 7: Does the application respond? ===
curl http://example.com                # Basic HTTP request
curl -v https://example.com            # Verbose (see TLS + headers)
curl -I https://example.com            # HEAD request (headers only)
curl -o /dev/null -s -w "%{http_code}\n" URL  # Just status code
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' URL
wget --spider URL                      # Check if URL is reachable

# === Local network info ===
ip addr show                           # All interfaces and IPs
ip route show                          # Routing table
ip neigh                               # ARP table (IP → MAC)
ss -tlnp                               # Listening TCP ports with process
ss -ulnp                               # Listening UDP ports
ss -s                                  # Socket statistics summary

# === Packet capture (advanced) ===
sudo tcpdump -i eth0 port 80           # Capture HTTP traffic
sudo tcpdump -i any host 10.0.1.50    # Traffic to/from specific host
sudo tcpdump -i eth0 -w capture.pcap  # Save to file (open in Wireshark)

# === DNS ===
dig +short example.com                 # Quick DNS lookup
dig @8.8.8.8 example.com              # Use specific DNS server
cat /etc/resolv.conf                   # Check DNS config
```

### Common Network Problems & Solutions
```
Problem: "Connection refused"
  Meaning: Host is reachable, but port is closed / service not running
  Fix: Check if service is running: systemctl status service
       Check port: ss -tlnp | grep PORT

Problem: "Connection timed out"
  Meaning: Can't reach the host at all
  Fix: Check firewall rules (security groups, iptables)
       Check routing (ip route, route tables)
       Check if host is up (ping)

Problem: "Name or service not known"
  Meaning: DNS can't resolve the hostname
  Fix: Check /etc/resolv.conf
       Try: dig hostname, nslookup hostname
       Try direct IP to bypass DNS

Problem: "502 Bad Gateway"
  Meaning: Load balancer/proxy can't reach backend
  Fix: Check backend service health
       Check backend is listening on expected port
       Check network between LB and backend

Problem: "Connection reset by peer"
  Meaning: Remote side forcibly closed the connection
  Fix: Check server logs for errors
       Check max connections/resource limits
       Check timeout settings
```

---

## 2.9 Networking Free Resources

| Resource | Link | Notes |
|----------|------|-------|
| Subnet Calculator | https://www.ipaddressguide.com/cidr | CIDR calculation |
| Practical Networking (YouTube) | https://www.youtube.com/c/PracticalNetworking | Best networking concepts |
| NetworkChuck (YouTube) | https://www.youtube.com/c/NetworkChuck | Entertaining network tutorials |
| Computer Networking (book) | "Computer Networking: A Top-Down Approach" | Gold standard textbook |
| Cloudflare Learning Center | https://www.cloudflare.com/learning/ | Excellent DNS/HTTP/TLS guides |
| Julia Evans Zines | https://wizardzines.com | Visual networking/Linux guides |
| DNS Toys | https://www.nslookup.io | Interactive DNS tool |
| HTTP Status Codes | https://httpstatuses.com | Quick reference |

---
