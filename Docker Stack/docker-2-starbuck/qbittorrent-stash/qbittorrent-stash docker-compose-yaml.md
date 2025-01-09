```yaml
---
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent-stash
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - ./qbittorrent:/config
      - /mnt/nfs/Raptor:/Raptor
    ports:
      - 8080:8080
      - 6890:6881
      - 6890:6881/udp
    restart: unless-stopped
networks: {}
```