# DHCP Process

Brief: Automates IP configuration for hosts.

## Key concepts
- DORA: Discover → Offer → Request → Acknowledge
- Lease, renewal (T1/T2), options (router, DNS, domain)
- Relay agents (DHCP helpers) across subnets

## Practical example
- Client boots, broadcasts Discover; server Offers; client Requests; server ACKs with lease.

## Common interview Q&A
- How does DHCP work across routers? What happens if no DHCP server responds?
