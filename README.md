# EUEM NGINX Configuration

This repository contains the nginx configuration for `euem.net`, which proxies requests to multiple backend services.

## Configuration Overview

The `euem.net` configuration file sets up:

### HTTP Server (Port 80)
- Handles Let's Encrypt ACME challenges for SSL certificate renewal
- Provides a `/healthz` endpoint for health checks
- Redirects all other traffic to HTTPS

### HTTPS Server (Port 443)
Routes traffic to different backend services based on URL paths:

| Path | Backend Port | Prefix Stripped | Description |
|------|--------------|-----------------|-------------|
| `/ssh/` | 3000 | Yes | SSH web interface |
| `/web-rtc/` | 3001 | Yes | WebRTC application |
| `/ssh-ws/` | 8000 | Yes | SSH WebSocket backend |
| `/rtc-ws/` | 8080 | Yes | WebRTC WebSocket backend |

### Key Features

**Prefix Stripping**: All paths use prefix stripping (trailing slash in `proxy_pass`). This means:
- Request to `https://euem.net/ssh/dashboard` sends `/dashboard` to the backend
- Backends don't need to know they're mounted under a subpath

**X-Forwarded-Prefix Header**: Set via a map for `/ssh` and `/web-rtc` paths, allowing apps to generate correct absolute URLs.

**WebSocket Support**: Configured with appropriate headers and timeouts for WebSocket connections.

## Managing NGINX

### Check Configuration Syntax

Before reloading or restarting nginx, always test the configuration:

```bash
sudo nginx -t
```

This checks for syntax errors without affecting the running service.

### Reload Configuration

After updating the config file, reload nginx to apply changes (no downtime):

```bash
sudo systemctl reload nginx
# or
sudo nginx -s reload
```

### Restart NGINX

Full restart (brief downtime):

```bash
sudo systemctl restart nginx
```

### Check NGINX Status

```bash
sudo systemctl status nginx
```

### Stop NGINX

```bash
sudo systemctl stop nginx
```

### Start NGINX

```bash
sudo systemctl start nginx
```

### View NGINX Logs

```bash
# Access logs
sudo tail -f /var/log/nginx/access.log

# Error logs
sudo tail -f /var/log/nginx/error.log
```

## Deployment Workflow

1. Edit the configuration file: `euem.net`
2. Test the syntax: `sudo nginx -t`
3. If test passes, reload: `sudo systemctl reload nginx`
4. If test fails, fix errors and repeat

## Configuration File Location

In production, this file should be placed in one of these locations:

- `/etc/nginx/sites-available/euem.net` (then symlinked to `sites-enabled/`)
- `/etc/nginx/conf.d/euem.net.conf`

## SSL Certificates

SSL certificates are managed by Let's Encrypt/Certbot:

- Certificate: `/etc/letsencrypt/live/euem.net/fullchain.pem`
- Private key: `/etc/letsencrypt/live/euem.net/privkey.pem`

### Renew Certificates

Certbot auto-renewal is typically set up via cron/systemd timer. Manual renewal:

```bash
sudo certbot renew
```

After renewal, reload nginx:

```bash
sudo systemctl reload nginx
```

## Backend Services

Ensure all backend services are running on their respective ports:

- Port 3000: SSH web interface
- Port 3001: WebRTC application
- Port 8000: SSH WebSocket backend
- Port 8080: WebRTC WebSocket backend

Check backend service status:

```bash
sudo systemctl status <service-name>
# or
sudo lsof -i :3000  # Check what's listening on port 3000
```

## Troubleshooting

### 502 Bad Gateway
- Check if backend services are running
- Verify backend ports are correct
- Check nginx error logs

### SSL Certificate Issues
- Verify certificate files exist and are readable
- Check certificate expiration: `sudo certbot certificates`
- Ensure Let's Encrypt can reach port 80 for renewals

### Configuration Not Taking Effect
- Ensure you ran `sudo systemctl reload nginx` after changes
- Check for syntax errors: `sudo nginx -t`
- Verify you edited the correct config file

