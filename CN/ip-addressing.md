# IP Addressing (DETAILED)

Brief: IP addressing uniquely identifies hosts and networks. IPv4 uses 32 bits (dotted decimal), IPv6 uses 128 bits (hex). Masks/CIDR split network vs host. Know private ranges, special addresses, and how to compute network, broadcast, and host ranges.

## Key concepts
- IPv4 classes (historical): A 0.0.0.0–127.255.255.255, B 128.0.0.0–191.255.255.255, C 192.0.0.0–223.255.255.255 (modern networks use classless CIDR).
- Private IPv4: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16. Special: Loopback 127.0.0.0/8, APIPA 169.254.0.0/16, Multicast 224.0.0.0/4, Limited broadcast 255.255.255.255.
- Subnet mask/CIDR: /n means n network bits. Usable hosts per subnet (traditional IPv4): 2^(host_bits) − 2 (network + broadcast).
- IPv6: 128-bit hex. Shortening: drop leading zeros; :: once per address. Link-local fe80::/10, Unique Local fc00::/7 (commonly fd00::/8), Multicast ff00::/8, Loopback ::1, Unspecified ::.
- ARP (IPv4) vs Neighbor Discovery (IPv6). IPv4 may fragment; IPv6 relies on PMTUD with source fragmentation only.

## Practical examples
1) Given 192.168.10.77/26
- /26 → mask 255.255.255.192; block size last octet = 256 − 192 = 64 → subnets .0, .64, .128, .192
- 77 is in range .64–.127 → Network 192.168.10.64, Broadcast 192.168.10.127
- First host .65, Last host .126, Hosts 2^(32−26)−2 = 62

2) /x to mask to hosts
- /24 → 255.255.255.0 → 254 hosts; /20 → 255.255.240.0 → 4094 hosts; /30 → 255.255.255.252 → 2 hosts

3) Valid vs invalid IPv4
- Valid: 10.0.0.1, 172.16.5.255, 192.168.1.0 (may be network addr depending on mask)
- Invalid: 256.1.1.1 (octet > 255), 1.2.3 (not 4 octets), 1.2.3.4.5 (too many), 10.01.1.1 (leading zero ambiguity)

4) IPv6 compression
- 2001:0db8:0000:0000:0000:ff00:0042:8329 → 2001:db8::ff00:42:8329

## Step-by-step subnet calculation
Example A: 10.10.34.129/20
- /20 → mask 255.255.240.0 (16 full bits + 4 in 3rd octet; 11110000 = 240)
- Block size in 3rd octet: 256 − 240 = 16 → subnets: 10.10.0.0, 10.10.16.0, 10.10.32.0, 10.10.48.0 ...
- 34 fits in 32–47 → Network 10.10.32.0/20; Broadcast 10.10.47.255; Hosts 2^12 − 2 = 4094; Range 10.10.32.1–10.10.47.254

Example B: 172.16.200.9/27
- /27 → mask 255.255.255.224; block size last octet = 32 → subnets .0, .32, .64, .96, .128, .160, .192, .224
- 200 fits in 192–223 → Network 172.16.200.192; Broadcast .223; Hosts 30; Range .193–.222

## CIDR notation examples
- Summarize two adjacent /25 networks x.y.z.0/25 and x.y.z.128/25 → x.y.z.0/24
- Summarize 192.168.8.0/24 and 192.168.9.0/24 → 192.168.8.0/23 (covers .8.0–.9.255)
- VLSM split 192.168.1.0/24 into /26 (62 hosts), /27 (30), /28 (14) as per needs

## Private vs public identification
- Private: 10.0.0.0/8, 172.16.0.0–172.31.255.255, 192.168.0.0/16. Others (excluding special) are public.

## Practice problems (with solutions)
1) 192.168.2.141/25 → Network? Broadcast? Hosts?
- /25 → 255.255.255.128; subnets .0–.127, .128–.255 → 141 in .128–.255
- Network 192.168.2.128; Broadcast .255; Hosts 126; Range .129–.254

2) From 10.0.0.0/24, how many /29 subnets? Hosts each?
- Borrow 5 bits (24→29) → 2^5 = 32 subnets; hosts 2^(3) − 2 = 6 each

3) Is 172.32.0.1 private? Is 172.20.5.9 private?
- 172.32.0.1: No; 172.20.5.9: Yes

4) Summarize 10.1.0.0/16 and 10.2.0.0/16
- Not adjacent; cannot summarize to a single prefix without also covering 10.0.0.0/15 or 10.0.0.0/14 depending on alignment. Adjacent pair 10.0.0.0/16 and 10.1.0.0/16 → 10.0.0.0/15.

5) IPv6: Which are valid? fe80::1, ::, 2001:db8:::1
- Valid: fe80::1, ::. Invalid: 2001:db8:::1 (multiple ::)

## Common interview Q&A
- How to quickly find network address? AND IP with mask; or use block size (256 − mask octet) to find subnet start.
- How many hosts in /x? 2^(32−x) − 2 (IPv4 traditional). For P2P, /31 usable (RFC 3021).
- IPv6 benefits? Vast space, simpler headers, no NAT needed, built-in IPsec, SLAAC.

---
Cheat: Mask octets progression → 255, 254, 252, 248, 240, 224, 192, 128, 0. Powers of two per bit: 128 64 32 16 8 4 2 1.