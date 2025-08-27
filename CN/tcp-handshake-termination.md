# TCP 3-way Handshake & 4-way Termination

Brief: TCP connection establishment and termination are critical processes that ensure reliable communication between hosts. The 3-way handshake synchronizes sequence numbers and establishes connection state, while the 4-way termination gracefully closes connections and prevents data loss.

## Detailed Theory

### TCP Connection States

#### Connection State Machine
```
CLOSED → SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
    ↓        ↓           ↓             ↓            ↓           ↓
LISTEN → SYN_RCVD → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
```

#### State Descriptions
- **CLOSED**: No connection exists
- **LISTEN**: Server waiting for connection requests
- **SYN_SENT**: Client sent SYN, waiting for SYN-ACK
- **SYN_RCVD**: Server received SYN, sent SYN-ACK, waiting for ACK
- **ESTABLISHED**: Connection established, data transfer can occur
- **FIN_WAIT_1**: Connection termination started, FIN sent
- **FIN_WAIT_2**: Received ACK for FIN, waiting for peer's FIN
- **CLOSE_WAIT**: Received FIN, sending ACK, application should close
- **LAST_ACK**: Sent FIN after CLOSE_WAIT, waiting for ACK
- **TIME_WAIT**: Connection closed, waiting for delayed packets
- **CLOSING**: Simultaneous close, both sent FIN

### TCP 3-Way Handshake

#### Purpose and Goals
- **Synchronize sequence numbers**: Both sides agree on starting sequence numbers
- **Establish connection parameters**: Window size, MSS, options
- **Verify reachability**: Ensure both directions work
- **Allocate resources**: Buffers, connection state

#### Handshake Process

**Step 1: SYN (Synchronize)**
```
Client → Server: SYN
TCP Header:
- SYN flag = 1
- Sequence number = ISN_client (Initial Sequence Number)
- ACK flag = 0
- Window size = client receive window
- Options: MSS, window scale, timestamps, etc.
```

**Step 2: SYN-ACK (Synchronize-Acknowledge)**
```
Server → Client: SYN-ACK
TCP Header:
- SYN flag = 1
- ACK flag = 1
- Sequence number = ISN_server
- Acknowledgment number = ISN_client + 1
- Window size = server receive window
- Options: MSS, window scale, timestamps, etc.
```

**Step 3: ACK (Acknowledge)**
```
Client → Server: ACK
TCP Header:
- SYN flag = 0
- ACK flag = 1
- Sequence number = ISN_client + 1
- Acknowledgment number = ISN_server + 1
- Window size = client receive window
- Data can be included (TCP Fast Open)
```

#### Detailed Packet Analysis
```
Packet 1: Client → Server
Flags: SYN
Seq: 1000 (ISN_client)
Ack: 0
Win: 65535
Options: MSS=1460, WS=7, TS, SACK

Packet 2: Server → Client
Flags: SYN, ACK
Seq: 2000 (ISN_server)
Ack: 1001 (ISN_client + 1)
Win: 65535
Options: MSS=1460, WS=7, TS, SACK

Packet 3: Client → Server
Flags: ACK
Seq: 1001
Ack: 2001 (ISN_server + 1)
Win: 65535
Data: HTTP GET request (optional)
```

#### Initial Sequence Number (ISN)
- **Purpose**: Prevent confusion with old connections
- **Generation**: Based on time, source/dest IP/port
- **RFC 6528**: ISN = M + F(source IP, dest IP, source port, dest port, secret)
- **Security**: Prevents sequence number prediction attacks

### TCP Options in Handshake

#### Maximum Segment Size (MSS)
```
Option Kind: 2
Option Length: 4
MSS Value: 1460 (typical for Ethernet)
```
- Largest TCP segment peer is willing to receive
- Default: 536 bytes if not specified
- Ethernet: 1460 (1500 MTU - 20 IP - 20 TCP)

#### Window Scale
```
Option Kind: 3
Option Length: 3
Shift Count: 7 (allows windows up to 65535 * 2^7 = 8MB)
```

#### Timestamps
```
Option Kind: 8
Option Length: 10
Timestamp Value: current time
Timestamp Echo Reply: 0 in SYN
```
- Used for RTT measurement
- Protection against wrapped sequence numbers (PAWS)

#### Selective Acknowledgment (SACK)
```
Option Kind: 4 (SACK Permitted)
Option Length: 2
```

### TCP 4-Way Termination

#### Graceful Connection Termination
TCP provides graceful shutdown allowing both sides to finish sending data before closing.

#### Termination Process

**Step 1: FIN from Active Closer**
```
Client → Server: FIN, ACK
TCP Header:
- FIN flag = 1
- ACK flag = 1 (acknowledging previous data)
- Sequence number = last_seq + 1
- Acknowledgment number = last_ack
```

**Step 2: ACK from Passive Closer**
```
Server → Client: ACK
TCP Header:
- FIN flag = 0
- ACK flag = 1
- Sequence number = server_seq
- Acknowledgment number = client_seq + 1 (FIN consumes sequence number)
```

**Step 3: FIN from Passive Closer**
```
Server → Client: FIN, ACK
TCP Header:
- FIN flag = 1
- ACK flag = 1
- Sequence number = server_seq
- Acknowledgment number = client_seq + 1
```

**Step 4: Final ACK from Active Closer**
```
Client → Server: ACK
TCP Header:
- FIN flag = 0
- ACK flag = 1
- Sequence number = client_seq + 1
- Acknowledgment number = server_seq + 1
```

#### Half-Close Scenario
- One side closes sending but continues receiving
- Allows peer to finish sending remaining data
- Common in file transfers and HTTP responses

### TIME_WAIT State

#### Purpose and Duration
- **Prevent delayed packets**: Packets from closed connection affecting new connection
- **Ensure final ACK delivery**: If final ACK is lost, resend when peer retransmits FIN
- **Duration**: 2 × MSL (Maximum Segment Lifetime)
- **Typical MSL**: 30 seconds to 2 minutes (TIME_WAIT = 1-4 minutes)

#### TIME_WAIT Issues
- **Port exhaustion**: Active closer holds port in TIME_WAIT
- **Server scalability**: Many connections in TIME_WAIT consume memory
- **Load balancers**: Especially problematic for high-traffic servers

#### Mitigation Strategies
```bash
# Linux: Reuse TIME_WAIT sockets for outgoing connections
echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse

# Reduce TIME_WAIT duration (careful!)
echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout

# SO_REUSEADDR socket option
setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(optval));
```

### Simultaneous Operations

#### Simultaneous Open
```
Host A          Host B
SYN    →        ← SYN
       ← SYN-ACK
SYN-ACK →
       ← ACK
ACK    →
```

#### Simultaneous Close
```
Host A          Host B
FIN    →        ← FIN
       ← ACK
ACK    →        TIME_WAIT
TIME_WAIT
```

### Connection Reset (RST)

#### When RST is sent
- Connection to closed port
- Invalid sequence numbers
- Application calls close() with data pending
- Network errors or firewall rules

#### RST Packet
```
TCP Header:
- RST flag = 1
- Sequence number = 0 or expected sequence
- No acknowledgment
- Connection immediately terminated
```

### Security Considerations

#### SYN Flood Attack
- **Attack**: Send many SYN packets without completing handshake
- **Impact**: Exhaust server's SYN queue
- **Mitigation**: SYN cookies, rate limiting, firewalls

#### SYN Cookies
```
ISN = hash(src_ip, dst_ip, src_port, dst_port, time, secret)
```
- Stateless SYN handling
- Encode connection info in sequence number
- Validate in returning ACK

#### Sequence Number Prediction
- **Attack**: Predict ISN to hijack connections
- **Mitigation**: Cryptographically secure ISN generation
- **Modern systems**: RFC 6528 compliant generators

### Practical Examples

#### Wireshark Analysis
```
# Capture TCP handshake
tcpdump -i eth0 "tcp[tcpflags] & tcp-syn != 0"

# Display connection states
netstat -an | grep :80

# Monitor TCP connections
ss -tuln
```

#### Python Socket Example
```python
import socket
import time

# Client demonstrating handshake
def tcp_client():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    # SYN sent here
    print("Connecting...")
    sock.connect(('127.0.0.1', 8080))
    print("Connected (handshake complete)")
    
    # Send data
    sock.send(b'Hello Server')
    response = sock.recv(1024)
    
    # FIN sent here
    print("Closing...")
    sock.close()
    print("Closed (termination complete)")

# Server demonstrating states
def tcp_server():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    
    # LISTEN state
    server.bind(('127.0.0.1', 8080))
    server.listen(5)
    print("Server listening...")
    
    while True:
        # Accept completes handshake
        client, addr = server.accept()
        print(f"Connection from {addr}")
        
        data = client.recv(1024)
        client.send(b'Hello Client')
        
        # Graceful close
        client.close()
```

#### Connection Monitoring
```bash
# View connection states
netstat -an | awk '/tcp/ {print $6}' | sort | uniq -c

# Monitor specific connection
ss -o state time-wait

# TCP statistics
cat /proc/net/sockstat
```

## Interview Questions

1. **Q: Explain the TCP 3-way handshake step by step.**
   **A:** Client sends SYN with initial sequence number → Server responds with SYN-ACK (acknowledges client's SYN and sends its own) → Client sends ACK to acknowledge server's SYN. This synchronizes sequence numbers and establishes connection state.

2. **Q: Why does TCP use a 3-way handshake instead of 2-way?**
   **A:** 3-way prevents issues with delayed connection requests. If only 2-way was used, old SYN packets could create unwanted connections. The third ACK confirms the client received the server's SYN-ACK and wants to proceed.

3. **Q: What is the purpose of TIME_WAIT state and why is it 2×MSL?**
   **A:** TIME_WAIT ensures delayed packets from closed connections don't interfere with new connections using the same port pair. 2×MSL allows maximum time for packets to traverse network in both directions and be discarded.

4. **Q: Explain the difference between active and passive close in TCP termination.**
   **A:** Active close is initiated by the side that sends the first FIN (enters FIN_WAIT_1 state). Passive close responds to the FIN (enters CLOSE_WAIT state). Active closer goes through TIME_WAIT; passive closer doesn't.

5. **Q: What happens during a simultaneous close?**
   **A:** Both sides send FIN at the same time. Each side transitions to CLOSING state instead of FIN_WAIT_2. After receiving and acknowledging each other's FIN, both go to TIME_WAIT state.

6. **Q: How does TCP prevent old duplicate segments from corrupting new connections?**
   **A:** Initial sequence numbers are chosen based on time and connection parameters. TIME_WAIT state prevents port reuse until delayed packets expire. Modern ISN generation uses cryptographic functions for security.

7. **Q: What is a SYN flood attack and how can it be mitigated?**
   **A:** Attacker sends many SYN packets without completing handshake, exhausting server's connection queue. Mitigated with SYN cookies (stateless handling), rate limiting, and firewall protection.

8. **Q: Explain what happens when a client connects to a closed port.**
   **A:** Server's TCP stack responds with RST (reset) packet indicating no application is listening on that port. Client receives connection refused error and immediately terminates connection attempt.

9. **Q: Why might a server experience port exhaustion related to TIME_WAIT?**
   **A:** When server actively closes connections (unusual), it enters TIME_WAIT state holding the port for 2×MSL. High connection rates can exhaust available ports. Solutions include connection pooling and SO_REUSEADDR.

10. **Q: How does TCP handle the case where the final ACK in termination is lost?**
    **A:** The passive closer (in LAST_ACK state) will retransmit its FIN. The active closer (in TIME_WAIT) will respond with ACK again. TIME_WAIT duration ensures this process can complete.

11. **Q: What TCP options are typically negotiated during the handshake?**
    **A:** Maximum Segment Size (MSS), Window Scale factor, Timestamps for RTT measurement, Selective Acknowledgment (SACK) permission, and TCP Fast Open cookies.

12. **Q: How does half-close work in TCP and when is it useful?**
    **A:** One side calls shutdown() to stop sending but continues receiving. Useful when client finishes sending request but needs to receive full response (like HTTP POST with large response).
