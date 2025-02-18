# TLS Setup Guide

This guide explains how to set up TLS/SSL certificates for your n8n and crawl4ai services using nginx reverse proxy with local domains.

## 1. Configure Local Domains

Add the following entries to your hosts file:

**For Linux/MacOS:**
```bash
sudo nano /etc/hosts
```
Add these lines:
```
127.0.0.1 n8n.local.io
127.0.0.1 crawl4ai.local.io
```

**For Windows:**
1. Open Notepad as Administrator
2. Open `C:\Windows\System32\drivers\etc\hosts`
3. Add the same lines:
```
127.0.0.1 n8n.local.io
127.0.0.1 crawl4ai.local.io
```

## 2. Generate SSL Certificates

1. Generate server key and certificate:
```bash
# Generate private key
openssl genrsa -out nginx/certs/nginx.key 2048

# Create certificate signing request (CSR)
openssl req -new -key nginx/certs/nginx.key \
  -out nginx/certs/nginx.csr \
  -subj "/C=US/ST=State/L=City/O=YourOrg/CN=n8n.local.io" \
  -addext "subjectAltName = DNS:n8n.local.io,DNS:crawl4ai.local.io"

# Create self-signed certificate
openssl x509 -req -in nginx/certs/nginx.csr \
  -signkey nginx/certs/nginx.key \
  -out nginx/certs/nginx.crt \
  -days 365 -sha256 \
  -extfile <(printf "subjectAltName=DNS:n8n.local.io,DNS:crawl4ai.local.io")
```

## 3. Set Proper Permissions

```bash
# Set secure permissions for certificates
chmod 600 nginx/certs/nginx.key
chmod 644 nginx/certs/nginx.crt
```

## 4. Verify Certificate Configuration

The certificates should match the nginx configuration:
```bash
# Check that certificate paths match nginx.conf
ls -l nginx/certs/nginx.key  # Should match ssl_certificate_key
ls -l nginx/certs/nginx.crt  # Should match ssl_certificate

# Verify certificate contents
openssl x509 -in nginx/certs/nginx.crt -text -noout
```

## 5. Testing

1. Test nginx configuration:
```bash
docker-compose exec nginx nginx -t
```

2. Test HTTPS connections:
```bash
# Test n8n endpoints
curl -k https://n8n.local.io
curl -k https://n8n.local.io:5678

# Test crawl4ai endpoint
curl -k https://crawl4ai.local.io
```

3. Test HTTP to HTTPS redirects:
```bash
# Should redirect to HTTPS
curl -I http://n8n.local.io
curl -I http://n8n.local.io:5678
curl -I http://crawl4ai.local.io
```

4. Access via browser:
- n8n interface: https://n8n.local.io:5678
- crawl4ai API: https://crawl4ai.local.io

## 6. Troubleshooting

1. Certificate issues:
```bash
# Verify certificate matches configuration
openssl x509 -in nginx/certs/nginx.crt -noout -text | grep "Subject Alternative Name"

# Check nginx logs for SSL errors
docker-compose logs nginx | grep "SSL"
```

2. Connection issues:
```bash
# Verify ports are listening
sudo netstat -tulpn | grep -E '80|443|5678'

# Test nginx upstream connections
docker-compose exec nginx curl -v http://n8n:5678
docker-compose exec nginx curl -v http://crawl4ai:11235
```

3. Domain resolution:
```bash
# Test local domain resolution
ping n8n.local.io
ping crawl4ai.local.io
```

## 7. Port Summary

The setup includes the following ports:
- 80: HTTP (redirects to HTTPS)
- 443: HTTPS for both services
- 5678: Additional port for n8n (both HTTP and HTTPS) # hack to be able to connect with Google OAuth API (redirect_uri requires full n8n URL)

All HTTP traffic is automatically redirected to HTTPS as configured in the nginx.conf.

## 8. Certificate Maintenance

Monitor certificate expiration:
```bash
openssl x509 -in nginx/certs/nginx.crt -noout -dates
```

Remember to regenerate certificates before they expire (365 days from generation).