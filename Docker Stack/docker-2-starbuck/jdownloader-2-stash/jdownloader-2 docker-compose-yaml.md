```yaml
---
services:
  jdownloader:
    image: jlesage/jdownloader-2
    container_name: jdownloader-2-stash
    ports:
      - 5800:5800
    volumes:
      - ./jdownloader-2:/config:rw
      - /mnt/nfs/Raptor/8_pstore/pdownload/jdownloader:/output:rw
    environment:
      - USER_ID=1000 # Set according to your system user
      - GROUP_ID=1000 # Set according to your system group
    restart: unless-stopped
```