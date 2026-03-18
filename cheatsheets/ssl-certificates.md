# SSL/TLS Administration & Certificate Management

Tools and commands for inspecting, generating, and verifying SSL/TLS certificates and keys.

## 1. Certificate & Key Inspection
* `openssl x509 -in cert.crt -text -noout` - View full certificate details including validity, subject, and issuer.
* `openssl x509 -in cert.crt -dates -noout` - Quickly check the `notBefore` and `notAfter` (expiration) dates.
* `openssl x509 -in cert.crt -subject -noout` - Display the owner's identity information (C, ST, L, O, OU, CN).
* `openssl x509 -in cert.crt -issuer -noout` - Display information about the Certificate Authority (CA).
* `openssl rsa -in server.key -check` - Verify the integrity of a private key file.

## 2. Generation & Renewal
* `openssl req -x509 -new -nodes -days 365 -key server.key -out server.crt` - Renew/generate a self-signed cert using an existing private key.
* `openssl req -x509 -new -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt` - Generate a new 2048-bit key and self-signed cert simultaneously.
* `openssl req -new -key server.key -out server.csr` - Create a Certificate Signing Request (CSR) for an external CA.
* `openssl req -x509 -new -nodes -days 365 -key server.key -out server.crt -subj "/C=CH/ST=State/L=City/O=Org/CN=localhost"` - Generate a cert in one line (non-interactive mode).

## 3. Modulus Verification (Consistency Check)
* `openssl x509 -noout -modulus -in server.crt | openssl md5` - Calculate MD5 hash of the certificate's modulus.
* `openssl rsa -noout -modulus -in server.key | openssl md5` - Calculate MD5 hash of the private key's modulus.
* **Note:** Both MD5 hashes must match exactly for the certificate to work with the key.

## 4. Live Service Testing
* `openssl s_client -connect localhost:443` - Connect to a local/remote port to inspect the active SSL handshake and chain.
* `echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -dates` - Extract expiration dates from a live website.

## 5. Format Conversion
* `openssl pkcs12 -in cert.pfx -out cert.pem -nodes` - Convert PFX/P12 files (Windows/IIS) to PEM format (Nginx/Apache).
* `openssl x509 -inform der -in cert.der -out cert.pem` - Convert binary DER certificates to Base64 PEM format.

## 6. Nginx Maintenance Workflow
* `sudo nginx -t` - Verify Nginx configuration syntax and SSL file paths before applying changes.
* `sudo systemctl reload nginx` - Apply new certificates without dropping active client connections.
* `sudo systemctl restart nginx` - Force a full restart if a reload fails to pick up the updated certificate.