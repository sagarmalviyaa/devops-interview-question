# Networking — Theory Questions

---

## Q1: What is the OSI model? Why should a DevOps engineer know it?

**Answer:**
The **OSI (Open Systems Interconnection) model** is a framework that describes how data travels from one computer to another over a network. It has **7 layers**, each handling a specific part of communication.

| Layer | Name | What It Does | Example |
|-------|------|-------------|---------|
| 7 | Application | What the user interacts with | HTTP, HTTPS, DNS, FTP, SSH |
| 6 | Presentation | Data formatting and encryption | SSL/TLS, JPEG, JSON |
| 5 | Session | Manages connections between apps | Login sessions, API sessions |
| 4 | Transport | Reliable data delivery | TCP (reliable), UDP (fast) |
| 3 | Network | Routing between networks | IP addresses, routers |
| 2 | Data Link | Communication within a local network | MAC addresses, switches |
| 1 | Physical | Actual cables and signals | Ethernet cables, Wi-Fi signals |

**Why DevOps engineers need this:**
- Troubleshooting: "Is the problem at the network level (Layer 3) or the application level (Layer 7)?"
- Load balancers operate at Layer 4 (TCP) or Layer 7 (HTTP) — knowing the difference matters.
- Firewalls, security groups, and network policies work at different layers.

---

## Q2: What is the difference between TCP and UDP?

**Answer:**

| Feature | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|---------|-----|-----|
| Connection | Connection-oriented (handshake first) | Connectionless (just send) |
| Reliability | Guaranteed delivery, in order | No guarantee, may arrive out of order |
| Speed | Slower (overhead for reliability) | Faster (no overhead) |
| Use cases | Web browsing, email, file transfer, SSH | Video streaming, DNS, gaming, VoIP |

**TCP three-way handshake:**
1. Client sends **SYN** (synchronize) — "I want to connect"
2. Server sends **SYN-ACK** — "OK, I'm ready"
3. Client sends **ACK** — "Great, let's talk"

**DevOps relevance:**
- Most services use TCP (HTTP, databases, SSH)
- DNS uses UDP for queries (fast) but TCP for zone transfers (reliable)
- Load balancers need to know which protocol to use
- Health checks often use TCP connection checks

---

## Q3: What is DNS and how does it work?

**Answer:**
**DNS (Domain Name System)** translates human-readable domain names (like `google.com`) into IP addresses (like `142.250.80.46`) that computers use to find each other.

**How a DNS lookup works (simplified):**
1. You type `www.example.com` in your browser
2. Browser checks its **local cache** — "Do I already know this?"
3. If not, asks the **operating system's DNS resolver** (also has a cache)
4. If not cached, asks the **recursive DNS server** (usually your ISP or 8.8.8.8)
5. Recursive server asks the **root DNS servers** → "Who handles `.com`?"
6. Root says → "Ask the `.com` TLD (Top-Level Domain) server"
7. TLD server says → "Ask `example.com`'s authoritative nameserver"
8. Authoritative nameserver responds with the IP address
9. The IP is cached at each level for future lookups

**Common DNS record types:**

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Maps domain to IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | Maps domain to IPv6 address | `example.com → 2606:2800:220:1:...` |
| **CNAME** | Alias for another domain | `www.example.com → example.com` |
| **MX** | Mail server for the domain | `example.com → mail.example.com` |
| **NS** | Nameserver for the domain | `example.com → ns1.example.com` |
| **TXT** | Text records (verification, SPF) | Used for email security, domain verification |
| **SRV** | Service location | Used by Kubernetes for service discovery |

**Useful commands:**
```bash
nslookup example.com          # Basic DNS lookup
dig example.com               # Detailed DNS lookup
dig +short example.com        # Just the IP
host example.com              # Simple lookup
```

---

## Q4: What is an IP address? What's the difference between IPv4 and IPv6?

**Answer:**
An **IP address** is a unique number assigned to every device on a network, like a postal address for computers.

**IPv4:**
- Format: `192.168.1.100` (four numbers, each 0–255)
- Total addresses: ~4.3 billion (running out!)
- Example: `10.0.0.1`, `172.16.0.1`

**IPv6:**
- Format: `2001:0db8:85a3:0000:0000:8a2e:0370:7334` (eight groups of hex numbers)
- Total addresses: 340 undecillion (essentially unlimited)
- Example: `::1` (localhost), `fe80::1`

**Private vs. Public IP addresses:**
- **Private** — Used within a local network, not routable on the internet
  - `10.0.0.0/8` — Large networks
  - `172.16.0.0/12` — Medium networks
  - `192.168.0.0/16` — Home/small networks
- **Public** — Unique on the internet, assigned by ISPs

---

## Q5: What is a subnet and what does CIDR notation mean?

**Answer:**
A **subnet (subnetwork)** divides a large network into smaller, manageable pieces. Think of it like dividing a building into floors.

**CIDR (Classless Inter-Domain Routing) notation:**
The `/number` after an IP address tells you how many bits are used for the network part.

```
192.168.1.0/24
├── 192.168.1  = Network part (first 24 bits)
└── .0         = Host part (last 8 bits = 256 addresses)
```

**Common CIDR blocks:**

| CIDR | Subnet Mask | Usable Hosts | Use Case |
|------|------------|-------------|----------|
| `/32` | 255.255.255.255 | 1 | Single host |
| `/24` | 255.255.255.0 | 254 | Small network |
| `/16` | 255.255.0.0 | 65,534 | Medium network |
| `/8` | 255.0.0.0 | 16 million+ | Large network |

**DevOps relevance:**
- AWS VPCs use CIDR blocks to define network ranges
- Security groups and firewall rules use CIDR for IP ranges
- Kubernetes pod networks use CIDR blocks
- `0.0.0.0/0` means "all IP addresses" (used in routing rules)

---

## Q6: What is a port? Name some common ports.

**Answer:**
A **port** is a number (0–65535) that identifies a specific service on a machine. If an IP address is like a building's street address, a port is like an apartment number.

**Common ports:**

| Port | Service | Protocol |
|------|---------|----------|
| 22 | SSH | TCP |
| 80 | HTTP | TCP |
| 443 | HTTPS | TCP |
| 53 | DNS | UDP/TCP |
| 25 | SMTP (email sending) | TCP |
| 3306 | MySQL | TCP |
| 5432 | PostgreSQL | TCP |
| 6379 | Redis | TCP |
| 27017 | MongoDB | TCP |
| 8080 | HTTP alternative (common for apps) | TCP |
| 9090 | Prometheus | TCP |
| 3000 | Grafana / Node.js apps | TCP |

**Port ranges:**
- **0–1023** — Well-known ports (require root to bind)
- **1024–49151** — Registered ports (for applications)
- **49152–65535** — Dynamic/ephemeral ports (temporary connections)

```bash
# Check what's listening on which port
ss -tulnp
netstat -tlnp

# Check if a specific port is open
nc -zv hostname 443
telnet hostname 443
```

---

## Q7: What is HTTP vs. HTTPS? How does TLS/SSL work?

**Answer:**
- **HTTP (HyperText Transfer Protocol)** — Sends data in plain text. Anyone intercepting the traffic can read it.
- **HTTPS (HTTP Secure)** — HTTP encrypted with TLS (Transport Layer Security). Data is encrypted so interceptors can't read it.

**How TLS works (simplified):**
1. **Client Hello** — Browser says "I want a secure connection" and lists supported encryption methods
2. **Server Hello** — Server picks an encryption method and sends its **SSL certificate** (which contains its public key)
3. **Certificate Verification** — Browser checks if the certificate is valid and trusted
4. **Key Exchange** — Browser and server agree on a shared secret key using the certificate's public key
5. **Encrypted Communication** — All data is now encrypted with the shared key

**SSL Certificates:**
- Issued by **Certificate Authorities (CAs)** like Let's Encrypt, DigiCert
- **Let's Encrypt** provides free certificates (auto-renewed every 90 days)
- Certificates contain: domain name, public key, issuer, expiration date

```bash
# Check a website's certificate
openssl s_client -connect example.com:443

# Check certificate expiration
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Generate a self-signed certificate (for testing)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

---

## Q8: What is a load balancer? What are the different types?

**Answer:**
A **load balancer** distributes incoming traffic across multiple servers so no single server gets overwhelmed. It improves availability, reliability, and performance.

**Types by OSI layer:**
- **Layer 4 (Transport)** — Routes based on IP address and port. Faster but less flexible. Can't inspect HTTP content.
- **Layer 7 (Application)** — Routes based on HTTP content (URL path, headers, cookies). Slower but smarter.

**Load balancing algorithms:**

| Algorithm | How It Works | Best For |
|-----------|-------------|----------|
| **Round Robin** | Each server gets a turn in order | Equal servers |
| **Weighted Round Robin** | Stronger servers get more turns | Mixed server sizes |
| **Least Connections** | Send to server with fewest active connections | Varying request durations |
| **IP Hash** | Same client IP always goes to same server | Session persistence |
| **Random** | Randomly pick a server | Simple setups |

**Common load balancers:**
- **Nginx** — Popular, can do both L4 and L7
- **HAProxy** — High-performance, widely used
- **AWS ALB** — Application Load Balancer (Layer 7)
- **AWS NLB** — Network Load Balancer (Layer 4)
- **Traefik** — Cloud-native, great with containers

---

## Q9: What is a firewall? What are security groups?

**Answer:**
A **firewall** controls what network traffic is allowed in and out of a system. It's like a security guard checking IDs at a door.

**Types:**
- **Host-based firewall** — Runs on a single machine (e.g., `iptables`, `ufw`, `firewalld`)
- **Network firewall** — Protects an entire network (hardware appliance or cloud service)
- **Security groups** — Cloud-based firewalls (AWS, Azure, GCP) that control traffic to/from cloud resources

**Linux firewall examples:**
```bash
# UFW (Uncomplicated Firewall) — Ubuntu
ufw allow 22/tcp          # Allow SSH
ufw allow 80/tcp           # Allow HTTP
ufw allow 443/tcp          # Allow HTTPS
ufw deny 3306/tcp          # Block MySQL from outside
ufw enable                 # Activate the firewall
ufw status                 # Check rules

# iptables (lower level)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -j DROP  # Drop everything else
```

**Security group rules (AWS example):**
- **Inbound:** Allow TCP port 443 from `0.0.0.0/0` (HTTPS from anywhere)
- **Inbound:** Allow TCP port 22 from `10.0.0.0/8` (SSH from internal network only)
- **Outbound:** Allow all traffic (default)

**Best practice:** Follow the **principle of least privilege** — only allow the traffic that's absolutely necessary.

---

## Q10: What is NAT (Network Address Translation)?

**Answer:**
**NAT** translates private IP addresses to a public IP address (and vice versa). It lets multiple devices on a private network share a single public IP address.

**Types:**
- **SNAT (Source NAT)** — Changes the source IP of outgoing traffic. Used when private servers need internet access.
- **DNAT (Destination NAT)** — Changes the destination IP of incoming traffic. Used for port forwarding.
- **PAT (Port Address Translation)** — A form of NAT that uses different ports to track connections. This is what your home router does.

**DevOps relevance:**
- **NAT Gateway (AWS)** — Lets private subnet instances access the internet without being directly reachable
- **Kubernetes** — Uses NAT for pod-to-external communication
- **Docker** — Uses NAT to map container ports to host ports

---

## Q11: What is a VPN? When would you use one in DevOps?

**Answer:**
A **VPN (Virtual Private Network)** creates an encrypted tunnel between two networks or a device and a network. It makes remote connections secure, as if you were on the same local network.

**Use cases in DevOps:**
- **Accessing private cloud resources** — Connect to AWS VPC private subnets from your laptop
- **Site-to-site VPN** — Connect your office network to your cloud network
- **Secure remote access** — Engineers accessing production systems from home
- **Cross-region connectivity** — Connect cloud resources in different regions

**Common VPN solutions:**
- **AWS VPN** — Managed VPN for AWS VPC
- **WireGuard** — Modern, fast, simple VPN protocol
- **OpenVPN** — Widely used, open-source
- **Tailscale** — Zero-config VPN built on WireGuard

---

## Q12: What is a proxy vs. a reverse proxy?

**Answer:**

**Forward Proxy (Proxy):**
- Sits between **clients** and the internet
- Clients send requests to the proxy, which forwards them to the destination
- Use cases: Content filtering, caching, anonymity
- Example: Corporate proxy that blocks certain websites

**Reverse Proxy:**
- Sits between the **internet** and your servers
- Clients think they're talking to the proxy, but it forwards requests to backend servers
- Use cases: Load balancing, SSL termination, caching, security
- Example: Nginx in front of your application servers

```
Forward Proxy:
Client → [Proxy] → Internet → Server

Reverse Proxy:
Client → Internet → [Reverse Proxy] → Server1
                                     → Server2
                                     → Server3
```

**Common reverse proxies:** Nginx, HAProxy, Traefik, Envoy, Caddy

---

## Q13: What is the difference between a router and a switch?

**Answer:**

| Feature | Switch | Router |
|---------|--------|--------|
| OSI Layer | Layer 2 (Data Link) | Layer 3 (Network) |
| Uses | MAC addresses | IP addresses |
| Connects | Devices within the same network | Different networks together |
| Analogy | Mail room in a building | Post office between cities |

- **Switch** — Connects devices in a local network (LAN). It learns which device is on which port using MAC addresses.
- **Router** — Connects different networks. It decides the best path for data to travel between networks using IP addresses.

**In cloud (AWS):**
- **VPC** = Your private network
- **Subnets** = Segments within the VPC
- **Route tables** = Rules for how traffic flows between subnets and the internet
- **Internet Gateway** = Router connecting your VPC to the internet

---

## Q14: What is DHCP?

**Answer:**
**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses to devices when they join a network. Without DHCP, you'd have to manually configure every device's IP address.

**How it works (DORA process):**
1. **Discover** — Device broadcasts "I need an IP address!"
2. **Offer** — DHCP server responds with an available IP
3. **Request** — Device says "I'll take that one"
4. **Acknowledge** — Server confirms the assignment

**What DHCP provides:**
- IP address
- Subnet mask
- Default gateway (router)
- DNS server addresses
- Lease time (how long the IP is valid)

**DevOps relevance:**
- Cloud instances get IPs via DHCP-like mechanisms
- Kubernetes pods get IPs from the cluster's IP pool
- Understanding DHCP helps troubleshoot "no network" issues

---

## Q15: What are the common HTTP status codes?

**Answer:**

| Code Range | Category | Meaning |
|-----------|----------|---------|
| **1xx** | Informational | Request received, continuing |
| **2xx** | Success | Request succeeded |
| **3xx** | Redirection | Further action needed |
| **4xx** | Client Error | Problem with the request |
| **5xx** | Server Error | Server failed to fulfill request |

**Most important codes:**

| Code | Meaning | Common Cause |
|------|---------|-------------|
| **200** | OK | Everything worked |
| **201** | Created | Resource successfully created (POST) |
| **301** | Moved Permanently | URL has changed permanently |
| **302** | Found (Temporary Redirect) | URL temporarily redirected |
| **400** | Bad Request | Malformed request |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | Authenticated but not allowed |
| **404** | Not Found | Resource doesn't exist |
| **429** | Too Many Requests | Rate limit exceeded |
| **500** | Internal Server Error | Server crashed or bug |
| **502** | Bad Gateway | Proxy/LB can't reach backend |
| **503** | Service Unavailable | Server overloaded or in maintenance |
| **504** | Gateway Timeout | Backend took too long to respond |

---

## Q16: What is a CDN (Content Delivery Network)?

**Answer:**
A **CDN** is a network of servers distributed around the world that cache and deliver content from the location closest to the user. This makes websites faster.

**How it works:**
1. Your website's static files (images, CSS, JS) are copied to CDN servers worldwide
2. When a user in Tokyo visits your site, they get files from a nearby CDN server instead of your origin server in New York
3. This reduces latency (delay) and load on your origin server

**Popular CDNs:**
- **CloudFlare** — Also provides DDoS protection and DNS
- **AWS CloudFront** — Integrates with S3 and other AWS services
- **Akamai** — Enterprise-grade
- **Fastly** — Developer-friendly, edge computing

**DevOps relevance:**
- Configure CDN for static assets to reduce server load
- Set proper cache headers (`Cache-Control`, `ETag`)
- Invalidate CDN cache during deployments
- Use CDN for SSL termination