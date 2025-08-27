# SSL/TLS Handshake

Brief: SSL/TLS establishes secure communication channels through cryptographic handshakes that provide authentication, key exchange, and encryption parameter negotiation. Understanding TLS is crucial for web security, API design, and network troubleshooting.

## Detailed Theory

### SSL/TLS Evolution

#### Protocol Versions
- **SSL 1.0**: Never released (internal use only)
- **SSL 2.0**: 1995, deprecated due to security flaws
- **SSL 3.0**: 1996, deprecated due to POODLE attack
- **TLS 1.0**: 1999, successor to SSL 3.0
- **TLS 1.1**: 2006, fixes CBC attacks
- **TLS 1.2**: 2008, current standard, AES-GCM
- **TLS 1.3**: 2018, simplified handshake, enhanced security

#### Key Improvements in TLS 1.3
- Reduced handshake round trips (1-RTT, 0-RTT)
- Removed insecure cipher suites
- Perfect Forward Secrecy mandatory
- Simplified state machine
- Better resumption mechanisms

### Cryptographic Foundations

#### Symmetric Cryptography
- **Purpose**: Bulk data encryption (fast)
- **Algorithms**: AES-128/256, ChaCha20
- **Modes**: GCM (Galois/Counter Mode), CCM, Poly1305
- **Key sizes**: 128-bit, 256-bit

#### Asymmetric Cryptography
- **Purpose**: Key exchange, digital signatures
- **RSA**: Key exchange and signatures (being phased out for key exchange)
- **ECDSA**: Elliptic Curve Digital Signature Algorithm
- **ECDH/ECDHE**: Elliptic Curve Diffie-Hellman (Ephemeral)
- **Ed25519**: Modern signature algorithm

#### Hash Functions
- **Purpose**: Message integrity, key derivation
- **Algorithms**: SHA-256, SHA-384, SHA-512
- **HMAC**: Hash-based Message Authentication Code
- **HKDF**: HMAC-based Key Derivation Function

### TLS 1.2 Handshake

#### Full Handshake Process

**Step 1: ClientHello**
```
Client → Server
Message: ClientHello
Contents:
- Protocol Version: TLS 1.2
- Random: 32 bytes (4 timestamp + 28 random)
- Session ID: For session resumption
- Cipher Suites: Supported encryption algorithms
- Compression Methods: Usually none
- Extensions: SNI, ALPN, supported groups, etc.
```

**Example Cipher Suites:**
```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_RSA_WITH_AES_256_CBC_SHA256
```

**Step 2: ServerHello**
```
Server → Client
Message: ServerHello
Contents:
- Protocol Version: TLS 1.2
- Random: 32 bytes
- Session ID: Empty or for resumption
- Cipher Suite: Selected from client's list
- Compression Method: None
- Extensions: Response to client extensions
```

**Step 3: Server Certificate**
```
Server → Client
Message: Certificate
Contents:
- Certificate Chain: Server cert + intermediate CAs
- Public Key: For key exchange/verification
- Extensions: Key usage, SAN, etc.
```

**Step 4: ServerKeyExchange (if needed)**
```
Server → Client
Message: ServerKeyExchange
Contents:
- ECDH Parameters: Curve, public key
- Signature: Server's signature on parameters
- Only sent for ECDHE, DHE cipher suites
```

**Step 5: CertificateRequest (optional)**
```
Server → Client
Message: CertificateRequest
Contents:
- Certificate Types: RSA, ECDSA
- Supported Signature Algorithms
- Certificate Authorities
```

**Step 6: ServerHelloDone**
```
Server → Client
Message: ServerHelloDone
Contents: Empty (handshake boundary marker)
```

**Step 7: ClientKeyExchange**
```
Client → Server
Message: ClientKeyExchange
Contents:
- ECDH Public Key: Client's contribution
- Or Pre-master Secret: Encrypted with server's public key (RSA)
```

**Step 8: CertificateVerify (if client cert)**
```
Client → Server
Message: CertificateVerify
Contents:
- Signature: Proves client owns private key
- Covers all previous handshake messages
```

**Step 9: ChangeCipherSpec**
```
Client → Server
Message: ChangeCipherSpec (separate protocol)
Contents: Switching to negotiated cipher
```

**Step 10: Finished (Client)**
```
Client → Server
Message: Finished (encrypted)
Contents:
- Verify Data: PRF(master_secret, "client finished", handshake_messages_hash)
- First encrypted message
```

**Step 11: ChangeCipherSpec (Server)**
```
Server → Client
Message: ChangeCipherSpec
Contents: Server switching to cipher
```

**Step 12: Finished (Server)**
```
Server → Client
Message: Finished (encrypted)
Contents:
- Verify Data: PRF(master_secret, "server finished", handshake_messages_hash)
- Handshake complete
```

### TLS 1.3 Handshake

#### Simplified 1-RTT Handshake
```
Client                                               Server

ClientHello
+ key_share                     -------->
                                                ServerHello
                                               + key_share
                                     + {EncryptedExtensions}
                                     + {CertificateRequest*}
                                            + {Certificate*}
                                      + {CertificateVerify*}
                                <--------      + {Finished}
{Certificate*}
{CertificateVerify*}
{Finished}                      -------->
[Application Data]              <------->     [Application Data]
```

#### Key Improvements
- **Fewer round trips**: 1-RTT vs 2-RTT in TLS 1.2
- **0-RTT resumption**: Early data with session tickets
- **Forward secrecy**: Mandatory ECDHE/DHE
- **Simplified cipher suites**: Separate authentication and key exchange

### Key Exchange Mechanisms

#### RSA Key Exchange (deprecated)
```
1. Client generates pre-master secret
2. Encrypts with server's RSA public key
3. Server decrypts with RSA private key
4. Both derive master secret
Problem: No forward secrecy
```

#### ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)
```
1. Server generates ephemeral ECDH key pair
2. Client generates ephemeral ECDH key pair
3. Both exchange public keys
4. Compute shared secret using ECDH
5. Server signs parameters for authentication
Benefit: Perfect Forward Secrecy
```

### Key Derivation

#### TLS 1.2 Key Derivation
```
pre_master_secret = ECDH(client_private, server_public)
master_secret = PRF(pre_master_secret, "master secret", 
                   client_random + server_random)

key_block = PRF(master_secret, "key expansion",
               server_random + client_random)

Split key_block into:
- client_write_MAC_key
- server_write_MAC_key  
- client_write_encryption_key
- server_write_encryption_key
- client_write_IV
- server_write_IV
```

#### TLS 1.3 Key Schedule
```
                    0
                    |
                    v
       PSK ->   HKDF-Extract = Early Secret
                    |
                    +-----> Derive-Secret(., "ext binder" | "res binder", "")
                    |                     = binder_key
                    |
                    +-----> Derive-Secret(., "c e traffic", ClientHello)
                    |                     = client_early_traffic_secret
                    |
                    v
              Derive-Secret(., "derived", "")
                    |
                    v
(EC)DHE -> HKDF-Extract = Handshake Secret
                    |
                    +-----> Derive-Secret(., "c hs traffic", ClientHello...ServerHello)
                    |                     = client_handshake_traffic_secret
                    |
                    +-----> Derive-Secret(., "s hs traffic", ClientHello...ServerHello)
                    |                     = server_handshake_traffic_secret
```

### Certificate Validation

#### Certificate Chain Validation
1. **Parse certificate chain**: Extract certificates
2. **Build chain**: Link each cert to its issuer
3. **Validate signatures**: Verify each cert is signed by its issuer
4. **Check validity dates**: Not before/after timestamps
5. **Verify purpose**: Key usage, extended key usage
6. **Root trust**: Chain ends at trusted root CA
7. **Revocation checking**: CRL or OCSP

#### Subject Alternative Names (SAN)
```
X509v3 Subject Alternative Name:
    DNS:example.com
    DNS:www.example.com
    DNS:*.app.example.com
    IP Address:192.168.1.100
```

#### Certificate Pinning
```javascript
// HTTP Public Key Pinning (HPKP) - deprecated
Public-Key-Pins: pin-sha256="base64=="; pin-sha256="backup=="; max-age=2592000

// Certificate Transparency
Expect-CT: max-age=86400, enforce, report-uri="https://example.com/ct-report"
```

### TLS Extensions

#### Server Name Indication (SNI)
```
Extension: server_name (0x0000)
server_name: example.com
```
- Allows multiple SSL certificates on single IP
- Required for virtual hosting with SSL

#### Application-Layer Protocol Negotiation (ALPN)
```
Extension: application_layer_protocol_negotiation (0x0010)
protocols: h2, http/1.1
```
- Negotiate application protocol during handshake
- HTTP/2 requires ALPN

#### Supported Groups (Elliptic Curves)
```
Extension: supported_groups (0x000a)
groups: x25519, secp256r1, secp384r1
```

#### Signature Algorithms
```
Extension: signature_algorithms (0x000d)
algorithms: ecdsa_secp256r1_sha256, rsa_pss_rsae_sha256
```

### Session Management

#### Session Resumption (TLS 1.2)
```
# Session ID Resumption
ClientHello (with previous session_id)
ServerHello (same session_id) → Resume session

# Session Tickets (RFC 5077)
NewSessionTicket → Encrypted session state
ClientHello (with session ticket) → Resume
```

#### TLS 1.3 Session Tickets
```
NewSessionTicket {
    ticket_lifetime
    ticket_age_add
    ticket_nonce
    ticket
    extensions
}
```

### Security Considerations

#### Perfect Forward Secrecy (PFS)
- **Definition**: Past communications secure even if long-term keys compromised
- **Requirement**: Ephemeral key exchange (ECDHE, DHE)
- **TLS 1.3**: Mandatory PFS

#### Common Attacks and Mitigations

**BEAST (2011)**
- **Attack**: CBC cipher mode vulnerability in TLS 1.0
- **Mitigation**: Use TLS 1.1+ or RC4 (now deprecated)

**CRIME (2012)**
- **Attack**: Compression-based attack
- **Mitigation**: Disable TLS compression

**POODLE (2014)**
- **Attack**: Padding oracle in SSL 3.0
- **Mitigation**: Disable SSL 3.0

**Heartbleed (2014)**
- **Attack**: OpenSSL implementation bug
- **Mitigation**: Update OpenSSL, revoke certificates

**FREAK (2015)**
- **Attack**: Downgrade to export-grade RSA
- **Mitigation**: Disable weak cipher suites

### Practical Implementation

#### OpenSSL Examples
```bash
# Generate private key
openssl genpkey -algorithm RSA -out server.key -pkcs8 -aes256

# Generate certificate signing request
openssl req -new -key server.key -out server.csr -config server.conf

# Self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Test TLS connection
openssl s_client -connect example.com:443 -servername example.com

# Check certificate
openssl x509 -in cert.pem -text -noout

# Test cipher suites
nmap --script ssl-enum-ciphers -p 443 example.com
```

#### Python TLS Client
```python
import ssl
import socket

# Create SSL context
context = ssl.create_default_context()
context.check_hostname = True
context.verify_mode = ssl.CERT_REQUIRED

# Connect with TLS
with socket.create_connection(('example.com', 443)) as sock:
    with context.wrap_socket(sock, server_hostname='example.com') as ssock:
        print(f"TLS version: {ssock.version()}")
        print(f"Cipher: {ssock.cipher()}")
        cert = ssock.getpeercert()
        print(f"Subject: {cert['subject']}")
        
        # Send HTTP request
        ssock.send(b'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n')
        response = ssock.recv(4096)
        print(response.decode())
```

#### Nginx TLS Configuration
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Certificate files
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Modern configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /path/to/chain.pem;
}
```

## Interview Questions

1. **Q: Explain the TLS handshake process step by step.**
   **A:** ClientHello (cipher suites, random) → ServerHello + Certificate + ServerKeyExchange → ClientKeyExchange → Both send ChangeCipherSpec and Finished messages. This establishes encryption keys and verifies certificate authenticity.

2. **Q: What is the difference between TLS 1.2 and TLS 1.3?**
   **A:** TLS 1.3 has simplified handshake (1-RTT vs 2-RTT), removed insecure cipher suites, mandatory perfect forward secrecy, 0-RTT resumption, and separate authentication from key exchange algorithms.

3. **Q: How does Perfect Forward Secrecy work and why is it important?**
   **A:** PFS uses ephemeral keys (ECDHE/DHE) that are discarded after each session. Even if long-term private keys are compromised, past communications remain secure because session keys cannot be reconstructed.

4. **Q: What is Server Name Indication (SNI) and why is it needed?**
   **A:** SNI allows multiple SSL certificates on a single IP address by including the hostname in the TLS handshake. This enables virtual hosting with SSL, essential for shared hosting and CDNs.

5. **Q: Explain certificate chain validation process.**
   **A:** Validate each certificate's signature against its issuer, check validity dates, verify key usage, build chain to trusted root CA, and optionally check revocation status via CRL or OCSP.

6. **Q: What are cipher suites and how are they structured?**
   **A:** Cipher suites define cryptographic algorithms: key exchange + authentication + bulk encryption + MAC. Example: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 means ECDHE key exchange, RSA authentication, AES-256-GCM encryption.

7. **Q: How does session resumption work in TLS?**
   **A:** TLS 1.2 uses session IDs or session tickets to avoid full handshake. TLS 1.3 uses only session tickets with PSK (Pre-Shared Key) for 0-RTT or 1-RTT resumption.

8. **Q: What is ALPN and how does it differ from NPN?**
   **A:** ALPN (Application-Layer Protocol Negotiation) negotiates application protocol during TLS handshake. Unlike deprecated NPN, ALPN allows server to make the final protocol choice, required for HTTP/2.

9. **Q: How do you secure TLS against downgrade attacks?**
   **A:** Use TLS_FALLBACK_SCSV cipher suite, implement strict TLS version policies, use HSTS headers, and monitor for unexpected protocol downgrades in logs.

10. **Q: What is OCSP stapling and why is it beneficial?**
    **A:** OCSP stapling allows servers to fetch and include OCSP responses in TLS handshake, reducing client latency and improving privacy by avoiding direct client-to-CA OCSP requests.

11. **Q: How does key derivation work in TLS?**
    **A:** TLS uses key derivation functions (PRF in 1.2, HKDF in 1.3) to generate multiple keys from the master secret: encryption keys, MAC keys, and IVs for both client and server directions.

12. **Q: What are the security implications of using RSA key exchange vs ECDHE?**
    **A:** RSA key exchange lacks forward secrecy - compromised server key allows decryption of all past communications. ECDHE provides forward secrecy through ephemeral keys, making it mandatory in TLS 1.3.
