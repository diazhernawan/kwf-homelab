```yaml
# STARR-MEDIASERVER

services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: 1_prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
    volumes:
      - /home/diaz/1_prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: 2_radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/diaz/2_radarr:/config
      - /mnt/smb/Raptor/1_storage:/1_storage
    ports:
      - 7878:7878
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: 3_sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /home/diaz/3_sonarr:/config
      - /mnt/smb/Raptor/1_storage:/1_storage
    ports:
      - 8989:8989
    restart: unless-stopped

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: 6_bazarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
    volumes:
      - /home/diaz/6_bazarr:/config
      - /mnt/smb/Raptor/1_storage:/1_storage #optional
    ports:
      - 6767:6767
    restart: unless-stopped

networks: {}
```