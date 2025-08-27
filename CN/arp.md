# ARP

Brief: Address Resolution Protocol maps IPv4 addresses to MAC addresses within a LAN.

## Key concepts
- ARP request (broadcast) → ARP reply (unicast); ARP cache with timeouts
- Gratuitous ARP for duplicate detection/updates; Proxy ARP
- For IPv6, Neighbor Discovery replaces ARP

## Practical example
- First ping triggers ARP request for gateway MAC; subsequent packets use cached entry.

## Common interview Q&A
- ARP spoofing/poisoning risks? How ARP cache helps performance?
