# DNS (Domain Name System)

Brief: DNS is the Internet's distributed directory service that translates human-readable domain names into IP addresses. It operates as a hierarchical, distributed database that enables scalable name resolution for billions of domains worldwide.

## Detailed Theory

### DNS Overview
- **Purpose**: Translate domain names to IP addresses (and vice versa)
- **Distributed System**: No single point of failure
- **Hierarchical Structure**: Root → TLD → Authoritative
- **Caching**: Improves performance and reduces load
- **Protocol**: Primarily UDP port 53, TCP for large responses
- **Global Infrastructure**: 13 root server clusters worldwide

### DNS Hierarchy

#### Root Level (.)
- **Root Servers**: 13 logical servers (A-M) with multiple physical instances
- **Anycast**: Same IP addresses advertised from multiple locations
- **Function**: Direct queries to appropriate TLD servers
- **Operators**: VeriSign, USC-ISI, Cogent, University of Maryland, NASA, etc.

#### Top-Level Domains (TLD)
- **Generic TLDs (gTLD)**: .com, .org, .net, .edu, .gov, .mil
- **Country Code TLDs (ccTLD)**: .us, .uk, .de, .jp, .cn
- **New gTLDs**: .app, .dev, .cloud, .tech (introduced 2013+)
- **Function**: Manage second-level domain registrations

#### Second-Level Domains
- **Examples**: google.com, github.io, amazon.co.uk
- **Authoritative Servers**: Contain actual DNS records
- **Nameservers**: ns1.google.com, ns2.google.com

#### Subdomains
- **Examples**: mail.google.com, www.github.io
- **Delegation**: Can be managed by same or different nameservers

### DNS Record Types

#### Address Records
**A Record** (IPv4 Address)
```
example.com.    3600    IN    A    192.168.1.100
```

**AAAA Record** (IPv6 Address)
```
example.com.    3600    IN    AAAA    2001:db8::1
```

#### Canonical Name
**CNAME Record** (Alias)
```
www.example.com.    3600    IN    CNAME    example.com.
blog.example.com.   3600    IN    CNAME    hosting.provider.com.
```

#### Mail Exchange
**MX Record** (Mail Server)
```
example.com.    3600    IN    MX    10 mail1.example.com.
example.com.    3600    IN    MX    20 mail2.example.com.
```
- Lower number = higher priority

#### Nameserver
**NS Record** (Nameserver)
```
example.com.    3600    IN    NS    ns1.example.com.
example.com.    3600    IN    NS    ns2.example.com.
```

#### Text Records
**TXT Record** (Text Information)
```
example.com.    3600    IN    TXT    "v=spf1 include:_spf.google.com ~all"
_dmarc.example.com. 3600 IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"
```

#### Service Records
**SRV Record** (Service Location)
```
_service._protocol.domain    TTL    IN    SRV    priority weight port target
_http._tcp.example.com.      3600   IN    SRV    10       5      80   web1.example.com.
```

#### Pointer Records
**PTR Record** (Reverse DNS)
```
100.1.168.192.in-addr.arpa.    3600    IN    PTR    example.com.
```

#### Start of Authority
**SOA Record** (Zone Information)
```
example.com.    3600    IN    SOA    ns1.example.com. admin.example.com. (
    2023082701    ; Serial number
    3600          ; Refresh interval
    1800          ; Retry interval
    604800        ; Expire time
    86400         ; Minimum TTL
)
```

### DNS Query Process

#### Recursive Query Example
1. **User types**: www.example.com in browser
2. **Operating System**: Checks local DNS cache
3. **Local DNS Resolver**: Checks cache, if miss continues
4. **Root Server Query**: Where is .com?
5. **Root Response**: Ask TLD server at 192.5.6.30
6. **TLD Server Query**: Where is example.com?
7. **TLD Response**: Ask ns1.example.com at 192.168.1.10
8. **Authoritative Query**: What is www.example.com?
9. **Authoritative Response**: 192.168.1.100
10. **Response Chain**: Back through resolver to user

#### Query Types
**Recursive Query**
- Resolver does all the work
- Client gets final answer
- Most common for end users

**Iterative Query**
- Client follows referrals
- Used between nameservers
- More efficient for servers

### DNS Message Structure

```
Header (12 bytes):
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                      ID                       |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    QDCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    ANCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    NSCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    ARCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

Flags:
QR: Query (0) or Response (1)
AA: Authoritative Answer
TC: Truncated
RD: Recursion Desired
RA: Recursion Available
RCODE: Response Code (0=No Error, 3=NXDOMAIN)
```

### DNS Caching and TTL

#### Time to Live (TTL)
- **Purpose**: Controls cache duration
- **Common Values**:
  - A/AAAA records: 300-3600 seconds
  - MX records: 3600-86400 seconds
  - NS records: 86400+ seconds
- **Trade-offs**: Lower TTL = faster updates, higher load

#### Cache Levels
1. **Browser Cache**: Very short-term
2. **OS Cache**: System-level caching
3. **Local Resolver**: ISP or corporate DNS
4. **Authoritative Servers**: Original source

#### Negative Caching
- **NXDOMAIN**: Domain doesn't exist
- **NODATA**: Domain exists but no records of requested type
- **TTL Source**: SOA minimum TTL field

### DNS Security

#### DNS Security Extensions (DNSSEC)
- **Purpose**: Authenticate DNS responses
- **Cryptographic Signatures**: Verify data integrity
- **Chain of Trust**: From root to domain
- **Record Types**: DNSKEY, RRSIG, DS, NSEC/NSEC3

#### Common Attacks
**DNS Spoofing/Cache Poisoning**
- Inject false DNS responses
- Redirect users to malicious sites
- Mitigation: DNSSEC, query randomization

**DNS Tunneling**
- Exfiltrate data through DNS queries
- Bypass firewalls
- Detection: Monitor unusual DNS patterns

**DDoS Attacks**
- Overwhelm DNS servers
- Amplification attacks using open resolvers
- Mitigation: Rate limiting, anycast

### DNS Tools and Commands

#### Command Line Tools
```bash
# Basic lookup
nslookup example.com

# Detailed query
dig example.com A +short
dig example.com MX
dig @8.8.8.8 example.com

# Reverse lookup
dig -x 8.8.8.8

# Trace query path
dig +trace example.com

# Check all record types
dig example.com ANY
```

#### Python DNS Example
```python
import socket
import dns.resolver

# Basic resolution
ip = socket.gethostbyname('example.com')
print(f"IP: {ip}")

# Advanced queries using dnspython
resolver = dns.resolver.Resolver()
resolver.nameservers = ['8.8.8.8']

# A record
answers = resolver.resolve('example.com', 'A')
for answer in answers:
    print(f"A: {answer}")

# MX record
answers = resolver.resolve('example.com', 'MX')
for answer in answers:
    print(f"MX: {answer.preference} {answer.exchange}")
```

### DNS Load Balancing

#### Round Robin DNS
```
example.com.    300    IN    A    192.168.1.10
example.com.    300    IN    A    192.168.1.11
example.com.    300    IN    A    192.168.1.12
```

#### Geographic DNS
- Different responses based on client location
- Content Delivery Networks (CDN)
- Disaster recovery

#### Weighted Round Robin
- Different weights for different servers
- Gradual traffic shifting
- Canary deployments

### Modern DNS Features

#### DNS over HTTPS (DoH)
- Encrypt DNS queries over HTTPS
- Port 443, looks like web traffic
- Privacy protection

#### DNS over TLS (DoT)
- Encrypt DNS queries over TLS
- Port 853, dedicated protocol
- RFC 7858

#### Anycast DNS
- Same IP address advertised from multiple locations
- Automatic failover
- Reduced latency

## Interview Questions

1. **Q: Explain how DNS resolution works step by step.**
   **A:** Client checks local cache → recursive resolver checks cache → queries root server for TLD → queries TLD server for authoritative nameserver → queries authoritative server for record → response propagates back with caching at each level.

2. **Q: What is the difference between A and CNAME records?**
   **A:** A records map domain names directly to IPv4 addresses. CNAME records create aliases pointing to other domain names. A record is terminal; CNAME must resolve to an A record eventually.

3. **Q: How does DNS caching work and why is TTL important?**
   **A:** DNS responses are cached at multiple levels (browser, OS, resolver) for the duration specified by TTL. Lower TTL enables faster updates but increases DNS server load. Higher TTL reduces load but slows propagation of changes.

4. **Q: What is the purpose of MX records and how do they work?**
   **A:** MX records specify mail servers for a domain with priority values. Lower numbers indicate higher priority. Mail servers try highest priority first, falling back to lower priorities if needed.

5. **Q: Explain the DNS hierarchy and why it's designed this way.**
   **A:** Root servers (.) → TLD servers (.com) → Authoritative servers (example.com). This distributed hierarchy provides scalability, fault tolerance, and administrative delegation without single points of failure.

6. **Q: What is reverse DNS and when is it used?**
   **A:** Reverse DNS maps IP addresses back to domain names using PTR records in special domains (in-addr.arpa for IPv4). Used for email server verification, logging, security, and troubleshooting.

7. **Q: How does DNSSEC provide security and what problems does it solve?**
   **A:** DNSSEC uses cryptographic signatures to authenticate DNS responses, preventing cache poisoning and man-in-the-middle attacks. It creates a chain of trust from root to domain but doesn't provide confidentiality.

8. **Q: What is the difference between recursive and iterative DNS queries?**
   **A:** Recursive queries ask the resolver to do all the work and return the final answer. Iterative queries return referrals, requiring the client to follow the chain of nameservers manually.

9. **Q: How can DNS be used for load balancing?**
   **A:** Round-robin DNS returns multiple A records for load distribution. Geographic DNS returns different IPs based on client location. Weighted round-robin assigns different probabilities to different servers.

10. **Q: What causes DNS propagation delay and how can it be minimized?**
    **A:** Propagation delay occurs due to caching at multiple levels and TTL values. Minimize by reducing TTLs before changes, using shorter TTLs for frequently changing records, and understanding cache hierarchies.

11. **Q: Explain DNS over HTTPS (DoH) and its benefits.**
    **A:** DoH encrypts DNS queries over HTTPS (port 443), providing privacy and preventing eavesdropping. It looks like regular web traffic, making it harder to block, but can complicate network monitoring and filtering.

12. **Q: How do you troubleshoot DNS issues?**
    **A:** Use tools like nslookup, dig, and ping. Check local cache, verify nameserver responses, trace query path with +trace, check TTL values, verify record types, and test from different locations and resolvers.
