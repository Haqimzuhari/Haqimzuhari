# 🌐 Cloudflare Quick Tunnel Setup

A guide to setup Cloudflare Tunnel for exposing your local development server to the internet using a free Cloudflare subdomain.

---

## 📋 Prerequisites

- Cloudflare account (free)
- Local application running (e.g., localhost:8080)

---

## 🚀 Installation

### Step 1: Install cloudflared

Follow this instruction below:
1. Go to `https://developers.cloudflare.com/`
2. Search for `Quick Tunnel`
3. Follow the installation instruction based on your OS
---

## 🎯 Quick Start (One-Time)

If you just need a **temporary tunnel** without creating named tunnel:

```bash
cloudflared tunnel --url http://localhost:8080
```

This gives you instant public URL without any setup!

---

## 🔧 Run as Background Service (Optional)

### Linux (systemd)

```bash
# Install service
sudo cloudflared service install

# Start service
sudo systemctl start cloudflared

# Enable on boot
sudo systemctl enable cloudflared

# Check status
sudo systemctl status cloudflared
```

**Happy Tunneling! 🚀**