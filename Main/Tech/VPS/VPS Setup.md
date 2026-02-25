

### Phase 1: OS Hardening & Cockpit Installation

This phase prepares the host and installs the foundational management tools.

Bash

```
# 1. Update system and install Firewalld & Cockpit
sudo apt update && sudo apt upgrade -y
sudo apt install firewalld cockpit -y

# 2. Enable services
sudo systemctl enable --now firewalld
sudo systemctl enable --now cockpit.socket

# 3. Configure Firewall (Open Web/VPN, Close Public Management)
sudo firewall-cmd --permanent --add-port={80/tcp,443/tcp,81/tcp} # NPM & Admin UI
sudo firewall-cmd --permanent --add-port=51820/udp             # WireGuard Tunnel
sudo firewall-cmd --permanent --remove-service=cockpit         # Remove public 9090
sudo firewall-cmd --permanent --zone=docker --add-interface=docker0
sudo firewall-cmd --permanent --zone=docker --add-masquerade

# 4. Allow Docker Bridge to talk to Host (For NPM -> Apps)
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="172.17.0.0/16" accept'
sudo firewall-cmd --reload

# 5. Configure Cockpit to trust the Admin Domain
sudo bash -c 'cat <<EOF > /etc/cockpit/cockpit.conf
[WebService]
Origins = https://admin.doubt.dev wss://admin.doubt.dev
ProtocolHeader = X-Forwarded-Proto
AllowUnencrypted = true
EOF'
sudo systemctl restart cockpit
```

---
#### Testing Cockpit

1. Open your browser to: `https://your-vps-ip:9090` (Note: It uses **https**).
    
2. Log in with your **Debian system username and password**.
   
### Phase 2: Docker & Dockge Installation

We install the container engine and the manager UI first.

Docker

```
# Update system and install prerequisites
sudo apt update && sudo apt upgrade -y
sudo apt install ca-certificates curl gnupg lsb-release -y

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine and Compose plugin
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-
```

Dockge

```
# 1. Install Dockge (The Manager)
sudo mkdir -p /opt/stacks /opt/dockge
cd /opt/dockge
sudo wget https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml

# 2. Secure Dockge Port (Bind to Internal Gateway only)
sudo sed -i 's/"5001:5001"/"172.17.0.1:5001:5001"/g' compose.yaml

sudo docker compose up -d
```

You can now access Dockge at `http://your-vps-ip:5001`

### Phase 3: The "Management" Master-Stack

Generate bcrypt hash for the password you want to use for Wireguard admin login.
Replace the placeholder in YAML below with the generated password hash remembering to use double dollar for single dollar because Docker interprets dollar differently.

`docker run --rm alpine sh -c "apk add --no-cache apache2-utils > /dev/null && htpasswd -nbB 12 '' your_password | cut -d : -f 2"`

Deploy this stack inside the **Dockge UI**. It consolidates Nginx Proxy Manager, AdGuard, and WireGuard into one controlled environment.

**In Dockge UI, Name the stack as "management" or whatever or else it will use the name of the first service(npm) as stack name which can be confusing**

YAML

```
services:
  npm:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: always
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./npm/data:/data
      - ./npm/letsencrypt:/etc/letsencrypt
      - /var/run/docker.sock:/var/run/docker.sock

  adguard:
    image: adguard/adguardhome
    restart: always
    ports:
      - '172.17.0.1:3000:3000' # Initial setup only
      - '172.17.0.1:8080:80'   # Dashboard
      - '53:53/tcp'
      - '53:53/udp'
    volumes:
      - ./adguard/work:/opt/adguardhome/work
      - ./adguard/conf:/opt/adguardhome/conf

  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    restart: always
    environment:
      - WG_HOST=YOUR_VPS_PUBLIC_IP
      - PASSWORD_HASH=$$2a$$12$$your_bcrypt_hash_here # Replace this! 
    ports:
      - "51820:51820/udp"        # Public Tunnel
      - "172.17.0.1:51821:51821" # Private UI
    volumes:
      - ./wireguard:/etc/wireguard
    cap_add: [NET_ADMIN, SYS_MODULE]
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
```


### Phase 4: Cloudflare & NPM Mapping

Finally, route your traffic through the "Cloudflare Shield" to your NPM gateway.

#### 1. Cloudflare DNS

Set all subdomains to **Proxied (Orange Cloud)** and SSL/TLS to **Full (Strict)**.

#### 2. NPM Proxy Hosts

| **Subdomain**         | **Forward Scheme** | **Forward IP** | **Port** | **WebSockets** |
| --------------------- | ------------------ | -------------- | -------- | -------------- |
| `manage.doubt.dev`    | HTTP               | `172.17.0.1`   | `5001`   | **ON**         |
| `vpn.doubt.dev`       | HTTP               | `172.17.0.1`   | `51821`  | **ON**         |
| `dns.doubt.dev`       | HTTP               | `172.17.0.1`   | `8080`   | **ON**         |
| **`admin.doubt.dev`** | **HTTPS**          | `172.17.0.1`   | `9090`   | **ON**         |

### Phase 5: Verification

1. **Direct IP:** `http://<vps-ip>:9090` must **Fail**.
    
2. **Domain:** `https://admin.doubt.dev` must **Work**.
    
3. **Terminal:** Open the terminal in Cockpit (`admin.doubt.dev`). It must be **Stable**.