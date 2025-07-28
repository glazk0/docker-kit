# Caddy

A modern, production-ready reverse proxy setup using Caddy web server with automatic HTTPS, HTTP/3 support, and comprehensive security headers.

## Overview

This Docker Compose configuration provides a robust reverse proxy solution using Caddy 2.10.0 with the following features:

- **Flexible Certificate Management**: Support for Let's Encrypt auto-renewal OR existing VPS certificates
- **HTTP/3 Support**: Latest HTTP protocol for improved performance
- **Cloudflare Integration**: Origin CA certificates with authenticated origin pulls
- **Security Headers**: HSTS, XSS protection, content type sniffing prevention, CSP
- **Compression**: Zstandard and Gzip compression for optimal bandwidth usage
- **Static Asset Caching**: Long-term caching with immutable headers for static resources
- **CORS Support**: Configurable cross-origin resource sharing with credentials support
- **Rate Limiting**: Built-in request throttling and IP-based rate limiting
- **Error Handling**: Custom error pages using http.cat integration
- **Structured Logging**: JSON logs with rotation and configurable levels
- **Production Ready**: Persistent volumes and health checks

## Quick Start

### Option 1: Using Existing VPS Certificates (Recommended for Production)

1. **Ensure your VPS has certificates in standard locations**:
   ```bash
   # Check your existing certificates
   ls -la /etc/ssl/certs/yourdomain.com.pem
   ls -la /etc/ssl/private/yourdomain.com.key
   ```

2. **Set Environment Variables**:
   ```bash
   cp .env.example .env
   # Edit .env with your values:
   # DOMAIN_NAME=yourdomain.com
   # EMAIL=your-email@example.com
   ```

3. **Create External Network**:
   ```bash
   docker network create caddy_network
   ```

4. **Start Caddy**:
   ```bash
   docker compose up -d
   ```

### Option 2: Using Let's Encrypt (Auto-renewal)

Follow the same steps but ensure no existing certificates conflict and Caddy will automatically obtain certificates from Let's Encrypt.

## Configuration

### Environment Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `DOMAIN_NAME` | Your domain name for SSL certificate | Yes | - |
| `EMAIL` | Email for Let's Encrypt registration | Yes | - |

### Certificate Options

This configuration supports multiple certificate sources:

#### Option 1: VPS System Certificates (Current Default)
```yaml
volumes:
  - /etc/ssl/certs:/etc/ssl/certs:ro      # System certificates
  - /etc/ssl/private:/etc/ssl/private:ro  # System private keys
```

**Requirements:**
- Certificates exist at `/etc/ssl/certs/{DOMAIN_NAME}.pem`
- Private key exists at `/etc/ssl/private/{DOMAIN_NAME}.key`
- For Cloudflare: Origin CA at `/etc/ssl/certs/cloudflare-origin-ca.pem`

#### Option 2: Local Certificate Management
```yaml
volumes:
  - ./ssl/certs:/etc/ssl/certs:ro     # Local certificates
  - ./ssl/private:/etc/ssl/private:ro # Local private keys
```

#### Option 3: Let's Encrypt (Automatic)
Remove certificate volume mounts and Caddy will automatically obtain certificates.

### Network Setup

The configuration uses two networks:
- `caddy_network`: External network for connecting other services  
- `default`: Internal bridge network for the Caddy container

To connect other services to Caddy, ensure they're on the `caddy_network`:

```yaml
networks:
  caddy_network:
    external: true
```

### Caddyfile Features

#### Global Configuration
- Automatic email configuration for Let's Encrypt
- Zstandard and Gzip compression enabled globally

#### Snippets (Reusable Blocks)

**`(default)`**: Basic compression settings
```caddyfile
encode zstd gzip
```

**`(static)`**: Static asset caching (60 days)
- Applies to: `.ico`, `.css`, `.js`, `.gif`, `.jpeg`, `.jpg`, `.png`, `.webp`, `.svg`, `.ttf`, `.json`

**`(security)`**: Comprehensive security headers
- HSTS with preload
- XSS protection
- Content type sniffing prevention
- Frame options (deny)
- Strict referrer policy
- Content Security Policy
- Server header hiding

**`(cors)`**: CORS support with preflight handling
- Configurable origin support
- Standard HTTP methods
- Custom headers support

**`(errors)`**: Custom error pages using http.cat
- Provides visual HTTP status code responses

### Current Routing

The current configuration routes traffic to a service named `www` on port `3002`. To modify this:

```caddyfile
{$DOMAIN_NAME} {
    encode zstd gzip
    handle {
        reverse_proxy your-service:port
    }
}
```

## Usage Examples

### Adding a New Service

1. **Add to your service's docker-compose.yml**:
   ```yaml
   networks:
     caddy_network:
       external: true
   ```

2. **Update Caddyfile**:
   ```caddyfile
   api.yourdomain.com {
       import default
       import security
       reverse_proxy api-service:8080
   }
   ```

3. **Reload Caddy**:
   ```bash
   docker-compose exec caddy caddy reload --config /etc/caddy/Caddyfile
   ```

### Multiple Domains

```caddyfile
example.com {
    import default
    import security
    reverse_proxy web-app:3000
}

api.example.com {
    import default
    import security
    import cors https://example.com
    reverse_proxy api-service:8080
}
```

### Static File Serving

```caddyfile
static.example.com {
    import default
    import security
    import static
    root * /srv/static
    file_server
}
```

## SSL Certificates

Caddy automatically obtains and renews SSL certificates from Let's Encrypt. Certificates are stored in the `caddy_data` volume and persist across container restarts.

### Certificate Locations
- Data: `/data` (mapped to `caddy_data` volume)
- Config: `/config` (mapped to `caddy_config` volume)

## Monitoring and Logs

### View Logs
```bash
docker-compose logs -f caddy
```

### Check Certificate Status
```bash
docker-compose exec caddy caddy list-certificates
```

### Reload Configuration
```bash
docker-compose exec caddy caddy reload --config /etc/caddy/Caddyfile
```

## Security Considerations

This configuration implements several security best practices:

1. **HSTS**: Forces HTTPS connections for 2 years with preload
2. **XSS Protection**: Blocks detected XSS attacks
3. **Content Sniffing**: Prevents MIME type confusion attacks
4. **Frame Options**: Prevents clickjacking with DENY policy
5. **Referrer Policy**: Limits referrer information leakage
6. **Comprehensive CSP**: Detailed Content Security Policy with safe defaults
7. **Permissions Policy**: Controls browser feature access
8. **Server Hiding**: Removes identifying server headers
9. **Certificate Security**: Read-only mounts and proper file permissions

### Important Security Notes

- **Certificate Protection**: All certificate volumes are mounted read-only (`:ro`)
- **Git Security**: The `.gitignore` file prevents accidental commit of certificates and environment files
- **File Permissions**: Ensure private keys have `600` permissions, certificates have `644`
- **Environment Variables**: Never commit `.env` files containing sensitive information

## File Structure & Git Management

```
caddy/
├── docker-compose.yaml    # Container orchestration
├── Caddyfile             # Caddy configuration
├── .env.example          # Environment template
├── .env                  # Your actual environment (gitignored)
└── README.md             # This documentation
```

**Important**: The `.gitignore` file at the repository root prevents committing:
- Environment files (`.env`, `*.env`)
- SSL certificates (`ssl/`, `*.pem`, `*.key`)
- Logs and temporary files
- Docker override files

## Performance Features

- **HTTP/3**: Latest protocol for reduced latency
- **Compression**: Zstandard (preferred) and Gzip fallback
- **Static Caching**: Long-term caching for static assets
- **Connection Reuse**: Efficient upstream connections

## Troubleshooting

### Common Issues

**Certificate Generation Failed**:
- Verify domain DNS points to your server
- Check firewall allows ports 80 and 443
- Ensure email is valid

**Service Connection Failed**:
- Verify service is on `caddy_network`
- Check service name and port in Caddyfile
- Confirm service is running

**Configuration Errors**:
```bash
docker-compose exec caddy caddy validate --config /etc/caddy/Caddyfile
```

### Debug Mode
Add to docker-compose.yml environment:
```yaml
environment:
  - CADDY_DEBUG=1
```

## File Structure

```
caddy/
├── docker-compose.yaml    # Container orchestration
├── Caddyfile             # Caddy configuration
└── README.md             # This documentation
```

## Credits

Configuration inspired by [rybbit-io/rybbit](https://github.com/rybbit-io/rybbit)

## License

This configuration is provided as-is for educational and production  use. You can find the full license in the root directory of this repository.
