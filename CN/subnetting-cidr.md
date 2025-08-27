# Subnetting & CIDR (DETAILED)

Brief: Subnetting splits a network into smaller segments by borrowing host bits; CIDR uses variable-length prefixes to allocate and aggregate efficiently. Master quick math, alignment, and summarization.

## Key concepts
- Borrow vs leave: New prefix = original prefix + borrowed bits. Hosts per subnet = 2^(host_bits_left) − 2 (traditional IPv4).
- Block size: 256 − mask_octet gives increment in that octet; use it to find subnet boundaries quickly.
- Alignment: Summarization (supernetting) works only when networks are contiguous and properly aligned (low bits all 0 at the summary length).
- VLSM: Allocate different prefix lengths per need; sort demands descending.
- Common masks: /30 (2 hosts), /29 (6), /28 (14), /27 (30), /26 (62), /24 (254), /23 (510), /22 (1022).

## Step-by-step examples
1) Split 192.168.1.0/24 into 4 equal subnets
- Need 4 subnets → borrow 2 bits (2^2 = 4) → new prefix /26 → mask 255.255.255.192
- Subnets:
  - 192.168.1.0/26  → hosts .1–.62, broadcast .63
  - 192.168.1.64/26 → hosts .65–.126, broadcast .127
  - 192.168.1.128/26 → hosts .129–.190, broadcast .191
  - 192.168.1.192/26 → hosts .193–.254, broadcast .255

2) Given 10.20.37.200/19 → network/broadcast/hosts
- /19 mask 255.255.224.0 (block 32 in 3rd octet)
- Third octet ranges: 0,32,64,96,128,160,192,224 → 37 ∈ 32–63
- Network 10.20.32.0/19; Broadcast 10.20.63.255; Hosts 2^(13)−2 = 8190; Range 10.20.32.1–10.20.63.254

3) VLSM from 172.16.0.0/24 for needs 60, 26, 12
- Sort by size: 60 → /26, 26 → /27, 12 → /28
- Assign:
  - 172.16.0.0/26 (hosts .1–.62)
  - 172.16.0.64/27 (hosts .65–.94)
  - 172.16.0.96/28 (hosts .97–.110)
- Remaining pool starts at 172.16.0.112/28

## CIDR aggregation examples
- 192.168.16.0/24 + 192.168.17.0/24 → 192.168.16.0/23
- Four /24s (192.168.0.0–192.168.3.0) → 192.168.0.0/22
- Two /25s (x.y.z.0/25 + x.y.z.128/25) → x.y.z.0/24

## Quick mental math tips
- Octet masks mapping: /25=128, /26=192, /27=224, /28=240, /29=248, /30=252.
- Hosts shortcut: For small subnets, 2^(8−borrowed) − 2 within last affected octet.
- Boundary memory aids: .0, .64, .128, .192 are /26; .0, .128 are /25.

## Practice problems (with solutions)
1) From 10.0.0.0/16, create 8 equal subnets. New prefix? Hosts each?
- Need 8 → borrow 3 → /19 → hosts 2^13 − 2 = 8190 each

2) What prefix gives ≈ 500 hosts per subnet?
- Need ≥ 502 → 2^9 = 512 → host bits 9 → /23

3) Network/broadcast for 172.31.5.201/20
- /20 → 255.255.240.0; block 16 in 3rd octet → network 172.31.0.0; broadcast 172.31.15.255

4) Can 192.168.4.0/23 and 192.168.6.0/23 be aggregated?
- Together they form 192.168.4.0/22 (alignment OK) → Yes

5) Allocate minimal subnets for needs 100, 50, 10 from 192.168.10.0/24
- 100 → /25 (126 hosts); 50 → /26 (62); 10 → /28 (14)
- One VLSM: 192.168.10.0/25, 192.168.10.128/26, 192.168.10.192/28

## Common interview Q&A
- How to find subnet quickly? Use block size = 256 − mask_octet at first non-255 octet.
- What is CIDR and why? Classless routing enabling flexible allocation and route aggregation to shrink routing tables.
- /30 vs /31? /30 gives 2 usable; /31 (RFC 3021) is for point-to-point (both usable, no broadcast).
