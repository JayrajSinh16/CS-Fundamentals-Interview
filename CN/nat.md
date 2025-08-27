# NAT Working

Brief: Network Address Translation maps private to public addresses, conserving IPv4 space and adding basic isolation.

## Key concepts
- Static NAT, Dynamic NAT, PAT (NAT overload/port address translation)
- Connection tracking; translation table keyed by 5-tuple
- NAT traversal issues for P2P; STUN/TURN/ICE

## Practical example
- Home router PAT: 192.168.1.10:51512 → 203.0.113.5:51512

## Common interview Q&A
- Pros/cons of NAT? Impact on end-to-end principle and IPv6 adoption.
