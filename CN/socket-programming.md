# Socket Programming (DETAILED)

Brief: Sockets provide endpoints for inter-process communication over the network. Know lifecycle (create → bind/connect → send/recv → close), TCP vs UDP behaviors, key APIs, and common pitfalls.

## Key concepts
- Socket types: TCP (stream, reliable, connection-oriented) vs UDP (datagram, best-effort, connectionless).
- Lifecycle (server TCP): socket() → bind() → listen() → accept() → recv()/send() → close().
- Lifecycle (client TCP): socket() → connect() → send()/recv() → close().
- UDP: socket() → bind() (server) or sendto() (client); use recvfrom()/sendto(); no handshake.
- Blocking vs non-blocking, timeouts, partial sends/reads, Nagle's algorithm (TCP_NODELAY), reuseaddr/port.

## Common functions
- POSIX-style: socket, bind, listen, accept, connect, send, recv, sendto, recvfrom, close, setsockopt, select/poll/epoll.
- Java: ServerSocket, Socket, DatagramSocket, DatagramPacket.
- Python: socket.socket, bind, listen, accept, connect, send, recv, sendto, recvfrom, settimeout.

## TCP examples

Java TCP Server (echo):
```java
import java.io.*;
import java.net.*;

public class TcpEchoServer {
    public static void main(String[] args) throws Exception {
        int port = 5000;
        try (ServerSocket server = new ServerSocket(port)) {
            System.out.println("Listening on " + port);
            while (true) {
                Socket client = server.accept();
                new Thread(() -> handle(client)).start();
            }
        }
    }
    static void handle(Socket client) {
        try (client;
             BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
             BufferedWriter out = new BufferedWriter(new OutputStreamWriter(client.getOutputStream()))) {
            String line;
            while ((line = in.readLine()) != null) {
                out.write("echo: " + line + "\n");
                out.flush();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Java TCP Client:
```java
import java.io.*;
import java.net.*;

public class TcpEchoClient {
    public static void main(String[] args) throws Exception {
        try (Socket s = new Socket("127.0.0.1", 5000);
             BufferedReader in = new BufferedReader(new InputStreamReader(s.getInputStream()));
             BufferedWriter out = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
             BufferedReader console = new BufferedReader(new InputStreamReader(System.in))) {
            String line;
            while ((line = console.readLine()) != null) {
                out.write(line + "\n");
                out.flush();
                System.out.println(in.readLine());
            }
        }
    }
}
```

Python TCP Server (echo):
```python
import socket

def run_server(host='127.0.0.1', port=5000):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        s.bind((host, port))
        s.listen()
        print(f'Listening on {host}:{port}')
        while True:
            conn, addr = s.accept()
            print('Connected by', addr)
            with conn:
                while True:
                    data = conn.recv(1024)
                    if not data:
                        break
                    conn.sendall(b'echo: ' + data)

if __name__ == '__main__':
    run_server()
```

Python TCP Client:
```python
import socket

def run_client(host='127.0.0.1', port=5000):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((host, port))
        for msg in [b'hello', b'world']:
            s.sendall(msg + b'\n')
            data = s.recv(1024)
            print('Received', data)

if __name__ == '__main__':
    run_client()
```

## UDP examples

Java UDP Echo:
```java
import java.net.*;

public class UdpEcho {
    public static void main(String[] args) throws Exception {
        int port = 6000;
        DatagramSocket socket = new DatagramSocket(port);
        byte[] buf = new byte[1024];
        System.out.println("UDP listening on " + port);
        while (true) {
            DatagramPacket packet = new DatagramPacket(buf, buf.length);
            socket.receive(packet);
            DatagramPacket reply = new DatagramPacket(packet.getData(), packet.getLength(),
                                                      packet.getAddress(), packet.getPort());
            socket.send(reply);
        }
    }
}
```

Python UDP Echo:
```python
import socket

def udp_echo(host='127.0.0.1', port=6000):
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
        s.bind((host, port))
        print(f'UDP listening on {host}:{port}')
        while True:
            data, addr = s.recvfrom(1024)
            s.sendto(data, addr)

if __name__ == '__main__':
    udp_echo()
```

## Socket lifecycle
- Create: socket(domain, type, protocol)
- Bind (server-side): assign local IP/port; failures: address in use → use SO_REUSEADDR
- Listen/Accept (TCP): move to passive open; backlog queue holds pending connections
- Connect (client): TCP 3-way handshake establishes connection; errors include timeout, refused
- Send/Receive: handle partial reads/writes; use loops or sendall in Python
- Close/Shutdown: FIN sequence; use shutdown for half-close when needed

## Error handling tips
- Timeouts: set socket timeouts; handle socket.timeout (Python) or SocketTimeoutException (Java)
- Non-blocking I/O: select/poll or NIO in Java for scalable servers
- Packet size: respect MTU; TCP streams don’t preserve message boundaries; frame your messages (length prefix, newline)
- Ports: ephemeral client ports; ensure server ports >1024 if no admin rights

## Common interview Q&A
- TCP vs UDP? Reliability/ordering/flow control vs low overhead/latency; pick based on use case (file transfer vs streaming/VoIP)
- What happens on accept()? New socket for the connection; listening socket remains open
- Nagle’s algorithm? Coalesces small packets; disable with TCP_NODELAY for latency-sensitive tiny writes
- How to avoid C10k issues? Non-blocking I/O, event loops, connection pooling, backpressure
- How to detect client disconnect? recv() returns 0 bytes (Python); readLine() returns null (Java)
