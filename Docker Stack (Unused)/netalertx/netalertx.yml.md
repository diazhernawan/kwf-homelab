```
---yaml
services:
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    ports:
      - 8191:8191
    environment:
      - LOG_LEVEL=info
    restart: unless-stopped
    
networks:
  portainer_default:
    external: true
```