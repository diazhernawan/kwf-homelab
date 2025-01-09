---
services:
  idrac6:
    image: domistyle/idrac6
    container_name: idrac6
    restart: unless-stopped
    ports:
      - 5800:5800
      - 5900:5900
    environment:
      - IDRAC_HOST=10.0.2.105
      - IDRAC_USER=root
      - IDRAC_PASSWORD=calvin
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.idrac6.rule=Host(`idrac6.katawarna.id`)"
      - "traefik.http.routers.idrac6.entrypoints=https"
      - "traefik.http.routers.idrac6.tls=true"
      - "traefik.http.services.idrac6.loadbalancer.server.port=5800"
    networks:
      - proxy
    volumes:
      - /mnt/smb/Raptor:/mnt/smb  # Mounts the directory for persistent data.
networks:
  proxy:
    external: true
