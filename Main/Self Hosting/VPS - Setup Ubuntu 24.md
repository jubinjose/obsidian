# VPS Setup Runbook — Ubuntu 24.04

#docker #ubuntu #vps #caddy 
## Stack: Caddy + Docker + Dockge + DockerRegistry

---
## Step 1 — Update & Reboot
```bash
apt update && apt upgrade -y && apt autoremove -y
reboot
```

---

## Step 2 — Firewall & Fail2ban
```bash
apt install -y fail2ban
systemctl enable fail2ban
systemctl start fail2ban

ufw allow OpenSSH
ufw allow 80
ufw allow 443
ufw enable
ufw status
```

---

## Step 3 — Create Non-Root User
```bash
adduser jubin
usermod -aG sudo jubin
```

---

## Step 4 — SSH Key Setup

### On local Windows machine (PowerShell)
```powershell
ssh-keygen -t ed25519 -C "jubin@scrambleid"
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh jubin@your-vps-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### Add SSH config for easy access (Optional)
Create `C:\Users\jubin\.ssh\config`:
```
Host vps
    HostName your-vps-ip
    User jubin
    IdentityFile ~/.ssh/id_ed25519
```

### Test login in new terminal (Optional)
```powershell
ssh vps
```

---

## Step 5 — Harden SSH
```bash
sudo nano /etc/ssh/sshd_config
```

Set:
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Port 22
```

Also check and update override file:
```bash
sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf
# Set: PasswordAuthentication no
```
```bash
sudo systemctl restart ssh
```

**Verify login in new terminal using KEY NOT PASWORD before closing current session.**

---

## Step 6 — Install Docker
```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker jubin
```

Log out and back in for group membership to take effect.
```bash
docker ps  # verify
```

---

## Step 7 — Install Caddy
```bash
sudo apt install -y caddy
sudo systemctl enable caddy
sudo systemctl status caddy
```

---

## Step 8 — Directory Structure
```bash
sudo mkdir -p /opt/dockge /opt/stacks /opt/registry
sudo chown -R jubin:jubin /opt/stacks /opt/registry
```

---

## Step 9 — Deploy Dockge
```bash
cd /opt/dockge
sudo curl -o compose.yaml https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml
docker compose up -d
```

Temporarily open port to verify:
```bash
sudo ufw allow 5001
```

Verify at `http://your-vps-ip:5001`

---

## Step 10 — Deploy Self-Hosted Registry
```bash
cat > /opt/registry/compose.yaml << 'EOF'
services:
  registry:
    image: registry:2
    restart: unless-stopped
    ports:
      - "5000:5000"
    volumes:
      - registry_data:/var/lib/registry
volumes:
  registry_data:
EOF

cd /opt/registry && docker compose up -d
```

Temporarily open port to verify:
```bash
sudo ufw allow 5000
```

---

## Step 11 — DNS Records

Create A records at your DNS provider:
| Subdomain | IP |
|---|---|
| `apps.yourdomain.com` | `your-vps-ip` |
| `registry.yourdomain.com` | `your-vps-ip` |

---

## Step 12 — Configure Caddy

Generate basic auth password hash:
```bash
caddy hash-password --plaintext 'yourpassword'
```

Edit Caddyfile:
```bash
sudo nano /etc/caddy/Caddyfile
```
```
apps.yourdomain.com {
    basicauth {
        yourusername <paste-hash-here>
    }
    reverse_proxy localhost:5001
}

registry.yourdomain.com {
    basicauth {
        yourusername <paste-hash-here>
    }
    reverse_proxy localhost:5000
}
```

Validate and restart:
```bash
sudo caddy fmt --overwrite /etc/caddy/Caddyfile
caddy validate --config /etc/caddy/Caddyfile
sudo systemctl restart caddy
sudo systemctl status caddy
```

Verify dockge & registry:
```bash

Open https://registry.yourdomain.com in browser
# Basic Auth prompt and then Dockge initial screen

curl -u username:password https://registry.yourdomain.com/v2/
# Expected response: {}
```

---

## Step 13 — Close Direct Ports

Once everything is behind Caddy:
```bash
sudo ufw delete allow 5001
sudo ufw delete allow 5000
sudo ufw status
```

---

## Final State

| Component | URL |
|---|---|
| Dockge UI | `https://apps.yourdomain.com` |
| Docker Registry | `https://registry.yourdomain.com` |

| Service | Status |
|---|---|
| UFW Firewall | ✅ 80, 443, SSH only |
| Fail2ban | ✅ |
| Docker | ✅ |
| Caddy + Auto TLS | ✅ |
| Dockge | ✅ Behind Caddy + basicauth |
| Self-hosted Registry | ✅ Behind Caddy + basicauth |