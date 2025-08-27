# Port Numbers

Brief: Identify application endpoints on hosts.

## Key concepts
- Well-known (0–1023), Registered (1024–49151), Dynamic/Ephemeral (49152–65535 typical)
- Common ports: HTTP 80/8080, HTTPS 443, DNS 53/UDP, SSH 22, FTP 20/21, SMTP 25, IMAP 143/993
- Multiplexing: sockets identified by 5-tuple (src/dst IP/port + protocol)

## Practical example
- Server binds fixed port; clients use ephemeral ports per connection.

## Common interview Q&A
- Why ephemeral ports? Can two apps bind same port? (SO_REUSEADDR semantics)?
