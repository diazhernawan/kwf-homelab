# WordPress Docker Compose Tutorial

---

## 🚀 Usage

1. **Go to Directory** ./docker_volumes/wordpress
2. **Clean** db-data & wp-data:

   ```bash
   sudo rm -rf wp-data db-data
   ```
3. **Recreate** db-data & wp-data:

   ```bash
   sudo mkdir wp-data db-data
   ```
4. **Populate**:
   * `config/php.conf.ini` → PHP tweaks
   * `.env` → Populate with IP / DB_NAME / DB_ROOT_PASSWORD
5. **Start** containers:

   ```bash
   docker compose up -d
   ```
6. (Optional) **Portainer** for web UI container management.

---



GUIDE:

> **Bind-mount bliss & Compose V2 ready!**

---

## Prerequisites

* Docker Engine & Docker Compose v2+ installed
* (Optional) Portainer running for GUI container management
* Created bind-mount directories on host:

  * `config/`
  * `wp-app/`
  * `wp-data/`
  * `db-data/`
* A `.env` file with `IP`, `DB_NAME`, and `DB_ROOT_PASSWORD`

   ```bash
   mkdir -p config wp-app wp-data db_data
   touch .env
   ```

---

## 🗂️ Directory Structure

```bash
.
├── compose.yaml          # Compose V2 file (no version header!)
├── .env                  # env vars
├── config/
│   └── php.conf.ini      # PHP tweaks (ro mount)
├── wp-app/               # WordPress core, themes, plugins…
├── wp-data/              # DB init scripts (e.g. init.sql)
└── db_data/              # MySQL data dir (empty on first run)
```

---

## 🔒 Example `.env`

```dotenv
IP=0.0.0.0
DB_NAME=wpdbtest
DB_ROOT_PASSWORD=supersecret123
```

> **Tip:** Keep this file out of Git!

---

## 📝 Sample `config/php.conf.ini`

```ini
file_uploads = On
memory_limit = 500M
upload_max_filesize = 30M
post_max_size = 30M
max_execution_time = 600
```

---

## 🐳 `compose.yaml`

```yaml
---
services:
  wp:
    image: wordpress:latest
    container_name: wordpress
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "${IP}:8082:80"
    volumes:
      - ./config/php.conf.ini:/usr/local/etc/php/conf.d/conf.ini:ro
      - ./wp-app:/var/www/html
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: "${DB_NAME}"
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: "${DB_ROOT_PASSWORD}"
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost/ || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - wordpress-network

  pma:
    image: phpmyadmin/phpmyadmin
    container_name: wordpress-phpmyadmin
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "${IP}:8081:80"
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      UPLOAD_LIMIT: "50M"
    depends_on:
      db:
        condition: service_started
    networks:
      - wordpress-network

  db:
    image: mysql:8.4
    container_name: wordpress-mysqldb
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "${IP}:3306:3306"
    command:
      - mysqld
      - --mysql-native-password=ON
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    volumes:
      - ./db-data:/var/lib/mysql
      - ./wp-data:/docker-entrypoint-initdb.d:ro
    environment:
      MYSQL_DATABASE: "${DB_NAME}"
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u root -p\"${DB_ROOT_PASSWORD}\" > /dev/null 2>&1"]
      interval: 30s
      timeout: 10s
      retries: 5
    networks:
      - wordpress-network

networks:
  wordpress-network:
  proxy:
    external: true
```



## 💡 Best Practices

* **Healthchecks** prevent race-conditions.
* **External networks** for multi-stack setups (e.g. Traefik).
* **Env files** keep secrets off Git.
* **Bind mounts only**—code & data remain host-editable.

---

*Happy deploying—your WordPress stack just got ninja-level!* 🚀
