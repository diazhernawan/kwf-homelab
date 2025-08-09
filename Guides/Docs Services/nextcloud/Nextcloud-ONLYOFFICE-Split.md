# Nextcloud + ONLYOFFICE (split stacks) — Docker Compose

This guide splits your deployment into **two separate Compose projects**: one for **Nextcloud** and one for **ONLYOFFICE Document Server**.  
It includes **persistent volumes** for ONLYOFFICE and keeps the proven **`sed` hack** that forces HTTPS inside the ONLYOFFICE Nginx to avoid the “Download failed” error when running behind reverse proxies.

> Networks assumed: external Docker networks named `frontend` and `backend`.  
> Domains: `nextcloud.katawarna.org` (Nextcloud) and `onlyoffice.katawarna.org` (ONLYOFFICE).

---

## Directory Layout

```
nextcloud-onlyoffice-split/
├─ .env
├─ nextcloud/
│  └─ compose.yaml
└─ onlyoffice/
   └─ compose.yaml
```

- `.env` is **shared** by both stacks so secrets stay in one place.
- ONLYOFFICE uses **named volumes** for logs, data, lib, and db.

---

## Environment Variables (`.env`)

```env
# ---- Nextcloud DB ----
DB_ROOT_PW=BEquyYNrsQY7n35u82RK
DB_NAME=nextcloud
DB_USER=nextcloud_admin
DB_PW=B2zXoRBBEGaiKChd7G8F

# ---- Nextcloud app ----
TRUSTED_DOMAINS=nextcloud.katawarna.org onlyoffice.katawarna.org
NC_ADMIN_USER=diazhernawan
NC_ADMIN_PASSWORD=2Me5u$4hbH8dYSkX#%9$

# ---- ONLYOFFICE ----
OO_JWT_SECRET=5961878d7c730866ae2c399df3733e9138b017b5127649ec

# ---- Whiteboard ----
JWT_SECRET_KEY=rHaizB9rbsEN4r6Un4JB

# ---- Timezone ----
TZ=Asia/Jakarta

```

> **Tip:** Consider rotating secrets and pinning images in production.

---

## Nextcloud Compose (`nextcloud/compose.yaml`)

```yaml
version: "3.9"

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
      - TZ=${TZ}
    volumes:
      - ./db:/var/lib/mysql
    networks: [backend]

  redis:
    image: redis:alpine
    container_name: nextcloud-redis
    restart: unless-stopped
    environment:
      - TZ=${TZ}
    networks: [backend]

  app:
    image: nextcloud:31.0.7
    container_name: nextcloud-app
    restart: unless-stopped
    depends_on: [redis, db]
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
      - TZ=${TZ}
    volumes:
      - ./app:/var/www/html
      - ./config:/var/www/html/config
      - /mnt/smb/nextcloud-data:/var/www/html/data
    networks: [frontend, backend]

  nextcloud-whiteboard-server:
    image: ghcr.io/nextcloud-releases/whiteboard:stable
    container_name: nextcloud-whiteboard-server
    restart: unless-stopped
    environment:
      NEXTCLOUD_URL: https://nextcloud.katawarna.org
      JWT_SECRET_KEY: ${JWT_SECRET_KEY}
      TZ: ${TZ}
    ports:
      - "3020:3002"
    networks: [frontend, backend]

networks:
  frontend:
    external: true
  backend:
    external: true

```

---

## ONLYOFFICE Compose (`onlyoffice/compose.yaml`)

```yaml
version: "3.9"

services:
  onlyoffice:
    image: onlyoffice/documentserver:latest
    container_name: onlyoffice
    restart: unless-stopped
    environment:
      - JWT_ENABLED=true
      - JWT_SECRET=${OO_JWT_SECRET}
      # Leave the header default ("Authorization") to match the NC app
      - WOPI_ENABLED=false
      - DICTIONARY_ENABLED=true
      - TZ=${TZ}
    ports:
      - "8080:80"
    networks: [frontend, backend]
    volumes:
      - onlyoffice_logs:/var/log/onlyoffice
      - onlyoffice_data:/var/www/onlyoffice/Data
      - onlyoffice_lib:/var/lib/onlyoffice
      - onlyoffice_db:/var/lib/postgresql
    entrypoint:
      - /bin/bash
      - -lc
      - |
        echo "Forcing https scheme inside ONLYOFFICE Nginx..."
        sed -Ei           's@(proxy_set_header[[:space:]]+X-Forwarded-Proto[[:space:]]+).+;@\1https;@'           /etc/onlyoffice/documentserver/nginx/includes/http-common.conf
        echo "Resulting X-Forwarded-Proto line:"
        grep -n 'X-Forwarded-Proto' /etc/onlyoffice/documentserver/nginx/includes/http-common.conf
        exec /app/ds/run-document-server.sh

networks:
  frontend:
    external: true
  backend:
    external: true

volumes:
  onlyoffice_logs:
  onlyoffice_data:
  onlyoffice_lib:
  onlyoffice_db:

```

### Why the `sed` hack?

ONLYOFFICE sometimes mis-detects the scheme when running behind certain LBs/proxies, which leads to mixed-content or JWT-download problems in the editor.  
We forcibly set `X-Forwarded-Proto https` **inside** the container’s Nginx so the service self-consistently believes it’s on HTTPS.

> Alternative: patch your edge proxy (Traefik) to **always** forward `X-Forwarded-Proto: https` to ONLYOFFICE and remove the `sed` block. If you do that, restart ONLYOFFICE and verify with `curl -I` that the header is present at the container boundary.

---

## Bring Up the Stacks

From the **root** directory (`nextcloud-onlyoffice-split/`):

```bash
# Create external networks once (if they don't exist yet)
docker network create frontend || true
docker network create backend  || true

# Nextcloud stack
cd nextcloud
docker compose --env-file ../.env up -d

# ONLYOFFICE stack
cd ../onlyoffice
docker compose --env-file ../.env up -d
```

Check health:

```bash
docker ps
docker logs -f nextcloud-app
docker logs -f onlyoffice
```

---

## Nextcloud → ONLYOFFICE Settings

In Nextcloud **Admin » ONLYOFFICE**:

- **ONLYOFFICE Docs address:** `https://onlyoffice.katawarna.org`
- **Secret key:** same as `OO_JWT_SECRET` in `.env`
- **Authorization header:** leave empty (defaults to `Authorization`)
- **Server address for internal requests from ONLYOFFICE Docs:** `https://nextcloud.katawarna.org`

Confirm with CLI (optional):

```bash
docker exec -u www-data nextcloud-app php occ config:app:get onlyoffice
```

---

## Troubleshooting

- **Download failed dialog** in editor
  - Ensure the sed line actually modified the file:
    ```bash
    docker exec -it onlyoffice bash -lc "grep -n 'X-Forwarded-Proto' /etc/onlyoffice/documentserver/nginx/includes/http-common.conf"
    ```
  - Restart ONLYOFFICE: `docker restart onlyoffice`.
- **JWT errors**
  - JWT secret must **match** between Nextcloud app and ONLYOFFICE env.
  - Keep `JWT_HEADER` default (`Authorization`) on both sides.
- **Logs**
  - Nextcloud: `docker logs -f nextcloud-app`
  - ONLYOFFICE: `docker logs -f onlyoffice`
  - With volumes, you can also inspect `/var/log/onlyoffice` on the host via `docker volume inspect` and mounting a debug container.

---

## Notes / Hardening

- Pin images (e.g., `onlyoffice/documentserver:9.0.4`, `nextcloud:31.0.7`) to avoid surprise changes.
- Back up **both** Nextcloud `/var/www/html` volumes and ONLYOFFICE named volumes.
- Limit which frontends can reach ONLYOFFICE at the proxy layer if you’d like to reduce exposure.
