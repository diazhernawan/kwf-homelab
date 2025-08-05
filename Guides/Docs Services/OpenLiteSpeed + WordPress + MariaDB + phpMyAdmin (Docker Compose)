# 🚀 OpenLiteSpeed + WordPress + MariaDB + phpMyAdmin (Docker Compose)

A production-ready stack with persistent storage and secure networking.

---

## 📂 Directory Structure

```
wp-ols/
├── db/                # MariaDB persistent data
├── lsws/
│   ├── admin_conf/    # OLS admin user persistence
│   ├── conf/          # OLS main configs
│   └── logs/          # OLS logs
├── ols/               # OLS web root (for serving WordPress)
├── wp/                # WordPress code & uploads
├── .env               # Environment variables (DB creds, etc.)
└── compose.yaml       # Docker Compose file
```

---

## ⚡️ Quickstart (Step by Step)

### 1. **Clone or Prepare Your Project Directory**

```bash
git clone <your-repo> wp-ols
cd wp-ols
```

*Or create and `cd` into it manually.*

---

### 2. **Create Required Folders**

```bash
mkdir -p db lsws/admin_conf lsws/conf lsws/logs ols wp
```

---

### 3. **Create Your `.env` File**

```ini
MYSQL_DATABASE=wordpress
MYSQL_ROOT_PASSWORD=y0oXowualfkAWLZqSIQ
MYSQL_USER=wpuser
MYSQL_PASSWORD=JULd2VUxEWh38wCfau8X
```

---

### 4. **Create Your `compose.yaml`**

Paste this YAML (edit ports if needed):

```yaml
services:
  ols:
    image: litespeedtech/openlitespeed:latest
    container_name: ols
    restart: unless-stopped
    ports:
      - "8088:80"       # HTTP
      - "7080:7080"     # WebAdmin
    volumes:
      - ./ols:/var/www/vhosts/localhost/html
      - ./lsws/conf:/usr/local/lsws/conf
      - ./lsws/logs:/usr/local/lsws/logs
      - ./lsws/admin_conf:/usr/local/lsws/admin/conf
    environment:
      - TZ=Asia/Jakarta
    networks:
      - frontend
      - backend

  db:
    image: mariadb:11
    container_name: ols-db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ./db:/var/lib/mysql
    networks:
      - backend

  wp:
    image: wordpress:php8.3
    container_name: ols-wp
    depends_on:
      - db
      - ols
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ./wp:/var/www/html
    networks:
      - frontend
      - backend
  
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: ols-phpmyadmin
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      #PMA_USER: ${MYSQL_USER}
      #PMA_PASSWORD: ${MYSQL_PASSWORD}
    networks:
      - frontend
      - backend

networks:
  frontend:
    external: true
  backend:
    external: true
```

---

### 5. **Create Docker Networks (One Time Only)**

```bash
docker network create frontend
docker network create backend
```

---

### 6. **Start Your Stack**

```bash
docker compose up -d
```

---

### 7. **Access Your Services**

* **WordPress:**       [http://YOUR\_SERVER\_IP:8080](http://YOUR_SERVER_IP:8080)
* **OpenLiteSpeed:**   [http://YOUR\_SERVER\_IP:8088](http://YOUR_SERVER_IP:8088)
* **OLS WebAdmin:**    [http://YOUR\_SERVER\_IP:7080](http://YOUR_SERVER_IP:7080)

  * Default: `admin` / `123456`
  * **Change password immediately!**
* **phpMyAdmin:**      [http://YOUR\_SERVER\_IP:8081](http://YOUR_SERVER_IP:8081)

  * Login with `wpuser` / your MYSQL\_PASSWORD or any DB user.

---

### 8. **First-Time Notes & Security**

* **phpMyAdmin login form**: Shown if `PMA_USER` and `PMA_PASSWORD` are **not** set.
* **OLS WebAdmin persistence**: Ensured by `lsws/admin_conf` volume mapping.
* **Restrict phpMyAdmin and OLS WebAdmin ports** for production, e.g., firewall or SSH tunnel only.
* **Back up** `db`, `wp`, `ols`, `lsws/conf`, and `lsws/admin_conf` regularly.

---

### 9. **Management Commands**

* **View logs:**        `docker compose logs -f`
* **Stop stack:**       `docker compose down`
* **Recreate/upgrade:** `docker compose pull && docker compose up -d`
* **Check status:**     `docker compose ps`

---

### 10. **Troubleshooting**

* If WordPress can’t connect to DB, check `.env` matches compose, and DB is fully up (`docker compose logs db`).
* If OLS admin user/password resets, check the `lsws/admin_conf` volume is present and mapped.

---

## 🏆 **You're Production Ready!**

For SSL, automatic backup, or advanced security: see [docs/SECURITY.md](docs/SECURITY.md) *(or ask ChatGPT for a hardening guide!)*

---
