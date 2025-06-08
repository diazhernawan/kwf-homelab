# WordPress Docker Compose Tutorial

> **Bind-mount bliss & Compose V2 ready!**

---

## Prerequisites

* Docker Engine & Docker Compose v2+ installed
* (Optional) Portainer running for GUI container management
* Created bind-mount directories on host:

  * `config/`
  * `wp-app/`
  * `wp-data/`
  * `db_data/`
* A `.env` file with `IP`, `DB_NAME`, and `DB_ROOT_PASSWORD`

---

## 🗂️ Directory Structure

```bash
.
├── compose.yaml          # Compose V2 file (no version header!)
├── .env                  # env vars
├── config/
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

## 🐳 `compose.yaml`

```yaml
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
    image: mysql:latest
    container_name: wordpress-mysqldb
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "${IP}:3306:3306"
    command:
      - --default_authentication_plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    volumes:
      - ./db_data:/var/lib/mysql
      - ./wp-data:/docker-entrypoint-initdb.d:ro
    environment:
      MYSQL_DATABASE: "${DB_NAME}"
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "--silent"]
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

---

## 🚀 Usage

1. **Clone** this repo.
2. **Create** bind-mount folders:

   ```bash
   mkdir -p config wp-app wp-data db_data
   ```
3. **Populate**:

   * `config/php.conf.ini` → PHP tweaks
   * `wp-app/` → WordPress code
   * `wp-data/` → DB init scripts
4. **Fill in** `.env` with real values.
5. **Start** containers:

   ```bash
   docker compose up -d
   ```
6. (Optional) **Portainer** for web UI container management.

---

## 💡 Best Practices

* **Healthchecks** prevent race-conditions.
* **External networks** for multi-stack setups (e.g. Traefik).
* **Env files** keep secrets off Git.
* **Bind mounts only**—code & data remain host-editable.

---

*Happy deploying—your WordPress stack just got ninja-level!* 🚀
