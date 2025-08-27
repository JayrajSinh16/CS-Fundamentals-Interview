# FTP Modes

Brief: File Transfer Protocol supports active and passive modes for data connection setup.

## Key concepts
- Control connection (TCP 21); data connection uses separate port(s)
- Active: server connects back to client (PORT) → firewall issues
- Passive: client connects to server (PASV) → firewall-friendly

## Practical example
- Modern deployments prefer FTPS/SFTP; passive mode behind NAT.

## Common interview Q&A
- Active vs passive differences? Why is SFTP not FTP over SSH but a different protocol?
