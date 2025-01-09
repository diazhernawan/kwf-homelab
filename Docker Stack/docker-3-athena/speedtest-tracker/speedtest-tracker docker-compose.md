services:
  speedtest-tracker:
    container_name: speedtest-tracker
    image: lscr.io/linuxserver/speedtest-tracker:latest
    ports:
      - '8072:80'   # Maps port 80 inside the container to port 8072 on the host for HTTP access.
      - '8472:443'  # Maps port 443 inside the container to port 8472 on the host for HTTPS access.
    environment:
      - PUID=1000  # Sets the user ID the service will run as.
      - PGID=1000  # Sets the group ID the service will run as.
      - APP_KEY=base64:3DQ7V3w1pxwa4jgIUE1dfWgEatQMB9mquO5Vmbbk1n0= # Application key for Laravel framework security. Generate a App Key here - https://speedtest-tracker.dev
      - DB_CONNECTION=sqlite  # Uses SQLite as the database.
      - SPEEDTEST_SCHEDULE="*/10 * * * *"  # Cron schedule, runs every 3 hours.
      - DISPLAY_TIMEZONE=Asia/Jakarta  # Sets the timezone for displaying data in the UI.
      - APP_DEBUG=true
    volumes:
      - ./data:/config  # Mounts the directory for persistent data.
    restart: unless-stopped  # Ensures the container restarts unless it is explicitly stopped.