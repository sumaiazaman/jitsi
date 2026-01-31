# Jitsi Meet Docker Deployment

A self-hosted video conferencing solution using Jitsi Meet deployed with Docker Compose. This setup provides a complete, production-ready video conferencing platform with minimal configuration.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Usage](#usage)
- [Production Deployment](#production-deployment)
- [Troubleshooting](#troubleshooting)
- [Maintenance](#maintenance)

## 🎯 Overview

This project deploys a complete Jitsi Meet video conferencing system using Docker containers. Jitsi Meet is a fully encrypted, 100% open-source video conferencing solution that you can use all day, every day, for free — with no account needed.

### Features

- ✅ Fully self-hosted video conferencing
- ✅ End-to-end encryption
- ✅ No account required for guests
- ✅ Screen sharing and presentation mode
- ✅ Optional authentication for room creation
- ✅ Scalable architecture
- ✅ Easy Docker-based deployment

## 🏗️ Architecture

The deployment consists of four main services:

```
┌─────────────────────────────────────────────────────┐
│                    Jitsi Meet                       │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │   Web    │  │ Prosody  │  │ Jicofo   │         │
│  │ (Nginx)  │  │  (XMPP)  │  │(Control) │         │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘         │
│       │             │              │               │
│       └─────────────┴──────────────┘               │
│                     │                              │
│              ┌──────┴──────┐                       │
│              │     JVB     │                       │
│              │(Videobridge)│                       │
│              └─────────────┘                       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Service Components

1. **Web (jitsi/web)** - Port 8000
   - Nginx web server serving the Jitsi Meet web interface
   - Entry point for users accessing the video conferencing platform

2. **Prosody (jitsi/prosody)**
   - XMPP server handling signaling and authentication
   - Manages user sessions and room creation

3. **Jicofo (jitsi/jicofo)**
   - Jitsi Conference Focus
   - Manages media sessions and coordinates between components
   - Handles room creation and participant management

4. **JVB (jitsi/jvb)** - Port 10000/UDP
   - Jitsi Videobridge
   - Routes video and audio streams between participants
   - Handles the actual media traffic

## 📦 Prerequisites

- Docker Engine 20.10 or higher
- Docker Compose 1.29 or higher
- Minimum 2GB RAM
- Minimum 2 CPU cores
- Open ports: 8000 (HTTP), 10000/UDP (media)

For production:
- A domain name
- Ports 80, 443, and 10000/UDP accessible from the internet

## 🚀 Quick Start

### 1. Clone or Download

```bash
git clone git@github.com:sumaiazaman/jitsi.git
cd jitsi
```

### 2. Configure Environment

```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your settings
nano .env  # or use your preferred editor
```

### 3. Start Services

```bash
# Start all services
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f
```

### 4. Access Jitsi Meet

Open your browser and navigate to:
```
http://localhost:8000
```

You can now create or join video conferences!

## ⚙️ Configuration

### Environment Variables

All configuration is done through the `.env` file. Key variables:

#### Basic Settings

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `PUBLIC_URL` | Public URL for accessing Jitsi | `http://localhost:8000` | `https://meet.example.com` |
| `TZ` | Timezone | `UTC` | `America/New_York` |
| `HTTP_PORT` | HTTP port | `8000` | `80` |
| `HTTPS_PORT` | HTTPS port | `8443` | `443` |

#### SSL/TLS Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `ENABLE_LETSENCRYPT` | Enable Let's Encrypt SSL | `0` (disabled) |
| `ENABLE_HTTP_REDIRECT` | Redirect HTTP to HTTPS | `0` (disabled) |

#### Authentication Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `ENABLE_AUTH` | Require authentication to create rooms | `0` (disabled) |
| `ENABLE_GUESTS` | Allow guests to join rooms | `1` (enabled) |

#### Service Passwords

**⚠️ IMPORTANT:** Change these in production!

| Variable | Description |
|----------|-------------|
| `JICOFO_AUTH_PASSWORD` | Jicofo authentication password |
| `JVB_AUTH_PASSWORD` | Videobridge authentication password |
| `JIGASI_XMPP_PASSWORD` | Jigasi (SIP gateway) password |
| `JIBRI_XMPP_PASSWORD` | Jibri (recording) password |
| `JIBRI_RECORDER_PASSWORD` | Jibri recorder password |

Generate strong passwords:
```bash
openssl rand -hex 16
```

## 💻 Usage

### Creating a Conference

1. Open Jitsi Meet in your browser
2. Enter a room name (e.g., "my-meeting-room")
3. Click "Go" or press Enter
4. Share the URL with participants

### Joining a Conference

Participants can join by:
1. Clicking the shared meeting link
2. Entering their name (no account required)
3. Clicking "Join meeting"

### Common Operations

#### Stop Services
```bash
docker-compose down
```

#### Restart Services
```bash
docker-compose restart
```

#### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f web
docker-compose logs -f jvb
```

#### Update to Latest Version
```bash
docker-compose pull
docker-compose up -d
```

## 🌐 Production Deployment

### 1. Domain Setup

Update your `.env` file:
```bash
PUBLIC_URL=https://meet.yourdomain.com
ENABLE_LETSENCRYPT=1
ENABLE_HTTP_REDIRECT=1
HTTP_PORT=80
HTTPS_PORT=443
```

### 2. DNS Configuration

Point your domain to your server's IP:
```
A Record: meet.yourdomain.com → YOUR_SERVER_IP
```

### 3. Firewall Configuration

Open required ports:
```bash
# Ubuntu/Debian with UFW
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 10000/udp

# CentOS/RHEL with firewalld
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --permanent --add-port=10000/udp
sudo firewall-cmd --reload
```

### 4. Security Hardening

#### Enable Authentication
```bash
ENABLE_AUTH=1
ENABLE_GUESTS=1  # Allow guests to join authenticated rooms
```

#### Change All Passwords
```bash
JICOFO_AUTH_PASSWORD=$(openssl rand -hex 16)
JVB_AUTH_PASSWORD=$(openssl rand -hex 16)
JIGASI_XMPP_PASSWORD=$(openssl rand -hex 16)
JIBRI_XMPP_PASSWORD=$(openssl rand -hex 16)
JIBRI_RECORDER_PASSWORD=$(openssl rand -hex 16)
```

### 5. Deploy
```bash
docker-compose down
docker-compose up -d
```

## 🔧 Troubleshooting

### Services Won't Start

**Check logs:**
```bash
docker-compose logs
```

**Check if ports are already in use:**
```bash
sudo lsof -i :8000
sudo lsof -i :10000
```

### Can't Connect to Video/Audio

**Issue:** Firewall blocking UDP port 10000

**Solution:**
```bash
# Check if port is open
sudo netstat -tulpn | grep 10000

# Open the port
sudo ufw allow 10000/udp
```

### Let's Encrypt Certificate Issues

**Issue:** Certificate generation fails

**Requirements:**
- Domain must point to your server
- Ports 80 and 443 must be accessible from internet
- No other service using ports 80/443

**Check:**
```bash
# Test domain resolution
nslookup meet.yourdomain.com

# Check port accessibility
curl -I http://meet.yourdomain.com
```

### Poor Video Quality

**Possible causes:**
- Insufficient bandwidth
- Server resources (CPU/RAM)
- Network latency

**Solutions:**
- Increase server resources
- Check network bandwidth
- Consider deploying JVB closer to users

## 🛠️ Maintenance

### Backup Configuration

```bash
# Backup environment file
cp .env .env.backup

# Backup Docker volumes
docker run --rm -v jitsi_web_config:/data -v $(pwd):/backup ubuntu tar czf /backup/web_config_backup.tar.gz /data
docker run --rm -v jitsi_prosody_config:/data -v $(pwd):/backup ubuntu tar czf /backup/prosody_config_backup.tar.gz /data
```

### Update Services

```bash
# Pull latest images
docker-compose pull

# Recreate containers with new images
docker-compose up -d

# Remove old images
docker image prune -a
```

### Monitor Resources

```bash
# Check container resource usage
docker stats

# Check disk usage
docker system df
```

### Clean Up

```bash
# Remove stopped containers
docker-compose down

# Remove volumes (⚠️ This deletes all data!)
docker-compose down -v

# Clean up unused Docker resources
docker system prune -a
```

## 📚 Additional Resources

- [Official Jitsi Documentation](https://jitsi.github.io/handbook/)
- [Jitsi Docker Documentation](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker)
- [Jitsi Community Forum](https://community.jitsi.org/)
- [GitHub Repository](https://github.com/jitsi/docker-jitsi-meet)

## 📄 License

This project uses Jitsi Meet, which is licensed under the Apache License 2.0.

## 🤝 Support

For issues and questions:
- Check the [Troubleshooting](#troubleshooting) section
- Visit [Jitsi Community Forum](https://community.jitsi.org/)
- Review [GitHub Issues](https://github.com/jitsi/docker-jitsi-meet/issues)

---

**Made with ❤️ using Jitsi Meet**

