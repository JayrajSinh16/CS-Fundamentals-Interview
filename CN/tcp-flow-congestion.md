# TCP Flow & Congestion Control

Brief: TCP manages sender rate using receiver feedback and network signals.

## Key concepts
- Flow control: receiver window (rwnd)
- Congestion control: slow start, congestion avoidance, fast retransmit/recovery
- CWND vs RWND; effective window = min(cwnd, rwnd)

## Practical example
- Initial cwnd ~10 MSS (modern stacks). Packet loss halves cwnd; slow start resets after timeout.

## Common interview Q&A
- How does AIMD work? Difference between Reno/CUBIC? Why bufferbloat hurts latency?
