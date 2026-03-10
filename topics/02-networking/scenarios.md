# Networking — Scenario-Based Questions

---

## S1: Users report that your website is very slow. How do you determine if it's a network issue?

**Answer:**

**Step-by-step diagnosis:**

1. **Check from the server side first:**
   ```bash
   # Is the server itself slow?
   top                    # Check CPU/memory
   curl -o /dev/null -s -w "Time: %{time_total}s\n" http://localhost
   # If localhost is fast, the problem is between the user and the server
   ```

2. **Check DNS resolution:**
   ```bash
   dig example.com
   # If DNS takes seconds, that's a problem
   # Normal DNS resolution: < 50ms
   ```

3. **Check network latency:**
   ```bash
   ping example.com
   # Normal: < 100ms for same region
   # High latency suggests network issues
   
   traceroute example.com
   # Shows each network hop — find where the delay is
   ```

4. **Check from the user's perspective:**
   ```bash
   # Use curl to measure each phase
   curl -w "DNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTLS: %{time_appconnect}s\nFirst byte: %{time_starttransfer}s\nTotal: %{time_total}s\n" -o /dev/null -s https://example.com
   ```

5. **Check for packet loss:**
   ```bash
   mtr example.com
   # Shows latency AND packet loss at each hop
   ```

6. **Check bandwidth:**
   ```bash
   iperf3 -c server-ip    # Test network throughput between two points
   ```

**Common network causes of slowness:**
- High latency (users far from server → use a CDN)
- Packet loss (bad network path → contact ISP/cloud provider)
- DNS issues (slow resolver → switch to 8.8.8.8 or 1.1.1.1)
- SSL/TLS handshake overhead (→ enable TLS session resumption)
- Bandwidth saturation (→ scale up or use load balancing)

---

## S2: A service on port 8080 is running but users can't access it from outside. What do you check?

**Answer:**

1. **Verify the service is running and listening:**
   ```bash
   ss -tlnp | grep 8080
   # Check if it's listening on 0.0.0.0:8080 (all interfaces)
   # If it's on 127.0.0.1:8080, it only accepts local connections!
   ```

2. **Check the local firewall:**
   ```bash
   # UFW
   ufw status
   ufw allow 8080/tcp
   
   # iptables
   iptables -L -n | grep 8080
   
   # firewalld
   firewall-cmd --list-all
   ```

3. **Check cloud security groups (if on cloud):**
   - AWS: Check the instance's security group inbound rules
   - Ensure port 8080 is allowed from the user's IP range

4. **Check Network ACLs (if on AWS):**
   - NACLs are stateless — check both inbound AND outbound rules

5. **Check if there's a load balancer or proxy in front:**
   - The LB might not have port 8080 configured
   - Check LB health checks — if they fail, traffic won't be routed

6. **Test connectivity step by step:**
   ```bash
   # From the server itself
   curl http://localhost:8080    # Should work
   
   # From another machine in the same network
   curl http://server-private-ip:8080
   
   # From outside
   curl http://server-public-ip:8080
   
   # Use nc to test raw connectivity
   nc -zv server-public-ip 8080
   ```

7. **Check if the application binds to the right interface:**
   - Some apps default to `127.0.0.1` (localhost only)
   - Change to `0.0.0.0` to accept connections from any interface

---

## S3: Your application is getting a "502 Bad Gateway" error. What's happening and how do you fix it?

**Answer:**

A **502 Bad Gateway** means the reverse proxy or load balancer received an invalid response (or no response) from the backend server.

**Common causes and fixes:**

1. **Backend server is down:**
   ```bash
   systemctl status my-app
   # If stopped, restart it
   systemctl restart my-app
   ```

2. **Backend server is overloaded:**
   ```bash
   top    # Check CPU/memory on the backend
   # Solution: Scale up (bigger server) or scale out (more servers)
   ```

3. **Backend is too slow (timeout):**
   ```nginx
   # In Nginx config, increase timeout
   proxy_read_timeout 300s;
   proxy_connect_timeout 75s;
   ```

4. **Wrong backend address in proxy config:**
   ```bash
   # Check Nginx config
   cat /etc/nginx/sites-enabled/myapp
   # Verify the proxy_pass URL is correct
   # proxy_pass http://localhost:3000;
   ```

5. **Backend crashed but service is still "running":**
   ```bash
   # Check application logs
   journalctl -u my-app --since "10 minutes ago"
   # Look for out-of-memory errors, unhandled exceptions
   ```

6. **Port mismatch:**
   - Proxy expects backend on port 3000, but app is running on 8080

**Debugging flow:**
```
User → [502] → Nginx → [?] → Backend
                  ↓
         Check Nginx error log:
         /var/log/nginx/error.log
         
         Common messages:
         "connect() failed" → Backend is down
         "upstream timed out" → Backend is too slow
         "no live upstreams" → All backends are unhealthy
```

---

## S4: You need to troubleshoot DNS resolution failures. Walk through your process.

**Answer:**

1. **Verify the problem:**
   ```bash
   nslookup example.com
   dig example.com
   # If these fail, it's a DNS issue
   
   # Try with a known-good DNS server
   dig @8.8.8.8 example.com
   # If this works, your configured DNS server is the problem
   ```

2. **Check your DNS configuration:**
   ```bash
   cat /etc/resolv.conf
   # Shows which DNS servers your system uses
   # nameserver 10.0.0.2
   
   # Check if the DNS server is reachable
   ping 10.0.0.2
   nc -zvu 10.0.0.2 53    # DNS uses port 53
   ```

3. **Check if it's a specific domain or all domains:**
   ```bash
   dig google.com         # Try a well-known domain
   dig internal.company   # Try an internal domain
   ```

4. **Clear DNS cache:**
   ```bash
   # systemd-resolved
   systemd-resolve --flush-caches
   
   # Check if caching is the issue
   dig +nocache example.com
   ```

5. **Check for DNS propagation issues:**
   - If you recently changed DNS records, they might not have propagated yet
   - Use `dig +trace example.com` to see the full resolution path
   - Check TTL (Time To Live) values

6. **Common fixes:**
   - Update `/etc/resolv.conf` with working DNS servers
   - Restart the DNS resolver: `systemctl restart systemd-resolved`
   - If using a custom DNS server, check its logs
   - In Kubernetes: check CoreDNS pods and configmap

---

## S5: Two microservices in the same network can't communicate. How do you debug this?

**Answer:**

1. **Verify both services are running:**
   ```bash
   # Check service A
   curl http://service-a:8080/health
   
   # Check service B
   curl http://service-b:8080/health
   ```

2. **Check DNS resolution between services:**
   ```bash
   # From service A's container/pod
   nslookup service-b
   dig service-b
   ```

3. **Check network connectivity:**
   ```bash
   # From service A, try to reach service B
   ping service-b
   nc -zv service-b 8080
   curl -v http://service-b:8080/
   ```

4. **Check firewall/security rules:**
   - Network policies (Kubernetes)
   - Security groups (cloud)
   - iptables rules on the hosts

5. **Check if they're on the same network:**
   ```bash
   # Docker
   docker network ls
   docker inspect container-a | grep NetworkMode
   docker inspect container-b | grep NetworkMode
   # Both should be on the same network
   
   # Kubernetes
   kubectl get pods -o wide    # Check pod IPs and nodes
   kubectl get networkpolicies # Check if any policies block traffic
   ```

6. **Check service configuration:**
   - Is service B listening on the right port and interface?
   - Is service A using the correct hostname/port for service B?
   - Are environment variables for service URLs set correctly?

7. **Check logs on both sides:**
   ```bash
   # Service A logs — look for connection errors
   docker logs service-a
   
   # Service B logs — look for incoming request issues
   docker logs service-b
   ```

---

## S6: You need to set up a network for a new application in AWS. It needs public-facing web servers and private database servers. Design the network.

**Answer:**

**Architecture:**

```
Internet
    │
    ├── Internet Gateway
    │
    ├── Public Subnet (10.0.1.0/24)
    │   ├── Web Server 1 (public IP)
    │   ├── Web Server 2 (public IP)
    │   └── NAT Gateway (for private subnet internet access)
    │
    └── Private Subnet (10.0.2.0/24)
        ├── Database Server (no public IP)
        └── Cache Server (no public IP)
```

**Step-by-step setup:**

1. **Create a VPC** with CIDR `10.0.0.0/16`

2. **Create subnets:**
   - Public subnet: `10.0.1.0/24` (for web servers)
   - Private subnet: `10.0.2.0/24` (for databases)
   - Add subnets in multiple Availability Zones for high availability

3. **Create an Internet Gateway** and attach it to the VPC

4. **Create route tables:**
   - Public route table: `0.0.0.0/0 → Internet Gateway`
   - Private route table: `0.0.0.0/0 → NAT Gateway`

5. **Create a NAT Gateway** in the public subnet (so private instances can download updates)

6. **Create security groups:**
   - Web servers: Allow 80/443 from `0.0.0.0/0`, allow 22 from your office IP
   - Database: Allow 5432 (PostgreSQL) from web server security group ONLY
   - No direct internet access to databases

7. **Best practices:**
   - Use multiple Availability Zones
   - Enable VPC Flow Logs for monitoring
   - Use Network ACLs as an additional layer of security
   - Keep security groups as restrictive as possible