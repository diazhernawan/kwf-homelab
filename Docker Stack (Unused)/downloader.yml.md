```yaml
version: "3.3"
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: 4_qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
      - WEBUI_PORT=8080
    volumes:
      - /home/diaz/4_qbittorrent:/config
      - /mnt/smb/Raptor/1_storage/torrents:/1_storage/torrents
    ports:
      - 8080:8080
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped

  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: 5_sabnzbd
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
    volumes:
      - /home/diaz/5_sabnzbd:/config
      - /mnt/smb/Raptor/1_storage:/1_storage
    ports:
      - 8085:8080
    restart: unless-stopped
      
  deluge:
    image: lscr.io/linuxserver/deluge:latest
    container_name: deluge
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
      - DELUGE_LOGLEVEL=error #optional
    volumes:
      - /home/diaz/8_deluge:/config
      - /mnt/smb/Raptor:/Raptor
    ports:
      - 8090:8112
      - 6882:6881
      - 6882:6881/udp
      - 58846:58846 #optional
    restart: unless-stopped

  jdownloader-2:
    container_name: stash_jdownloader2
    ports:
      - 5800:5800
    volumes:
      - /home/diaz/docker-data/Jdownloader:/config:rw
      - /mnt/smb/Raptor/8_pstore/pdownload/jdownloader:/jdownloader:rw
    image: jlesage/jdownloader-2
    restart: unless-stopped

networks: {}
```