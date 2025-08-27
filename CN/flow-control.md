# Flow Control Mechanisms

Brief: Prevent fast senders from overwhelming slow receivers.

## Key concepts
- Stop-and-Wait – simple, poor utilization on high BDP links
- Sliding Window – multiple in-flight frames; window advertised by receiver
- TCP flow control – receive window (rwnd) in headers

## Practical example
- TCP receiver shrinks rwnd when app reads slowly, throttling sender.

## Common interview Q&A
- Flow vs congestion control? How does TCP window scaling work?
