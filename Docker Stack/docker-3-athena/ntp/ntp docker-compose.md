---
services:
  ntp:
    image: cturra/ntp
    container_name: ntp
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
    ports:
      - "123:123/udp"
    read_only: true
    tmpfs:
      - /etc/chrony:rw,mode=1750
      - /run/chrony:rw,mode=1750
    volumes:
      - ./chrony.conf:/etc/chrony/chrony.conf  # Mount your custom chrony.conf
      - ./chrony_logs:/var/log/chrony  # For logs (optional)
      - ./chrony_data:/var/lib/chrony  # To persist chrony database