---
title: "Bring Your Own Webpage to SOAR"
linkTitle: "BYO-webpage"
date: 2025-05-28T11:17:12-05:00
weight: 50
description: "Deploy a Flask web application behind Nginx with SSL support on Rocky Linux"
---

## Overview

This comprehensive guide demonstrates how to deploy a Flask web application behind an Nginx reverse proxy with SSL termination on Rocky Linux systems. The deployment bypasses SOAR authentication mechanisms, requiring careful security consideration and implementation at your own discretion.

**Security Warning**: This configuration bypasses standard SOAR authentication protocols. Ensure proper access controls and security measures are implemented before deploying to production environments.

---

## Prerequisites

### System Requirements

- **Operating System**: Rocky Linux 8 or 9
- **Access Level**: Root or sudo privileges required
- **Network**: Outbound internet access for package installation

### Required Packages

The following packages must be installed on your system:

- `python3` and `pip3` - Python runtime and package manager
- `nginx` - Web server and reverse proxy (already installed)
- `flask` - Python web framework (installed via pip)

---

## Application Architecture

### Directory Structure

The application follows a structured layout for maintainability and security:

```
/opt/sase/
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
sudo mkdir -p /opt/sase/static
cd /opt/sase

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

@app.route("/sase/")
def sase_index():
    """Main application endpoint"""
    logger.info(f"Request to /sase/ from {request.remote_addr}")
    return jsonify({
        "status": "success",
        "message": "Flask application is running on /sase/",
        "version": "1.0.0"
    })

@app.route("/sase/health")
def health_check():
    """Health check endpoint for monitoring"""
    return jsonify({"status": "healthy", "service": "sase-flask-app"})

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
    app.run(debug=False, host='127.0.0.1', port=8001)
EOF
```

#### 1.3 WSGI Configuration

Create the WSGI entry point:

```bash
cat > wsgi.py <<'EOF'
#!/usr/bin/env python3
"""
WSGI configuration for SASE Flask application
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
chown -R nginx:nginx /opt/sase

# Set appropriate permissions
chmod -R 755 /opt/sase
```

### Step 2: Nginx Reverse Proxy Configuration

#### 2.1 Nginx Configuration

Add the following location block to `/etc/nginx/conf.d/cyops-api.conf`:

**Important**: Back up the existing configuration before making changes:

```bash
sudo cp /etc/nginx/conf.d/cyops-api.conf /etc/nginx/conf.d/cyops-api.conf.backup
```

Add this location block to the server configuration:

```nginx
# SASE Flask Application Proxy
location /sase/ {
    proxy_pass http://127.0.0.1:8001;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# Static files optimization (optional)
location /sase/static/ {
    alias /opt/sase/static/;
    expires 1d;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

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
cd /opt/sase
source venv/bin/activate

# Test the application locally
gunicorn --bind 127.0.0.1:8001 --workers 2 --timeout 30 wsgi:app

# In another terminal, test the endpoint
curl -I http://127.0.0.1:8001/sase/
```

#### 3.2 Gunicorn Configuration File

Create a configuration file for Gunicorn:

```bash
cat > /opt/sase/gunicorn.conf.py <<'EOF'
# Gunicorn configuration file
import multiprocessing

# Server socket
bind = "127.0.0.1:8001"
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
accesslog = "/var/log/sase/access.log"
errorlog = "/var/log/sase/error.log"
loglevel = "info"
access_log_format = '%(h)s %(l)s %(u)s %(t)s "%(r)s" %(s)s %(b)s "%(f)s" "%(a)s"'

# Process naming
proc_name = 'sase-flask-app'

# Daemon mode
daemon = False
pidfile = '/var/run/sase/sase.pid'
user = 'nginx'
group = 'nginx'
tmp_upload_dir = None

# SSL (if needed)
# keyfile = '/path/to/keyfile'
# certfile = '/path/to/certfile'
EOF

# Create log and run directories
sudo mkdir -p /var/log/sase /var/run/sase
sudo chown nginx:nginx /var/log/sase /var/run/sase
```

### Step 4: Systemd Service Configuration

#### 4.1 Service File Creation

Create a robust systemd service file:

```bash
sudo tee /etc/systemd/system/sase.service > /dev/null <<'EOF'
[Unit]
Description=SASE Flask Application via Gunicorn
Documentation=https://docs.gunicorn.org/
After=network.target
Requires=network.target

[Service]
Type=exec
User=nginx
Group=nginx
WorkingDirectory=/opt/sase
Environment=PATH=/opt/sase/venv/bin
Environment=PYTHONPATH=/opt/sase
ExecStart=/opt/sase/venv/bin/gunicorn --config /opt/sase/gunicorn.conf.py wsgi:app
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
ReadWritePaths=/var/log/sase /var/run/sase
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
sudo systemctl enable sase.service

# Start the service
sudo systemctl start sase.service

# Check service status
sudo systemctl status sase.service
```

## Testing and Validation

### Functional Testing

#### 1. Service Health Check

```bash
# Check if the service is running
sudo systemctl is-active sase.service

# Verify the application is listening on the correct port
sudo netstat -tlnp | grep 8001

# Test the health endpoint
curl -s http://127.0.0.1:8001/sase/health | python3 -m json.tool
```

### Expected Response

When accessing `https://your.server.ip/sase/`, you should receive:

```json
{
  "status": "success",
  "message": "Flask application is running on /sase/",
  "version": "1.0.0"
}
```

---

## Monitoring and Maintenance

### Log Management

#### Application Logs

```bash
# View Gunicorn access logs
sudo tail -f /var/log/sase/access.log

# View Gunicorn error logs
sudo tail -f /var/log/sase/error.log

# View systemd service logs
sudo journalctl -u sase.service -f
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
sudo tee /etc/logrotate.d/sase <<'EOF'
/var/log/sase/*.log {
    daily
    missingok
    rotate 30
    compress
    notifempty
    create 644 nginx nginx
    postrotate
        /bin/systemctl reload sase.service > /dev/null 2>&1 || true
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
  sudo chown -R nginx:nginx /opt/sase
  sudo chmod -R 755 /opt/sase
  ```
- **SELinux Context**: Check SELinux context on RHEL-based systems
  ```bash
  sudo setsebool -P httpd_can_network_connect 1
  sudo restorecon -R /opt/sase
  ```

#### 502 Bad Gateway Error

**Symptoms**: Nginx returns 502 Bad Gateway

**Causes and Solutions**:

- **Gunicorn Not Running**: Check if the service is active
  ```bash
  sudo systemctl status sase.service
  sudo systemctl restart sase.service
  ```
- **Port Binding Issues**: Verify no other service uses port 8001
  ```bash
  sudo netstat -tlnp | grep 8001
  sudo lsof -i :8001
  ```
- **Firewall Blocking**: Check firewall rules
  ```bash
  sudo firewall-cmd --list-all
  ```
