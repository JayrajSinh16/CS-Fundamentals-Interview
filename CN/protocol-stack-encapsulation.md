# Protocol Stack & Encapsulation

Brief: Data is wrapped with headers/trailers at each layer; reverse on receive.

## Key concepts
- Encapsulation order: App → Trans → Net → Link → Physical
- Headers: TCP (ports, seq), IP (src/dst), Ethernet (MAC)
- MTU/fragmentation: IP fragmentation (IPv4), Path MTU Discovery

## Practical example
- A 1KB HTTP payload becomes: TCP hdr (20B) + IP hdr (20B) + Eth hdr (14B) + FCS (4B)

## Common interview Q&A
- What info is in TCP/IP/Ethernet headers? How does MTU affect fragmentation?
