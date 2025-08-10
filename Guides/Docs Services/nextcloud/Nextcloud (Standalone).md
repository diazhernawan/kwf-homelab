# Nextcloud (Standalone) on Docker — Katawarna

This guide deploys **Nextcloud only** on `docker-prod06` (Debian 12 VM on Proxmox) with MariaDB, Redis, and a dedicated cron container. It assumes you publish the service through your VPS/edge (Pangolin) and store user data on a NAS (CIFS/SMB).

---

## 0) Preliminaries

1. **Host**: Debian 12 with Docker & Docker Compose installed.
2. **Networks** (already created as external):
   - `frontend` (for edge-facing containers)
   - `backend` (for internal services)
3. **DNS / Edge**: `nextcloud.katawarna.org` points to the VPS/edge that proxies to `docker-prod06:12000`.
4. **NAS mount**: You plan to store data at `/mnt/smb/Helios/nextcloud-data`.
5. **Time zone**: `Asia/Jakarta`.

> **Tip**: Keep the edge proxy responsible for TLS termination and HSTS. We still configure Nextcloud to be reverse‑proxy aware.

---

## Step-by-step (do this in order)

1. **Create external networks (once)**

```bash
docker network create frontend || true
docker network create backend  || true
```

2. **Create directories**

```bash
mkdir -p /home/kwf-admin/docker_volumes/nextcloud/{app,config,db}
sudo mkdir -p /mnt/smb/Helios/nextcloud-data
```

3. **Mount the NAS (CIFS) and verify**

```bash
# creds file
sudo bash -c 'cat >/root/.smb-nextcloud <<EOF
username=YOUR_SMB_USER
password=YOUR_SMB_PASS
EOF'
sudo chmod 600 /root/.smb-nextcloud

# /etc/fstab line (adjust //SERVER/nextcloud-data)
echo "//SERVER/nextcloud-data  /mnt/smb/Helios/nextcloud-data  cifs  credentials=/root/.smb-nextcloud,uid=33,gid=33,file_mode=0660,dir_mode=0770,vers=3.0,cache=none,actimeo=1,noserverino  0  0" | sudo tee -a /etc/fstab

# mount & checks
sudo mount -a
mount | grep Helios
ls -ld /mnt/smb/Helios/nextcloud-data
sudo -u www-data touch /mnt/smb/Helios/nextcloud-data/.write_test && echo "write ok"
```

**Reminder:** do **not** start containers until `mount | grep Helios` shows the CIFS mount with `uid=33,gid=33` and the write test prints `write ok`.

4. **Ownership & permissions (host)**

```bash
cd /home/kwf-admin/docker_volumes/nextcloud
sudo chown -R 33:33 app config
sudo chown -R 999:999 db
sudo chown -R 33:33 /mnt/smb/Helios/nextcloud-data

sudo find app -type d -exec chmod 0750 {} +
sudo find app -type f -exec chmod 0640 {} +
sudo find config -type d -exec chmod 0750 {} +
sudo find config -type f -exec chmod 0640 {} +
sudo find /mnt/smb/Helios/nextcloud-data -type d -exec chmod 2770 {} +
sudo find /mnt/smb/Helios/nextcloud-data -type f -exec chmod 0660 {} +
```

5. **Create `.env`**

```bash
cat > /home/kwf-admin/docker_volumes/nextcloud/.env <<'ENV'
DB_ROOT_PW=change_me_root
DB_NAME=nextcloud
DB_USER=nextcloud_admin
DB_PW=change_me_db
TRUSTED_DOMAINS="nextcloud.katawarna.org onlyoffice.katawarna.org *.katawarna.id *.katawarna.org"
NC_ADMIN_USER=diazhernawan
NC_ADMIN_PASSWORD=strong_admin_password
ENV
chmod 600 /home/kwf-admin/docker_volumes/nextcloud/.env
```

6. **Save `compose.yaml` from this guide** into `/home/kwf-admin/docker_volumes/nextcloud/compose.yaml`.

7. **Bring it up**

```bash
cd /home/kwf-admin/docker_volumes/nextcloud
docker compose pull
docker compose up -d
docker logs -f --tail=100 nextcloud-app
```

8. **Make Nextcloud proxy-aware & enable cron**

```bash
docker exec -u www-data nextcloud-app php occ config:system:set trusted_proxies 0 --value="103.197.189.106"
docker exec -u www-data nextcloud-app php occ config:system:set overwriteprotocol --value="https"
docker exec -u www-data nextcloud-app php occ config:system:set overwrite.cli.url --value="https://nextcloud.katawarna.org"

docker exec -u www-data nextcloud-app php occ config:system:set trusted_domains --type=json --value='["nextcloud.katawarna.org","onlyoffice.katawarna.org","*.katawarna.id","*.katawarna.org"]'

docker exec -u www-data nextcloud-app php occ background:cron
```

9. **Defaults & maintenance window**

```bash
docker exec -u www-data nextcloud-app php occ config:system:set default_phone_region --value="ID"
docker exec -u www-data nextcloud-app php occ config:system:set maintenance_window_start --type=integer --value="19"   # 02:00–06:00 WIB
```

10. **HSTS at edge & verify**

```bash
# after setting HSTS at the proxy…
curl -sI https://nextcloud.katawarna.org | grep -i strict-transport
```

11. **SMTP (Gmail app password)**

```bash
docker exec -u www-data nextcloud-app php occ config:system:set mail_from_address --value="katawarnalab"
docker exec -u www-data nextcloud-app php occ config:system:set mail_domain --value="gmail.com"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpmode --value="smtp"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtphost --value="smtp.gmail.com"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpport --type=integer --value="587"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpsecure --value="tls"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpauth --type=integer --value="1"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpauthtype --value="LOGIN"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpname --value="katawarnalab@gmail.com"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtppassword --value="<APP_PASSWORD>"
```

12. **Health checks & logs**

```bash
curl -sI https://nextcloud.katawarna.org/status.php
docker exec -u www-data nextcloud-app php occ log:tail -f
docker logs -f nextcloud-app
```

## 1) Directory Layout

```text
/home/kwf-admin/docker_volumes/nextcloud/
├── app/           # Nextcloud code (bind mount for easier upgrades)
├── config/        # Nextcloud config.php and app configs
├── db/            # MariaDB datadir
├── compose.yaml   # Docker Compose file
└── .env           # Environment variables

# Data directory (on NAS)
/mnt/smb/Helios/nextcloud-data
```

Create directories:

```bash
mkdir -p \
  /home/kwf-admin/docker_volumes/nextcloud/{app,config,db} \
  /mnt/smb/Helios/nextcloud-data
```

---

## 2) Mount the NAS (CIFS/SMB)

Create credentials file (adjust username/password):

```bash
sudo bash -c 'cat >/root/.smb-nextcloud <<EOF
username=YOUR_SMB_USER
password=YOUR_SMB_PASS
EOF'
sudo chmod 600 /root/.smb-nextcloud
```

Add an `/etc/fstab` line (edit `//SERVER/SHARE` to your NAS path):

```fstab
//SERVER/nextcloud-data  /mnt/smb/Helios/nextcloud-data  cifs  credentials=/root/.smb-nextcloud,uid=33,gid=33,file_mode=0660,dir_mode=0770,vers=3.0,cache=none,actimeo=1,noserverino  0  0
```

Mount & verify:

```bash
sudo mount -a
mount | grep Helios
# Expect to see uid=33,gid=33 and the cifs options above
```

> **Reminder**: If the mount is missing or mis‑owned, Nextcloud will throw errors during install. Always verify the mount **before** `docker compose up`.

---

## 3) Environment Variables (`.env`)

Create `/home/kwf-admin/docker_volumes/nextcloud/.env`:

```dotenv
# ---- Nextcloud DB ----
DB_ROOT_PW=change_me_root
DB_NAME=nextcloud
DB_USER=nextcloud_admin
DB_PW=change_me_db

# ---- Nextcloud app (bootstrap) ----
TRUSTED_DOMAINS="nextcloud.katawarna.org onlyoffice.katawarna.org *.katawarna.id *.katawarna.org"
NC_ADMIN_USER=diazhernawan
NC_ADMIN_PASSWORD=strong_admin_password

# (Optional) Whiteboard/other secrets would go here if used later
```

> After the first successful login, **remove** `NC_ADMIN_USER` and `NC_ADMIN_PASSWORD` from `.env` **and** from the compose `environment:` section (or make them default‑empty) to avoid warnings.

---

## 4) Compose File (`compose.yaml`)

```yaml
services:
  db:
    image: mariadb:10.11
    container_name: nextcloud-db
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED
    environment:
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PW}
      - MYSQL_PASSWORD=${DB_PW}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - TZ=Asia/Jakarta
    volumes:
      - ./db:/var/lib/mysql
    networks:
      - backend

  redis:
    image: redis:alpine
    container_name: nextcloud-redis
    restart: unless-stopped
    environment:
      - TZ=Asia/Jakarta
    networks:
      - backend

  app:
    image: nextcloud:31.0.7
    container_name: nextcloud-app
    restart: unless-stopped
    depends_on:
      - redis
      - db
    ports:
      - "12000:80"
    environment:
      - MYSQL_PASSWORD=${DB_PW}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_HOST=db
      - NEXTCLOUD_TRUSTED_DOMAINS=${TRUSTED_DOMAINS}
      - NEXTCLOUD_DEFAULT_PHONE_REGION=ID
      - NEXTCLOUD_ADMIN_USER=${NC_ADMIN_USER}
      - NEXTCLOUD_ADMIN_PASSWORD=${NC_ADMIN_PASSWORD}
      - REDIS_HOST=redis
      - TZ=Asia/Jakarta
    volumes:
      - ./app:/var/www/html
      - ./config:/var/www/html/config
      - /mnt/smb/Helios/nextcloud-data:/var/www/html/data
    networks:
      - frontend
      - backend

  cron:
    image: nextcloud:31.0.7
    container_name: nextcloud-cron
    restart: unless-stopped
    depends_on:
      - app
    entrypoint: /cron.sh
    environment:
      - TZ=Asia/Jakarta
    volumes:
      - ./app:/var/www/html
      - ./config:/var/www/html/config
      - /mnt/smb/Helios/nextcloud-data:/var/www/html/data
    networks:
      - backend

networks:
  frontend:
    external: true
  backend:
    external: true
```

> Uses **no bracket shorthand** for networks per house style. `app` is on both networks; `db/redis/cron` stay internal (`backend`).

---

## 5) Ownership & Permissions (host)

Set correct ownership **before** starting containers:

```bash
cd /home/kwf-admin/docker_volumes/nextcloud
# www-data typically 33:33, mysql in mariadb:10.11 typically 999:999
sudo chown -R 33:33 app config
sudo chown -R 999:999 db
sudo chown -R 33:33 /mnt/smb/Helios/nextcloud-data

# safe perms
sudo find app -type d -exec chmod 0750 {} \;
sudo find app -type f -exec chmod 0640 {} \;
sudo find config -type d -exec chmod 0750 {} \;
sudo find config -type f -exec chmod 0640 {} \;
# data dir: group‑friendly, keep setgid
sudo find /mnt/smb/Helios/nextcloud-data -type d -exec chmod 2770 {} \;
sudo find /mnt/smb/Helios/nextcloud-data -type f -exec chmod 0660 {} \;
```

---

## 6) Bring it up

```bash
cd /home/kwf-admin/docker_volumes/nextcloud
docker compose pull
docker compose up -d

# quick sniff
docker logs -f --tail=100 nextcloud-app
```

Access `https://nextcloud.katawarna.org` through the edge proxy. The admin account is created from the `.env` values at first boot.

> After first login: remove `NC_ADMIN_USER` and `NC_ADMIN_PASSWORD` from `.env` and from `app.environment:` in `compose.yaml` (or change to `NC_ADMIN_USER=${NC_ADMIN_USER:-}` to silence warnings). Set `.env` to `chmod 600`.

---

## 7) Post‑install (one‑time OCC)

Run these to make Nextcloud reverse‑proxy aware, enable caches/locking, and set basics.

```bash
# Reverse proxy awareness (edge IP)
docker exec -u www-data nextcloud-app php occ config:system:set trusted_proxies 0 --value="103.197.189.106"
docker exec -u www-data nextcloud-app php occ config:system:set overwriteprotocol --value="https"
docker exec -u www-data nextcloud-app php occ config:system:set overwrite.cli.url --value="https://nextcloud.katawarna.org"

# Trusted domains (wildcards supported)
docker exec -u www-data nextcloud-app php occ config:system:set trusted_domains --type=json \
  --value='["nextcloud.katawarna.org","onlyoffice.katawarna.org","*.katawarna.id","*.katawarna.org"]'

# Caching & file locking
docker exec -u www-data nextcloud-app php occ config:system:set memcache.local --value='\OC\Memcache\APCu'
docker exec -u www-data nextcloud-app php occ config:system:set memcache.distributed --value='\OC\Memcache\Redis'
docker exec -u www-data nextcloud-app php occ config:system:set memcache.locking --value='\OC\Memcache\Redis'
docker exec -u www-data nextcloud-app php occ config:system:set redis host --value="redis"
docker exec -u www-data nextcloud-app php occ config:system:set redis port --type=integer --value="6379"
docker exec -u www-data nextcloud-app php occ config:system:set filelocking.enabled --type=boolean --value=true

# Background jobs → cron
docker exec -u www-data nextcloud-app php occ background:cron

# Defaults & hygiene
docker exec -u www-data nextcloud-app php occ config:system:set default_phone_region --value="ID"
docker exec -u www-data nextcloud-app php occ config:system:set maintenance_window_start --type=integer --value="19"  # 02:00–06:00 WIB

# (Optional) Set pretty name for emails/UI
docker exec -u www-data nextcloud-app php occ config:app:set theming name --value="Nextcloud Katawarna"
```

---

## 8) HSTS (Strict‑Transport‑Security)

Set HSTS at the **edge proxy** for `nextcloud.katawarna.org` (recommended), e.g.:

- **Nginx**
  ```nginx
  add_header Strict-Transport-Security "max-age=15552000; includeSubDomains; preload" always;
  ```
- **Caddy**
  ```caddy
  header Strict-Transport-Security "max-age=15552000; includeSubDomains; preload"
  ```
- **Traefik (file)**
  ```yaml
  headers:
    stsSeconds: 15552000
    stsIncludeSubdomains: true
    stsPreload: true
    forceSTSHeader: true
  ```

Verify from the VPS:

```bash
curl -sI https://nextcloud.katawarna.org | grep -i strict-transport
```

> If you can’t change the edge right away, you can inject HSTS inside the NC container (Apache) using a `headers` conf; edge is still preferred.

---

## 9) SMTP (Gmail example with App Password)

1. Enable **2‑Step Verification** for the Gmail account, create an **App password**.
2. Configure in Nextcloud:

```bash
# From address (example: katawarnalab@gmail.com)
docker exec -u www-data nextcloud-app php occ config:system:set mail_from_address --value="katawarnalab"
docker exec -u www-data nextcloud-app php occ config:system:set mail_domain --value="gmail.com"

# Option A: STARTTLS (587)
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpmode --value="smtp"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtphost --value="smtp.gmail.com"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpport --type=integer --value="587"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpsecure --value="tls"

# Auth
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpauth --type=integer --value="1"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpauthtype --value="LOGIN"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtpname --value="katawarnalab@gmail.com"
docker exec -u www-data nextcloud-app php occ config:system:set mail_smtppassword --value="YOUR_APP_PASSWORD"
```

Test via UI (Basic settings → **Send email**). If CLI test is needed:

```bash
docker exec -u www-data nextcloud-app php -r 'define("OC_CONSOLE",1); require "/var/www/html/lib/base.php"; $m=\OC::$server->getMailer(); $msg=$m->createMessage(); $msg->setSubject("Nextcloud SMTP test"); $msg->setTo(["you@example.com"=>"Test"]); $msg->setPlainBody("SMTP OK ".date("c")); $m->send($msg); echo "Sent\n";'
```

---

## 10) Health checks & Logs

```bash
# App reachable via edge
curl -sI https://nextcloud.katawarna.org/status.php

# Watch Nextcloud app logs
docker exec -u www-data nextcloud-app php occ log:tail -f

# Container logs
docker logs -f nextcloud-app
```

If you see `"no app in context – strict cookie check"`, point any monitoring to `/status.php` rather than `/`.

---

## 11) Backup (quick pattern)

```bash
# DB dump
mysqldump -h 127.0.0.1 -P 3306 -u${DB_USER} -p${DB_PW} ${DB_NAME} > /backup/nextcloud-$(date +%F).sql

# App/config (bind mounts)
rsync -a --delete /home/kwf-admin/docker_volumes/nextcloud/{app,config} /backup/

# Data (NAS): snapshot/replicate on the NAS side
```

Put Nextcloud into maintenance mode for consistent backups if needed:

```bash
docker exec -u www-data nextcloud-app php occ maintenance:mode --on
# ...backup...
docker exec -u www-data nextcloud-app php occ maintenance:mode --off
```

---

## 12) Upgrades

```bash
cd /home/kwf-admin/docker_volumes/nextcloud
docker compose pull
docker compose up -d
```

If the image upgrades major versions, skim the Nextcloud release notes and run:

```bash
docker exec -u www-data nextcloud-app php occ maintenance:repair --include-expensive
```

---

## 13) Integrate with Existing Stack (keep Nextcloud & ONLYOFFICE in separate directories)

The linked stack defines shared external networks. We’ll attach **both** projects to the same `frontend` and `backend` networks so they can resolve each other by container name while remaining in separate folders.

### A) Networks (one‑time, if not already present)

```bash
# Create external user-defined networks if your main stack hasn't created them yet
docker network create frontend || true
docker network create backend  || true
```

### B) ONLYOFFICE in its own folder

Directory: `/home/kwf-admin/docker_volumes/onlyoffice/`

```
/home/kwf-admin/docker_volumes/onlyoffice/
├── .env
├── compose.yaml
├── data/
├── db/
├── lib/
└── logs/
```

**`onlyoffice/.env`** (example)

```dotenv
# JWT for NC↔OO
ONLYOFFICE_JWT=change_me_super_secret
TZ=Asia/Jakarta
```

**`onlyoffice/compose.yaml`**

```yaml
services:
  onlyoffice:
    image: onlyoffice/documentserver:latest
    container_name: onlyoffice
    restart: unless-stopped
    ports:
      - "12400:80"   # Pangolin/edge will publish this as https://office.katawarna.org
    environment:
      - JWT_ENABLED=true
      - JWT_SECRET=${ONLYOFFICE_JWT}
      - JWT_HEADER=Authorization
      - TZ=Asia/Jakarta
    volumes:
      - ./data:/var/www/onlyoffice/Data
      - ./logs:/var/log/onlyoffice
      - ./lib:/var/lib/onlyoffice
      - ./db:/var/lib/postgresql
    entrypoint:
      - /bin/bash
      - -lc
      - |
        echo "Forcing https scheme inside ONLYOFFICE Nginx..."
        sed -Ei 's@proxy_set_header[[:space:]]+X-Forwarded-Proto[[:space:]]+.*;@proxy_set_header X-Forwarded-Proto https;@' /etc/onlyoffice/documentserver/nginx/includes/http-common.conf
        echo "Resulting X-Forwarded-Proto line:"
        grep -n 'X-Forwarded-Proto' /etc/onlyoffice/documentserver/nginx/includes/http-common.conf
        exec /app/ds/run-document-server.sh
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://localhost/healthcheck"]
      interval: 30s
      timeout: 3s
      retries: 5
    networks:
      - frontend
      - backend

networks:
  frontend:
    external: true
  backend:
    external: true
```

> **Proto/HSTS behind reverse proxy**: If previews or downloads misbehave (mixed content or websocket redirects), force `X-Forwarded-Proto https` inside the DocumentServer’s nginx:
>
> ```bash
> docker exec onlyoffice bash -lc "sed -Ei 's@proxy_set_header[[:space:]]\+X-Forwarded-Proto[[:space:]]\+.*;@proxy_set_header X-Forwarded-Proto https;@' /etc/onlyoffice/documentserver/nginx/includes/http-common.conf && supervisorctl restart all"
> ```

### C) Point Nextcloud to ONLYOFFICE

After both projects are up and on the same `frontend` and `backend` networks:

> **Reminder:** In the Nextcloud GUI (Admin → ONLYOFFICE), set **ONLYOFFICE Docs address** to `https://office.katawarna.org/` and paste the **Secret key** you used above, then click **Save** to run the built‑in connectivity test.
>
> **If you use the internal StorageUrl (`http://nextcloud-app/`):** Add `nextcloud-app` to **trusted_domains**, otherwise Document Server → Nextcloud calls may 400.
>
> ```bash
> docker exec -u www-data nextcloud-app php occ config:system:set trusted_domains \
>   --type=json \
>   --value='["nextcloud.katawarna.org","onlyoffice.katawarna.org","*.katawarna.id","*.katawarna.org","nextcloud-app"]'
> ```
>
> **Or** set StorageUrl to the public URL instead (less efficient but simpler):
>
> ```bash
> docker exec -u www-data nextcloud-app php occ config:app:set onlyoffice StorageUrl --value "https://nextcloud.katawarna.org/"
> ```

```bash
# Public URL (through edge)
PUBLIC_DS_URL=https://office.katawarna.org
# Internal URL (Docker-to-Docker)
INTERNAL_DS_URL=http://onlyoffice

# Configure via OCC (requires the ONLYOFFICE app enabled in Nextcloud)
docker exec -u www-data nextcloud-app php occ config:app:set onlyoffice DocumentServerUrl --value "${PUBLIC_DS_URL}/"
docker exec -u www-data nextcloud-app php occ config:app:set onlyoffice DocumentServerInternalUrl --value "${INTERNAL_DS_URL}/"
docker exec -u www-data nextcloud-app php occ config:app:set onlyoffice StorageUrl --value "http://nextcloud-app/"

# JWT shared secret
docker exec -u www-data nextcloud-app php occ config:app:set onlyoffice jwt_secret --value "${ONLYOFFICE_JWT}"
docker exec -u www-data nextcloud-app php occ config:app:set onlyoffice jwt_header  --value "Authorization"
```

> **Notes**
>
> - `nextcloud-app` is reachable from ONLYOFFICE because both join the **backend** network and `nextcloud-app` uses a fixed `container_name`.
> - Keep Nextcloud and ONLYOFFICE compose files and volumes in **separate directories** as shown.
> - Edge routing: map `nextcloud.katawarna.org → 10.0.1.56:12000` and `office.katawarna.org → 10.0.1.56:12400` in Pangolin.

### D) Start order

You can bring them up independently:

```bash
# Nextcloud
cd /home/kwf-admin/docker_volumes/nextcloud
docker compose up -d

# ONLYOFFICE
cd /home/kwf-admin/docker_volumes/onlyoffice
docker compose up -d
```

### E) Health checks

### F) Quick smoke tests

```bash
# 1) From Nextcloud → Document Server (internal & public)
docker exec nextcloud-app bash -lc 'curl -sI http://onlyoffice/healthcheck | head -n1'
docker exec nextcloud-app bash -lc 'curl -sI https://office.katawarna.org/web-apps/apps/api/documents/api.js | head -n1'

# 2) From Document Server → Nextcloud (internal StorageUrl)
docker exec onlyoffice bash -lc 'curl -sI http://nextcloud-app/status.php | head -n1'

# 3) Verify JWT seen by both sides
docker exec onlyoffice bash -lc 'echo "OO JWT=$JWT_SECRET"'
docker exec -u www-data nextcloud-app php occ config:app:get onlyoffice jwt_secret
```

### G) Clear/rotate Nextcloud logs

```bash
# Clear (truncate) the current log
docker exec nextcloud-app bash -lc 'truncate -s 0 /var/www/html/data/nextcloud.log && echo "Nextcloud log cleared"'

# Optional: set log level to WARNING and rotate at 100 MB
docker exec -u www-data nextcloud-app php occ config:system:set loglevel --type=integer --value=2
docker exec -u www-data nextcloud-app php occ config:system:set log_rotate_size --type=integer --value=104857600

# Watch fresh logs while testing
docker exec -u www-data nextcloud-app php occ log:tail -f
```

---

## 14) Single Sign‑On with authentik (OIDC)

> This uses Nextcloud’s **OpenID Connect Login** app with authentik as the identity provider. Recommended over SAML for simplicity and modern flows.

### A) authentik — Create Application & OIDC Provider

1. **Base URL:** `AUTHENTIK_BASE_URL=https://authentik.katawarna.id`
2. **Applications → Create with Provider** (type: **OAuth2/OpenID Connect**):
   - Application name: `Nextcloud`
   - Note the **application slug** (used in the discovery URL below).
   - Provider name: `nextcloud`
   - Authorization flow: **Authorization Code**
   - **Redirect URI (strict):** `https://nextcloud.katawarna.org/apps/user_oidc/code`
   - Signing key: pick any available
   - **Advanced protocol settings:**
     - Subject mode: **Based on the User's UUID**
     - (Optional) If you created a custom scope mapping, add scope `nextcloud` to Selected scopes
   - Save and record **Client ID**, **Client Secret**, and **Application slug**.

### B) Nextcloud — Install & configure OIDC app

1. **Install app**: Apps → Security → **OpenID Connect Login** (or via CLI)
   ```bash
   docker exec -u www-data nextcloud-app php occ app:install oidc_login || true
   docker exec -u www-data nextcloud-app php occ app:enable oidc_login
   ```
2. **Admin → Security → OpenID Connect** (UI):
   - **Discovery endpoint:** `https://authentik.katawarna.id/application/o/<application_slug>/.well-known/openid-configuration`  *(replace with your slug)*
   - **Client ID / Secret:** from authentik provider
   - **Scopes:** `openid profile email` *(+ `nextcloud` if you created the custom scope mapping)*
   - **Custom end session endpoint (optional):** `https://authentik.katawarna.id/application/o/end-session/`
   - **Login button text:** `Login with Katawarna SSO`
   - **Auto-create users:** ✅
   - **Attribute mapping:**
     - **User ID mapping:** `sub`  *(or `user_id` if you’re linking existing users via a custom claim)*
     - **Display name mapping:** `name`
     - **Email mapping:** `email`
     - **Quota mapping (optional):** `quota`
     - **Groups mapping (optional):** `groups` *(enable group provisioning if you want NC to create groups)*
   - **Use unique user ID:** enable unless you intentionally want to use a custom string as the Federated Cloud ID.
   - Save.

> **Note:** If your authentik publishes groups, you may want to enable the NC feature “Only allow group‑listed users to use OIDC” after you’ve confirmed group sync.

### C) Test the login flow

1. Open a private/incognito window → `https://nextcloud.katawarna.org` → click **Login with Katawarna SSO**.
2. Approve the consent screen in authentik → you should land back in NC as a new user.
3. Confirm the new user’s email/display name and group assignment.

**CLI sanity:**

```bash
# Check the OIDC app is enabled
docker exec -u www-data nextcloud-app php occ app:list | sed -n '/Enabled:/,/Disabled:/p' | grep oidc_login || true
# Confirm trusted_proxies/overwrite settings already present in this guide
```

### D) Keep a break‑glass admin

- Keep one **local** NC admin (password login) in case IdP is down: `diazhernawan` is fine.
- If you ever get locked out, disable OIDC via CLI:
  ```bash
  docker exec -u www-data nextcloud-app php occ app:disable oidc_login
  ```

### E) Optional hardening & UX

- Enforce SSO first: set the OIDC provider as default. Keep a link for direct login when needed: `https://nextcloud.katawarna.org/login?direct=1`.
- Map avatar to the IdP `picture` claim (if provided) in the OIDC app’s advanced options.

### F) Troubleshooting

- **Redirect URI mismatch** → ensure it’s exactly: `https://nextcloud.katawarna.org/apps/user_oidc/code` in authentik.
- **Issuer/metadata error** → check Provider URL ends with `/application/o/<slug>/.well-known/openid-configuration` and is reachable from NC container.
  ```bash
  docker exec nextcloud-app curl -sI https://authentik.katawarna.id/application/o/<slug>/.well-known/openid-configuration | head -n1
  ```
- **Access forbidden – Failed to provision the user** → usually one of:
  1. **Auto‑create users disabled**: Edit your OIDC provider in NC and enable **Auto‑create users** (or similar wording).
  2. **Username not valid**: Set **User ID mapping** to `sub` and tick **Use unique user ID**; or use a sanitized claim like `preferred_username` that matches NC username rules.
  3. **Missing claims**: Make sure the provider issues `email` and `name` by including scopes `openid profile email`. In authentik, ensure the default **profile** and **email** scope mappings are attached to the provider.
  4. **Groups/quota mappings** (optional) require the claim to actually exist; remove them for first login.
- **Groups not showing** → add a `groups` claim in authentik (Blueprint/Policy), include it in the provider, and list `groups` in NC’s “Groups claim”.

---

### Done

Your Nextcloud is integrated with ONLYOFFICE and now supports **SSO via authentik (OIDC)**. Separate stacks, shared networks, clean JWT handshakes, and a modern login flow. Ship it! 🚀
