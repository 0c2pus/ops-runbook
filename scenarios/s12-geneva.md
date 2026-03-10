# Scenario: "Geneva" - Renewing an SSL Certificate

## 🚩 Issue
The Nginx web server is serving a "Hello, World!" page over HTTPS, but the SSL certificate has expired. A new certificate must be generated and the Nginx configuration updated while maintaining the original Issuer and Subject information.

## 🔍 Investigation
- Verified Nginx configuration: `/etc/nginx/sites-available/default`
- Identified SSL certificate and key paths: 
    - `ssl_certificate /etc/nginx/ssl/nginx.crt;`
    - `ssl_certificate_key /etc/nginx/ssl/nginx.key;`
- Inspected the expired certificate metadata:
    - `Subject: CN = localhost, O = Acme, OU = IT Department, L = Geneva, ST = Geneva, C = CH`
- Confirmed expiration using: `openssl x509 -in /etc/nginx/ssl/nginx.crt -dates -noout`

## 🛠 Resolution
- Generated a new self-signed certificate using the existing private key to maintain the same Issuer/Subject:
  `sudo openssl req -x509 -new -nodes -days 365 -key /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt`
- Provided the exact metadata matching the original certificate during the interactive prompt:
    - Country (C): `CH`
    - State (ST): `Geneva`
    - Locality (L): `Geneva`
    - Organization (O): `Acme`
    - Unit (OU): `IT Department`
    - Common Name (CN): `localhost`
- Validated the new certificate expiration date: `openssl x509 -in /etc/nginx/ssl/nginx.crt -dates -noout`
- Tested Nginx configuration syntax: `sudo nginx -t`
- Reloaded Nginx to apply the new certificate: `sudo systemctl reload nginx`

## ✅ Verification
- Checked the live certificate via OpenSSL client:
  `echo | openssl s_client -connect localhost:443 2>/dev/null | openssl x509 -noout -dates`
- Verified the subject remained unchanged:
  `echo | openssl s_client -connect localhost:443 2>/dev/null | openssl x509 -noout -subject`

## 💡 Lessons Learned
- **Metadata Precision:** When renewing certificates for specific requirements, the Subject string must match exactly, including casing and abbreviations.
- **Service Reload:** Using `systemctl reload` is preferred over `restart` as it applies new SSL certificates without terminating existing connections.
- **OpenSSL Inspection:** Using `openssl x509` with the `-text` or `-subject` flags is the most reliable way to extract "recipes" from old certificates before renewal.