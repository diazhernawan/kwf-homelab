# OpenLiteSpeed + WordPress + MariaDB + phpMyAdmin (Docker Compose)

> **Production-ready Docker stack for high-performance WordPress with persistent storage and secure networking**

---

\$1\`\`\`
ols-server/
├── db/                # MariaDB persistent data
├── ols-data/          # WordPress files (site code, plugins, uploads)
├── ols-config/
│   ├── conf/          # OLS main configs
│   ├── logs/          # OLS logs
│   └── admin\_conf/    # OLS WebAdmin user persistence
├── php-config/        # Custom php.ini for PHP settings (uploads, etc)
├── .env               # Environment variables
└── compose.yaml       # Docker Compose file

````


---

## 1. Project Preparation

**1.1. Change to your working directory:**
```bash
cd ols-server
````

\$1\`\`\`bash
mkdir -p db ols-data ols-config/conf ols-config/logs ols-config/admin\_conf php-config

````


---

## 2. Download WordPress Into `ols-data`
```bash
curl -LO https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
mv wordpress/* ols-data/
rm -rf wordpress latest.tar.gz
````

---

\$1

---

## 4. Increase WordPress Upload Limit (Optional, for Large Imports)

1. **Create `php-config/php.ini` in your project root with:**

   ```ini
   upload_max_filesize = 5120M
   post_max_size = 5120M
   memory_limit = 6144M
   max_execution_time = 600
   max_input_time = 600
   ```
2. **Update `compose.yaml`** to add this volume in the `ols` service:

   ```yaml
     - ./php-config/php.ini:/usr/local/etc/php/conf.d/custom.ini
   ```
3. **Restart your stack:**

   ```bash
   docker compose restart
   ```
4. **Check in WordPress:**
   Go to Media > Add New, or WordPress Importer—you’ll see the new 5GB limit!

\$2

## 5. Create Your `.env` File

```ini
# .env
MYSQL_DATABASE=wordpress_database
MYSQL_ROOT_PASSWORD=y0oXowualfkAWLZqSIQ
MYSQL_USER=BBF_MASTER
MYSQL_PASSWORD=JULd2VUxEWh38wCfau8X
```

---

## 6. Create `compose.yaml`

```yaml
services:
  ols:
    image: litespeedtech/openlitespeed:latest
    container_name: ols
    restart: unless-stopped
    ports:
      - "8088:80"
      - "7080:7080"
    volumes:
      - ./ols-data:/var/www/vhosts/localhost/html
      - ./ols-config/conf:/usr/local/lsws/conf
      - ./ols-config/logs:/usr/local/lsws/logs
      - ./ols-config/admin_conf:/usr/local/lsws/admin/conf
      - ./config/php.ini:/usr/local/etc/php/conf.d/custom.ini  # <--- If you did Step 3a
    environment:
      - TZ=Asia/Jakarta
    networks:
      - frontend
      - backend

  db:
    image: mariadb:11
    container_name: db
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

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
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

## 7. Create Docker Networks (One Time Only)

```bash
docker network create frontend
docker network create backend
```

---

## 8. Start the Stack

```bash
docker compose up -d
```

---

## 9. First-Time WordPress Setup

* Go to **OLS WebAdmin**: http\://YOUR\_SERVER\_IP:7080  (login: admin / 123456)

  * Make sure the Document Root is `/var/www/vhosts/localhost/html` (default)
  * Change WebAdmin password immediately
  * *(Recommended)*: Add a new admin user in WebAdmin → Users
* Go to **WordPress:** http\://YOUR\_SERVER\_IP:8088

  * Run the installer, use credentials from `.env`:

    * **Database Name:** `wordpress_database`
    * **Username:** `BBF_MASTER`
    * **Password:** `JULd2VUxEWh38wCfau8X`
    * **Database Host:** `db`   *(do not use `localhost`!)*
    * **Table Prefix:** `wp_` (or your choice)

---

## 10. Access All Services

* **WordPress:**       http\://YOUR\_SERVER\_IP:8088
* **OpenLiteSpeed:**   http\://YOUR\_SERVER\_IP:8088
* **OLS WebAdmin:**    http\://YOUR\_SERVER\_IP:7080
* **phpMyAdmin:**      http\://YOUR\_SERVER\_IP:8081

---

## 11. Security & Housekeeping

* Restrict access to `:8081` and `:7080` in production (use firewall or SSH tunnel).
* Regularly back up: `db`, `ols-data`, `ols-config/`
* Use the WordPress dashboard for updates (core, plugins, themes).
* If WordPress can’t update files, **fix permissions** as in Step 3.
* Don’t forget to set strong passwords for everything!

---

## 12. Common Management Commands

* **View logs:**        `docker compose logs -f`
* **Stop stack:**       `docker compose down`
* **Update stack:**     `docker compose pull && docker compose up -d`
* **Check status:**     `docker compose ps`

---

## 13. Troubleshooting

* If WordPress can’t connect to MariaDB, check `.env` and DB status (`docker compose logs db`).
* If OLS admin user resets, ensure the `ols-config/admin_conf` volume exists and is mapped.
* If you get permission errors in WP, repeat Step 3 or adjust folder ownership.

---

> For SSL, auto-backup, or advanced security: see [docs/SECURITY.md](docs/SECURITY.md) or ask ChatGPT for a hardening guide!

---
