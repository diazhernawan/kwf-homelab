```yaml
---
services:
  meshcentral:
    restart: always
    container_name: meshcentral
    image: typhonragewind/meshcentral:latest
    ports:
      - 443:443 
    environment:
      - HOSTNAME=meshcentral #your hostname
      - REVERSE_PROXY=false #set to your reverse proxy IP if you want to put meshcentral behind a reverse proxy
      - REVERSE_PROXY_TLS_PORT=
      - IFRAME=false #set to true if you wish to enable iframe support
      - ALLOW_NEW_ACCOUNTS=true #set to false if you want disable self-service creation of new accounts besides the first (admin)
      - WEBRTC=true #set to true to enable WebRTC - per documentation it is not officially released with meshcentral, but is solid enough to work with. Use with caution
      - BACKUPS_PW=brabus18 #password for the autobackup function
      - BACKUP_INTERVAL=24 # Interval in hours for the autobackup function
      - BACKUP_KEEP_DAYS=10 #number of days of backups the function keeps
    volumes:
      - ./meshcentral/data:/opt/meshcentral/meshcentral-data #config.json and other important files live here. A must for data persistence
      - ./meshcentral/user_files:/opt/meshcentral/meshcentral-files #where file uploads for users live
      - ./meshcentral/backups:/opt/meshcentral/meshcentral-backups #Backups location
networks: {}
```