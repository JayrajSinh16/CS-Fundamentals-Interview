# Error Detection Methods

Brief: Detect corruption in frames/packets using redundancy.

## Key concepts
- Parity bits (odd/even) – simple, low coverage
- Checksums – Internet checksum (TCP/UDP)
- CRC – Robust polynomial-based detection (Ethernet CRC32)
- Hamming codes – detection + correction

## Practical example
- Ethernet frame includes CRC32; corrupted frames are dropped by NIC.

## Common interview Q&A
- Checksum vs CRC? Why CRC better at burst error detection?
