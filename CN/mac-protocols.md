# MAC Protocols (CSMA/CD, CSMA/CA)

Brief: Coordinate medium access to avoid/collide transmissions.

## Key concepts
- CSMA/CD – Ethernet (half-duplex legacy); sense carrier, transmit, detect collision, backoff
- CSMA/CA – Wi‑Fi; avoid collisions using RTS/CTS, ACKs, interframe spaces
- Hidden/exposed node problems

## Practical example
- Wi‑Fi uses exponential backoff and ACK per frame; collisions inferred via missing ACK.

## Common interview Q&A
- Why CSMA/CD obsolete? Full‑duplex switched Ethernet eliminates collisions.
