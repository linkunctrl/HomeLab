# Vaultwarden Homelab Setup Documentation

## Initial Deployment
```bash
docker pull vaultwarden/server:latest
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
```

**Environment Details:**
* **Docker version:** 29.3.0, build 5927d80

---

## Troubleshooting Log

### Problem 1: HTTP 500 Internal Server Error
**Cause:**
Used the wrong volume path in the initial Docker run command. Mounting `/vw-data/` creates the directory at the root of the filesystem, which requires root privileges to write to. Since the Vaultwarden container runs as user 1000 (not root), it couldn't write its database and config files to that directory.

**Fix:**
Switching the volume path to `~/vw-data/` places the directory in the home folder which the current user already owns.

```bash
sudo mkdir -p /vw-data
sudo chown -R 1000:1000 /vw-data
sudo chmod 755 /vw-data
 
docker restart vaultwarden
```

---

### Problem 2: Insecure URL (HTTPS Required)
**Cause:**
Vaultwarden requires HTTPS for secure credential handling and browser compatibility.

**Fix:**
Implemented Caddy with the DuckDNS plugin for DNS-01 challenges.

**1. Build Custom Caddy:**
```bash
sudo pacman -S go
yay -S xcaddy

# Verify Go installation
go version

# Build Caddy with DuckDNS support
xcaddy build --with github.com/caddy-dns/duckdns

# Move build to working directory
mv caddy ~/vaultwarden/
chmod +x ~/vaultwarden/caddy
```

**2. DNS Configuration:**
* Obtain free DNS name and token at `duckdns.org`.
* Update IP from public to the system's private IP.

**3. Docker Compose Setup (`docker-compose.yml`):**
```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      WEBSOCKET_ENABLED: "true" 
    volumes:
      - ./vw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy:/usr/bin/caddy  
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      DOMAIN: "https://vaultwarden.example.com" 
      EMAIL: "y/n@example.com"                
      DUCKDNS_TOKEN: "(the token from earlier goes here)"                  
      LOG_FILE: "/data/access.log"
```

**4. Caddyfile Configuration:**
```caddy
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  tls {
    dns duckdns {$DUCKDNS_TOKEN}
  }
  encode gzip
  reverse_proxy /notifications/hub vaultwarden:3012
  reverse_proxy vaultwarden:80
}
```

---

### Problem 3: Port 80 Binding Conflict
**Cause:**
Error: `bind: address already in use`. Multiple processes or legacy containers were attempting to occupy port 80.

**Fix:**
Identified and cleared conflicting processes and reset the network state.

**1. Identify Conflict:**
```bash
sudo ss -tulpn | grep :80
```

**2. Cleanup:**
```bash
docker compose down
docker stop vaultwarden && docker rm vaultwarden
sudo kill <PID> # For remaining processes on port 80
docker network prune -f
```

**3. Configuration Updates:**
* Removed the `version` line from `docker-compose.yml`.
* Updated `Caddyfile` with explicit resolvers:

```caddy
{$DOMAIN}:443 {
  log {
    level INFO
  }

  tls {
    dns duckdns {$DUCKDNS_TOKEN}
    resolvers 1.1.1.1 8.8.8.8
  }

  encode gzip

  reverse_proxy /notifications/hub vaultwarden:3012
  reverse_proxy vaultwarden:80
}
```

**4. Final Re-deployment:**
```bash
# Clear old ACME certs to force fresh challenge
sudo rm -rf ~/vaultwarden/caddy-data/caddy/acme

# Start services
docker compose up -d

# Verify status
sudo docker ps
```
