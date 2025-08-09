# Nextcloud (Standalone) on Docker — Katawarna

This guide deploys **Nextcloud only** on `docker-prod06` (Debian 12 VM on Proxmox) with MariaDB, Redis, and a dedicated cron container. It assumes you publish the service through your VPS/edge (Pangolin) and store user data on a NAS (CIFS/SMB).

---

## 0) Preliminaries

1. **Host**: Debian 12 with Docker & Docker Compose installed.
2. **Networks** (already created as external):

   * `frontend` (for edge-facing containers)
   * `backend` (for internal services)
3. **DNS / Edge**: `nextcloud.katawarna.org` points to the VPS/edge that proxies to `docker-prod06:12000`.
4. **NAS mount**: You plan to store data at `/mnt/smb/Helios/nextcloud-data`.
5. **Time zone**: `Asia/Jakarta`.

> **Tip**: Keep the edge proxy responsible for TLS termination and HSTS. We still configure Nextcloud to be reverse‑proxy aware.

---

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

* **Nginx**

  ```nginx
  add_header Strict-Transport-Security "max-age=15552000; includeSubDomains; preload" always;
  ```
* **Caddy**

  ```caddy
  header Strict-Transport-Security "max-age=15552000; includeSubDomains; preload"
  ```
* **Traefik (file)**

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

### Done

Your Nextcloud‑only stack is production‑ready: MariaDB + Redis + Cron, proxy‑aware, with NAS storage and edge TLS/HSTS. Ship it! 🚀
