# Latency, Bandwidth, Throughput

Brief: Core performance metrics.

## Key concepts
- Latency: time to traverse path; Bandwidth: capacity; Throughput: achieved rate
- BDP = bandwidth × RTT; affects window sizes and buffering
- Jitter and packet loss impact on RTP/VoIP

## Practical example
- Tune TCP window for long fat networks (LFN) to fill the pipe.

## Common interview Q&A
- Why high bandwidth but low throughput? (window limits, loss, RTT)
