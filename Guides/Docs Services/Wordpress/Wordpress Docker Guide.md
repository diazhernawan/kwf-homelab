# WordPress Docker Compose Tutorial

> **Reminder**: Before you start, ensure that:
>
> * Docker Engine and Docker Compose v2+ are installed.
> * Portainer is installed and running (optional but recommended).
> * You have created the necessary bind mount directories on your host:
>
>   * `config/`
>   * `wp-app/`
>   * `db-init/`
> * You have a valid `.env` file with `IP`, `DB_NAME`, and `DB_ROOT_PASSWORD`.

## Directory Structure

Your project should look like this:

```bash
.
├── config/          # PHP tweaks, nginx vhosts, etc.
│   └── php.conf.ini
├── wp-app/          # WordPress codebase (themes, plugins, uploads…)
├── db-init/         # *.sql or *.sh scripts for database initialization
├── docker-compose.yaml
└── .env             # IP, DB_NAME, DB_ROOT_PASSWORD
```

## .env File

```
IP=0.0.0.0
DB_NAME=wpdbtest
DB_ROOT_PASSWORD=your_root_password
```

## docker-compose.yaml

```yaml
version: "3.8"

services:
  wp:
    image: wordpress:latest
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "${IP}:8082:80"
    volumes:
      - ./config/php.conf.ini:/usr/local/etc/php/conf.d/conf.ini
      - wp_app:/var/www/html
    environment:
      WORDPRESS_DB_HOST: db
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
      - db_data:/var/lib/mysql
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
    external: true

volumes:
  wp_app:
  db_data:
```

## Usage

1. Clone or copy this repo.
2. Create the bind mount directories on the host:

   ```bash
   mkdir -p config wp-app db-init
   ```
3. Populate `config/php.conf.ini`, your WordPress files in `wp-app/`, and any init scripts in `db-init/`.
4. Fill in your `.env` file with correct values.
5. Start the stack:

   ```bash
   docker compose up -d
   ```
6. (Optional) Use Portainer to manage your containers via the web UI.

## Best Practices

* **Avoid `container_name`**: Let Compose auto-generate names for better scaling.
* **Use named volumes**: Improves portability and data management.
* **Healthchecks**: Ensure dependent services are ready before starting.
* **Networks**: Use external networks for multi-stack communication.
* **Environment files**: Keep secrets out of version control by using `.env`.
