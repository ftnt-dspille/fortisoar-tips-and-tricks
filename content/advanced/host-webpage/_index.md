---
title: "Bring Your Own Webpage to SOAR"
linkTitle: "BYO-webpage"
date: 2025-05-28T11:17:12-05:00
weight: 50
description: "Deploy a Flask web application behind Nginx with SSL support on Rocky Linux"
---

## Overview

This guide deploys a Flask web app on a FortiSOAR appliance, fronted by the appliance's existing Nginx. The app sits beside FortiSOAR's own services and can optionally re-use FortiSOAR's authentication so only logged-in users can reach it.

**Two authentication options:**

1. **Recommended — Flask validates the JWT itself.** Your Flask app reads `Authorization: Bearer <jwt>` off each request and verifies it by calling `/api/3/actors/current` (the actual "who am I" endpoint — `/api/3/people/current` does **not** exist). A 200 means the caller is a logged-in FortiSOAR user; anything else, refuse the request. This is the only pattern that works reliably on a stock FortiSOAR appliance.
2. **Unauth fallback** — your app is reachable to anyone who can hit the appliance. Only choose this if the app is itself behind a separate VPN/firewall, runs only on `127.0.0.1`, or implements its own auth.

Both patterns are shown below. The path names used here (`/opt/byo-app/`, port `8002`, `byo-app.service`) are deliberately chosen to avoid colliding with FortiSOAR's existing `sase` service on the appliance.

{{% notice warning "nginx `auth_request` does not work here" %}}
You might be tempted to gate the location with `auth_request /api/3/actors/current` (or `…/people/current`) so that nginx itself rejects unauthenticated callers. **Don't.** The `/api/*` location on a FortiSOAR appliance proxies through a `try_files → @siteupcrud → @rewriteapp → /app.php/...` named-location chain. nginx's `auth_request` module only accepts `2xx`/`401`/`403` from the subrequest; the rewrite chain returns `404`, so the outer request becomes a confusing `500 Internal Server Error` even with a valid token. Do the JWT check inside your Flask app instead.
{{% /notice %}}

{{% notice warning "`https://localhost` is not always equivalent to the user-facing URL" %}}
The Flask app and the FortiSOAR API live on the same appliance, so it's tempting to have the app call `https://localhost/api/3/...`. That works on a **single-node** deployment where the user's browser hits the same appliance. In any **multi-appliance** or **gateway-fronted** deployment — including most lab and demo environments — the appliance you `ssh` into is **not** the one serving end-user traffic. A JWT issued by the local DAS will be rejected by the local `/api/*` (the local Symfony app trusts a different identity store). Call the **same external base URL the user's browser used** instead, and pass that URL into the app via an environment variable.
{{% /notice %}}

**Embedded inside the FortiSOAR designer.** If you load this app via a custom widget using the `cs-iframe="..." auth="true"` directive, FortiSOAR appends `?token=<jwt>` to the iframe URL on load. Your Flask app can read that token and use it as `Authorization: Bearer <token>` for any callbacks to `/api/3/*`. See the *Integration patterns* section at the end.

---

## Prerequisites

### System Requirements

- **Operating System**: Rocky Linux 8 or 9 (matches what FortiSOAR ships on)
- **Access Level**: Root or sudo privileges required
- **Network**: Outbound internet access for package installation
- **SELinux**: Enforcing by default — handled in Step 1.5 below; don't skip it

### Required Packages

The following packages must be installed on your system:

- `python3` and `pip3` - Python runtime and package manager
- `nginx` - Web server and reverse proxy (already installed on a FortiSOAR appliance)
- `flask` and `gunicorn` - Python web framework + WSGI server (installed via pip)

---

## Application Architecture

### Directory Structure

The application follows a structured layout for maintainability and security:

```
/opt/byo-app/
├── app.py              # Main Flask application
├── wsgi.py            # WSGI entry point
├── venv/             # Virtual environment (recommended)
└── static/           # Static assets directory (optional)
    ├── css/
    ├── js/
    └── images/
```

### Component Overview

- **Flask Application**: Serves dynamic content and API endpoints
- **Gunicorn**: WSGI HTTP server for Python applications
- **Nginx**: Reverse proxy, SSL termination, and static file serving
- **Systemd**: Service management for application lifecycle

---

## Implementation Steps

### Step 1: Flask Application Configuration

#### 1.1 Environment Setup

Create the application directory and establish proper permissions:

```bash
# Switch to root user
sudo su

# Create application directory
sudo mkdir -p /opt/byo-app/static
cd /opt/byo-app

# Create Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install flask gunicorn
```

#### 1.2 Application Code

Create the main Flask application file:

```bash
cat > app.py <<'EOF'
from flask import Flask, send_from_directory, jsonify, request
import os
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = Flask(__name__)
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY', 'dev-key-change-in-production')

@app.route("/byo-app/")
def byo_app_index():
    """Main application endpoint"""
    logger.info(f"Request to /byo-app/ from {request.remote_addr}")
    return jsonify({
        "status": "success",
        "message": "Flask application is running on /byo-app/",
        "version": "1.0.0"
    })

@app.route("/byo-app/health")
def health_check():
    """Health check endpoint for monitoring"""
    return jsonify({"status": "healthy", "service": "byo-app"})

@app.route("/byo-app/whoami")
def whoami():
    """JWT-gated endpoint — proves the caller is a logged-in FortiSOAR user.

    Pattern: forward the incoming Authorization header to FortiSOAR's
    /api/3/actors/current. If FortiSOAR returns 200, the caller is
    authenticated and the response body is their Actor record. Anything
    else, refuse.

    FSR_BASE_URL must be the *external* base URL the user's browser used,
    not https://localhost (see the topology warning at the top of this
    doc). Pass it in via an environment variable on the systemd unit.
    """
    import requests
    base = os.environ.get("FSR_BASE_URL", "").rstrip("/")
    if not base:
        return jsonify({"error": "FSR_BASE_URL not configured"}), 500
    auth = request.headers.get("Authorization", "")
    if not auth:
        return jsonify({"error": "missing Authorization header"}), 401
    r = requests.get(f"{base}/api/3/actors/current",
                     headers={"Authorization": auth, "Accept": "application/json"},
                     # Set verify=True with a real CA bundle in production.
                     verify=False, timeout=10)
    if r.status_code != 200:
        return jsonify({"error": "FortiSOAR rejected the token",
                        "upstream_status": r.status_code}), 401
    return jsonify({"actor": r.json()})

@app.route("/static/<path:path>")
def static_files(path):
    """Serve static files"""
    return send_from_directory('static', path)

@app.errorhandler(404)
def not_found(error):
    """Custom 404 handler"""
    return jsonify({"error": "Resource not found"}), 404

@app.errorhandler(500)
def internal_error(error):
    """Custom 500 handler"""
    logger.error(f"Internal server error: {error}")
    return jsonify({"error": "Internal server error"}), 500

if __name__ == "__main__":
    app.run(debug=False, host='127.0.0.1', port=8002)
EOF
```

#### 1.3 WSGI Configuration

Create the WSGI entry point:

```bash
cat > wsgi.py <<'EOF'
#!/usr/bin/env python3
"""
WSGI configuration for the BYO Flask application
"""
import sys
import os

# Add the application directory to Python path
sys.path.insert(0, os.path.dirname(__file__))

from app import app

if __name__ == "__main__":
    app.run()
EOF
```

#### 1.4 Set Proper Permissions

Configure appropriate file ownership and permissions:

```bash
# Set ownership to nginx user
chown -R nginx:nginx /opt/byo-app

# Set appropriate permissions
chmod -R 750 /opt/byo-app
```

`750` is enough because only `nginx` (the gunicorn user) needs to execute, and matches the permission set used by FSR's own services in `/opt/`.

#### 1.5 SELinux: allow nginx to talk to your Flask process

FortiSOAR appliances ship with SELinux enforcing. Two booleans matter:

```bash
# Allow nginx (and your gunicorn-under-nginx) to make outbound network
# connections — required if /byo-app/whoami calls back to FSR_BASE_URL.
sudo setsebool -P httpd_can_network_connect on

# Restore SELinux labels on the app directory so nginx can read it.
sudo restorecon -R /opt/byo-app
```

If `setsebool` is missing (`policycoreutils-python-utils` not installed), `dnf install -y policycoreutils-python-utils` first. Skipping this step is the #1 reason `/byo-app/whoami` returns a 502 with `Permission denied` in `/var/log/nginx/error.log`.

### Step 2: Nginx Reverse Proxy Configuration

#### 2.1 Why we use an include file, not a new `conf.d/*.conf`

FortiSOAR's `/etc/nginx/conf.d/cyops-api.conf` is one large `server {…}` block (it listens on `443 ssl` for `localhost`). Our location blocks have to live **inside that server block** — top-level `location` directives dropped into `conf.d/byo-app.conf` will fail `nginx -t` with `"location" directive is not allowed here` because everything in `conf.d/*.conf` is loaded at the `http {…}` scope.

The clean answer is to put the locations in a file outside `conf.d/` and reference it with a single `include` line from inside the existing server block. That keeps our change isolated, doesn't fork the upstream config, and is trivial to re-apply if an FSR upgrade drops a `cyops-api.conf.rpmnew`.

#### 2.2 Back up first

```bash
sudo cp /etc/nginx/conf.d/cyops-api.conf /etc/nginx/conf.d/cyops-api.conf.backup
```

#### 2.3 Create the locations file

```bash
sudo tee /etc/nginx/byo-app.locations > /dev/null <<'EOF'
# BYO Flask Application Proxy
location /byo-app/ {
    proxy_pass http://127.0.0.1:8002;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    # Forward the user's Authorization header so the Flask app can validate
    # the JWT against /api/3/actors/current. Without this, the app cannot
    # tell who the caller is.
    proxy_set_header Authorization $http_authorization;
}

# Static files optimization (optional)
location /byo-app/static/ {
    alias /opt/byo-app/static/;
    expires 1d;
    add_header Cache-Control "public, immutable";
    access_log off;
}
EOF
```

#### 2.4 Wire the include into `cyops-api.conf`

Open `/etc/nginx/conf.d/cyops-api.conf` and add this single line **just before the final closing `}`** of the `server` block (the one that opens with `listen 443 ssl;`):

```nginx
    include /etc/nginx/byo-app.locations;
```

If you prefer to script it, this `sed` works on a standard FSR appliance where the closing brace of the server block is the last line of the file:

```bash
sudo sed -i '$ i\    include /etc/nginx/byo-app.locations;' /etc/nginx/conf.d/cyops-api.conf
```

{{% notice info "Surviving FSR upgrades" %}}
An FSR RPM upgrade may write a new `cyops-api.conf` and stash yours as `cyops-api.conf.rpmnew`. If `/byo-app/` stops responding after an upgrade, diff the two files and re-add the one `include /etc/nginx/byo-app.locations;` line into the new copy. Because the locations file itself lives outside `conf.d/`, it is untouched by the upgrade.
{{% /notice %}}

#### 2.2 Configuration Validation

Test and reload the Nginx configuration:

```bash
# Test configuration syntax
sudo nginx -t

# If test passes, reload Nginx
sudo systemctl reload nginx

# Verify Nginx status
sudo systemctl status nginx
```

### Step 3: Gunicorn Application Server

#### 3.1 Manual Testing

Test the application manually before creating the service:

```bash
cd /opt/byo-app
source venv/bin/activate

# Test the application locally
gunicorn --bind 127.0.0.1:8002 --workers 2 --timeout 30 wsgi:app

# In another terminal, test the endpoint
curl -I http://127.0.0.1:8002/byo-app/
```

#### 3.2 Gunicorn Configuration File

Create a configuration file for Gunicorn:

```bash
cat > /opt/byo-app/gunicorn.conf.py <<'EOF'
# Gunicorn configuration file
import multiprocessing

# Server socket
bind = "127.0.0.1:8002"
backlog = 2048

# Worker processes
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = "sync"
worker_connections = 1000
timeout = 30
keepalive = 2

# Restart workers after this many requests, to help prevent memory leaks
max_requests = 1000
max_requests_jitter = 100

# Logging
accesslog = "/var/log/byo-app/access.log"
errorlog = "/var/log/byo-app/error.log"
loglevel = "info"
access_log_format = '%(h)s %(l)s %(u)s %(t)s "%(r)s" %(s)s %(b)s "%(f)s" "%(a)s"'

# Process naming
proc_name = 'byo-app'

# Daemon mode
daemon = False
pidfile = '/var/run/byo-app/byo-app.pid'
user = 'nginx'
group = 'nginx'
tmp_upload_dir = None

# SSL (if needed)
# keyfile = '/path/to/keyfile'
# certfile = '/path/to/certfile'
EOF

# Create log and run directories
sudo mkdir -p /var/log/byo-app /var/run/byo-app
sudo chown nginx:nginx /var/log/byo-app /var/run/byo-app
```

### Step 4: Systemd Service Configuration

#### 4.1 Service File Creation

Create a robust systemd service file:

```bash
sudo tee /etc/systemd/system/byo-app.service > /dev/null <<'EOF'
[Unit]
Description=BYO Flask Application via Gunicorn
Documentation=https://docs.gunicorn.org/
After=network.target
Requires=network.target

[Service]
Type=exec
User=nginx
Group=nginx
WorkingDirectory=/opt/byo-app
Environment=PATH=/opt/byo-app/venv/bin
Environment=PYTHONPATH=/opt/byo-app
# External base URL the user's browser hits — used by /byo-app/whoami to
# validate JWTs. On a single-node deployment this can be https://localhost.
# On a multi-appliance / lab-gateway deployment, set it to the user-facing URL.
Environment=FSR_BASE_URL=https://your.fortisoar.example.com
ExecStart=/opt/byo-app/venv/bin/gunicorn --config /opt/byo-app/gunicorn.conf.py wsgi:app
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true
Restart=always
RestartSec=10

# Security settings
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/log/byo-app /var/run/byo-app
PrivateDevices=true
ProtectKernelTunables=true
ProtectControlGroups=true
RestrictRealtime=true
SystemCallFilter=@system-service
SystemCallErrorNumber=EPERM

[Install]
WantedBy=multi-user.target
EOF
```

#### 4.2 Service Management

Enable and start the service:

```bash
# Reload systemd configuration
sudo systemctl daemon-reload

# Enable service to start on boot
sudo systemctl enable byo-app.service

# Start the service
sudo systemctl start byo-app.service

# Check service status
sudo systemctl status byo-app.service
```

## Testing and Validation

### Functional Testing

#### 1. Service Health Check

```bash
# Check if the service is running
sudo systemctl is-active byo-app.service

# Verify the application is listening on the correct port
sudo netstat -tlnp | grep 8002

# Test the health endpoint
curl -s http://127.0.0.1:8002/byo-app/health | python3 -m json.tool
```

### Expected Response

When accessing `https://your.server.ip/byo-app/`, you should receive:

```json
{
  "status": "success",
  "message": "Flask application is running on /byo-app/",
  "version": "1.0.0"
}
```

---

## Monitoring and Maintenance

### Log Management

#### Application Logs

```bash
# View Gunicorn access logs
sudo tail -f /var/log/byo-app/access.log

# View Gunicorn error logs
sudo tail -f /var/log/byo-app/error.log

# View systemd service logs
sudo journalctl -u byo-app.service -f
```

#### Nginx Logs

```bash
# View Nginx access logs
sudo tail -f /var/log/nginx/access.log

# View Nginx error logs
sudo tail -f /var/log/nginx/error.log
```

### Log Rotation

Configure log rotation to prevent disk space issues:

```bash
sudo tee /etc/logrotate.d/byo-app <<'EOF'
/var/log/byo-app/*.log {
    daily
    missingok
    rotate 30
    compress
    notifempty
    create 644 nginx nginx
    postrotate
        /bin/systemctl reload byo-app.service > /dev/null 2>&1 || true
    endscript
}
EOF
```

## Troubleshooting Guide

### Common Issues and Solutions

#### 403 Forbidden Error

**Symptoms**: HTTP 403 response when accessing the application

**Causes and Solutions**:

- **File Permissions**: Ensure nginx user can read application files
  ```bash
  sudo chown -R nginx:nginx /opt/byo-app
  sudo chmod -R 755 /opt/byo-app
  ```
- **SELinux Context**: Check SELinux context on RHEL-based systems
  ```bash
  sudo setsebool -P httpd_can_network_connect 1
  sudo restorecon -R /opt/byo-app
  ```

#### 502 Bad Gateway Error

**Symptoms**: Nginx returns 502 Bad Gateway

**Causes and Solutions**:

- **Gunicorn Not Running**: Check if the service is active
  ```bash
  sudo systemctl status byo-app.service
  sudo systemctl restart byo-app.service
  ```
- **Port Binding Issues**: Verify no other service uses port 8002
  ```bash
  sudo netstat -tlnp | grep 8002
  sudo lsof -i :8002
  ```
- **Firewall Blocking**: Check firewall rules
  ```bash
  sudo firewall-cmd --list-all
  ```
