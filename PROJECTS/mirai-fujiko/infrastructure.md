# Infrastructure

---

## Overview

```
Oracle Free VPS (ARM64)
  ├── Ubuntu 22.04 (recommended for ARM)
  ├── OpenClaw (agent runtime)
  ├── Nginx (reverse proxy + SSL)
  └── Certbot (Let's Encrypt SSL)

Your Notebook
  ├── FastAPI model router (:8000)
  ├── Parakeet v2 (STT)
  ├── Kokoro or Piper (TTS)
  ├── Ollama (LLMs)
  └── Cloudflare Tunnel (exposes :8000 to VPS)
```

---

## VPS Setup Checklist

### Initial VPS Configuration
- [ ] Ubuntu 22.04 installed (ARM64)
- [ ] Non-root user created with sudo
- [ ] SSH key auth only (disable password auth)
- [ ] UFW firewall configured:
  ```bash
  sudo ufw allow ssh
  sudo ufw allow 80
  sudo ufw allow 443
  sudo ufw enable
  # Port 18789 NOT exposed publicly (Nginx proxies it)
  ```
- [ ] Swap file created (2GB recommended):
  ```bash
  sudo fallocate -l 2G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  ```
- [ ] System updated: `sudo apt update && sudo apt upgrade -y`
- [ ] Timezone set: `sudo timedatectl set-timezone Europe/Berlin`

---

### Node.js Installation (required for OpenClaw)
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version  # Should show v22.x.x
```

---

### OpenClaw Installation
```bash
# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Verify
openclaw --version
openclaw doctor

# Run onboarding (interactive wizard)
openclaw onboard --install-daemon

# Onboarding choices:
# - Model provider: Anthropic (or your preferred)
# - Channels: Skip for now (add Telegram after Nginx setup)
# - Skills: Yes
# - Install as daemon: Yes (systemd service)
# - Hatch bot: Do later

# Verify gateway running
openclaw gateway status
# Should be listening on ws://127.0.0.1:18789
```

---

### Nginx Setup
```bash
sudo apt install nginx -y

# Create config
sudo nano /etc/nginx/sites-available/mirai
```

```nginx
server {
    server_name mirai.yourdomain.com;

    # WebChat UI
    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }

    listen 80;
}
```

```bash
sudo ln -s /etc/nginx/sites-available/mirai /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

### SSL (Let's Encrypt)
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d mirai.yourdomain.com

# Auto-renewal test
sudo certbot renew --dry-run
```

---

### Telegram Bot Setup
```bash
# 1. Create bot via @BotFather on Telegram
#    Command: /newbot
#    Name: Mirai (or "Mirai & Fujiko")
#    Username: your_mirai_bot
#    Save the token

# 2. Add to OpenClaw
openclaw onboard
# Select: Update values → Channels → Telegram
# Enter bot token

# 3. Restrict to your Telegram user only
# In OpenClaw config (~/.openclaw/config.json):
# "channels": {
#   "telegram": {
#     "allowFrom": ["YOUR_TELEGRAM_USER_ID"]
#   }
# }
```

---

### Agent Configuration (Mirai + Fujiko)
```bash
# After OpenClaw is running, configure two agents
# Edit: ~/.openclaw/config.json

# Key sections to set:
# agents.mirai.systemPrompt → paste from agents.md
# agents.fujiko.systemPrompt → paste from agents.md
# agents.mirai.keywords → routing keywords
# agents.fujiko.keywords → routing keywords
# scheduler.standup → "30 8 * * *" (8:30am daily)
# scheduler.wrap → triggered by keyword, not time
# scheduler.weekly → "30 8 * * 1" (8:30am Monday)
```

---

## Notebook Setup Checklist

### Cloudflare Tunnel Extension
```bash
# Assuming cloudflared already installed and running
# Add route for model router

# In your Cloudflare Zero Trust dashboard:
# Tunnels → your tunnel → Edit → Add public hostname:
#   Subdomain: models
#   Domain: yourdomain.com
#   Service: http://localhost:8000

# Or via config file, add to ~/.cloudflared/config.yml:
ingress:
  - hostname: models.yourdomain.com
    service: http://localhost:8000
  # ... your existing routes
  - service: http_status:404

# Restart tunnel
sudo systemctl restart cloudflared
```

### Model Router (FastAPI)
See `model-router.md` for full FastAPI code.

```bash
# Install dependencies
pip install fastapi uvicorn python-multipart httpx

# Run as service (systemd or PM2)
# See model-router.md for service setup

# Verify accessible from VPS:
# curl https://models.yourdomain.com/health
# Should return: {"status": "ok", "services": {...}}
```

### Parakeet v2 STT
```bash
# Install NVIDIA NeMo (Parakeet v2 requires it)
pip install nemo_toolkit[asr]

# Or use faster-whisper as alternative if NeMo is too heavy
pip install faster-whisper

# Test
# See model-router.md for integration
```

### Kokoro TTS
```bash
# Install Kokoro
pip install kokoro-onnx soundfile

# Test
python -c "from kokoro_onnx import Kokoro; k = Kokoro('kokoro-v0_19.onnx', 'voices.bin'); print('ok')"
```

### Ollama
```bash
# Install Ollama (Windows with WSL or native Linux)
curl -fsSL https://ollama.ai/install.sh | sh

# Pull a model
ollama pull mistral  # or llama3, qwen2.5, etc.

# Verify
ollama list
```

---

## Environment Variables

### VPS (.env in OpenClaw config dir)
```bash
# LLM APIs (fallback)
ANTHROPIC_API_KEY=your_key
OPENAI_API_KEY=your_key  # for Whisper + TTS fallback

# GitHub (vault access)
GITHUB_TOKEN=your_personal_access_token
GITHUB_REPO=ikuma/rectified-linear-focus
GITHUB_BRANCH=main

# Social posting
LINKEDIN_ACCESS_TOKEN=your_token
MEDIUM_INTEGRATION_TOKEN=your_token
THREADS_ACCESS_TOKEN=your_token  # when available

# Telegram
TELEGRAM_BOT_TOKEN=your_token

# Model router (your notebook via Cloudflare Tunnel)
MODEL_ROUTER_URL=https://models.yourdomain.com
MODEL_ROUTER_SECRET=your_shared_secret  # simple auth header

# Database
SQLITE_PATH=/home/youruser/.openclaw/mirai.db
```

### Notebook (.env for FastAPI router)
```bash
MODEL_ROUTER_SECRET=same_shared_secret_as_above
PARAKEET_MODEL_PATH=/path/to/parakeet-model
KOKORO_MODEL_PATH=/path/to/kokoro-model
OLLAMA_HOST=http://localhost:11434
OPENAI_API_KEY=your_key  # fallback only
```

---

## Monitoring & Maintenance

### Health Check Script (VPS cron)
```bash
# /home/youruser/scripts/health-check.sh
#!/bin/bash
OPENCLAW_STATUS=$(openclaw gateway status 2>&1)
if ! echo "$OPENCLAW_STATUS" | grep -q "running"; then
    openclaw gateway start
    echo "$(date): Restarted OpenClaw" >> ~/logs/health.log
fi
```

```bash
# Add to crontab: crontab -e
*/5 * * * * /home/youruser/scripts/health-check.sh
```

### OpenClaw Updates
```bash
# Check for updates weekly
openclaw --version
# Update when available (read release notes first)
npm update -g openclaw
openclaw doctor --repair  # fix any config issues after update
```

### Backup (weekly)
```bash
# /home/youruser/scripts/backup.sh
#!/bin/bash
DATE=$(date +%Y%m%d)
cp ~/.openclaw/mirai.db ~/backups/mirai-$DATE.db
# Optional: push to GitHub private repo
```

---

## Troubleshooting

### OpenClaw not starting
```bash
openclaw doctor
openclaw gateway status
journalctl --user -u openclaw-gateway -f  # logs
```

### WebChat not loading
```bash
sudo nginx -t  # check config
sudo systemctl status nginx
curl http://127.0.0.1:18789  # test direct access
```

### Model router unreachable
```bash
# From VPS:
curl https://models.yourdomain.com/health
# Check Cloudflare Tunnel status in dashboard
# Check cloudflared service on notebook:
sudo systemctl status cloudflared
```

### Agent routing wrong (Mirai getting Fujiko's messages)
```bash
# Check keyword config in ~/.openclaw/config.json
# Test with: openclaw tui
# Type messages and check which agent responds
```

---

## Links
- [[agents]] (Mirai + Fujiko design)
- [[model-router]] (notebook FastAPI setup)
- [[database]] (SQLite schema)
- [[build-plan]] (setup order)
