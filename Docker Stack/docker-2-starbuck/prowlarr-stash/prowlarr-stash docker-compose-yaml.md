```yaml
---
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr-stash
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
    volumes:
      - ./2_prowlarr:/config
    ports:
      - "9696:9696"
    restart: unless-stopped
```