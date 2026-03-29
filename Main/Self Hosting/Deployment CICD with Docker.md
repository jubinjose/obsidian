#cicd #github-actions
### On VPS as jubin

bash

````bash
# Create cicd user
sudo adduser cicd
sudo usermod -aG docker cicd

# Create stack folder and set ownership
sudo mkdir -p /opt/stacks/helper
sudo chown -R cicd:cicd /opt/stacks

# Create docker-compose.yml
cat > /opt/stacks/helper/docker-compose.yml << 'EOF'
services:
  helper:
    image: registry.scrambleid.com/helper:latest
    restart: unless-stopped
    ports:
      - "7001:7001"
EOF

# Login to registry as cicd (one-time)
sudo -u cicd docker login registry.scrambleid.com

# Update Caddyfile
sudo nano /etc/caddy/Caddyfile

# Validate and restart Caddy
sudo caddy fmt --overwrite /etc/caddy/Caddyfile
caddy validate --config /etc/caddy/Caddyfile
sudo systemctl restart caddy
```

### Final Caddyfile
```
apps.scrambleid.com {
    basicauth {
        jubin <PASSWORD-HASH>
    }

    handle /helper* {
        uri strip_prefix /helper
        reverse_proxy localhost:7001
    }

    handle {
        reverse_proxy localhost:5001
    }
}

registry.scrambleid.com {
    basicauth {
        jubin <PASSWORD-HASH>
    }
    reverse_proxy localhost:5000
}
````
Note: PASSWORD-HASH is the basic auth password hash generated for caddy during setup
### On Local Windows Machine (PowerShell)

powershell

```powershell
# Generate cicd SSH key (no passphrase)
ssh-keygen -t ed25519 -C "cicd@scrambleid" -f $env:USERPROFILE\.ssh\id_ed25519_cicd -N ""

# View public key to copy to VPS
cat $env:USERPROFILE\.ssh\id_ed25519_cicd.pub

# View private key to copy to GitHub secret
Get-Content $env:USERPROFILE\.ssh\id_ed25519_cicd -Raw
```

### On VPS — Add cicd public key

bash

````bash
# As jubin
sudo mkdir -p /home/cicd/.ssh
echo "paste-id_ed25519_cicd.pub-contents-here" | sudo tee /home/cicd/.ssh/authorized_keys
sudo chown -R cicd:cicd /home/cicd/.ssh
sudo chmod 700 /home/cicd/.ssh
sudo chmod 600 /home/cicd/.ssh/authorized_keys
```

### GitHub Repo Settings

**Secrets** (Settings → Secrets and variables → Actions → Secrets):
| Secret | Value |
|---|---|
| `CICD_SSH_PRIVATE_KEY` | Contents of `id_ed25519_cicd` private key |
| `REGISTRY_PASSWORD` | Registry basicauth password |
| `REGISTRY_USERNAME` | `jubin` |
| `SSH_PRIVATE_KEY` | Existing key for old PM2 workflow |

**Variables** (Settings → Secrets and variables → Actions → Variables):
| Variable | Value |
|---|---|
| `CICD_SSH_HOST` | `107.175.91.41` |
| `CICD_SSH_USERNAME` | `cicd` |
| `SSH_HOST` | Existing host for old PM2 workflow |
| `SSH_USERNAME` | Existing username for old PM2 workflow |

### Repo Files Added
```
helper/
  Dockerfile
  docker-compose.yml        ← not needed in repo, lives on VPS
  .github/
    workflows/
      deploy-docker.yml
````
### Dockerfile

dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --omit=dev

COPY src/ ./src/
COPY config/ ./config/
COPY VERSION ./

EXPOSE 7001

ENV APP_ENV=vps

CMD ["node", "./src/app.js"]
```

### deploy-docker.yml

yaml

```yaml
name: Docker CD

on:
  workflow_dispatch:
    inputs:
      reason:
        description: 'Reason for manual trigger'
        required: true
        default: 'Manual trigger via UI'

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Log manual trigger reason
        if: github.event_name == 'workflow_dispatch'
        env:
          REASON: ${{ github.event.inputs.reason }}
        run: |
          echo "Manual trigger reason: $REASON"

      - name: Login to registry
        run: |
          echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login registry.scrambleid.com \
            -u "${{ secrets.REGISTRY_USERNAME }}" \
            --password-stdin

      - name: Build and push image
        run: |
          docker build -t registry.scrambleid.com/helper:latest .
          docker push registry.scrambleid.com/helper:latest

      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1.2.5
        with:
          host: ${{ vars.CICD_SSH_HOST }}
          username: ${{ vars.CICD_SSH_USERNAME }}
          key: ${{ secrets.CICD_SSH_PRIVATE_KEY }}
          port: 22
          script: |
            set -e
            cd /opt/stacks/helper
            docker compose pull
            docker compose up -d
            docker image prune -f
```