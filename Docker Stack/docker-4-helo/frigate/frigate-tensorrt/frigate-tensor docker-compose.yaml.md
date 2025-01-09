
###Frigate docker compose for NVIDIA GPU

```yaml
---
services:
  frigate:
    container_name: frigate
    privileged: true # this may not be necessary for all setups
    restart: unless-stopped
    image: ghcr.io/blakeblackshear/frigate:0.14.1-tensorrt
    shm_size: "128mb" # update for your cameras based on calculation above
    deploy: # Nvidia GPU support
      resources:
        reservations:
          devices:
            - driver: nvidia
#              device_ids: ["0"]
              count: 1 # number of GPUs
              capabilities: [gpu]
    runtime: nvidia
    devices:
#      - /dev/bus/usb:/dev/bus/usb # Passes the USB Coral, needs to be modified for other versions
#      - /dev/nvidia0:/dev/nvidia0
#      - /dev/nvidiactl:/dev/nvidiactl
#      - /dev/nvidia-modeset:/dev/nvidia-modeset
#      - /dev/nvidia-uvm:/dev/nvidia-uvm
#      - /dev/nvidia-uvm-tools:/dev/nvidia-uvm-tools
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./config:/config
      - ./config/config.yml:/config/config.yml
      - /mnt/smb/frigate:/media/frigate
      - type: tmpfs # Optional: 1GB of memory, reduces SSD/SD Card wear
        target: /tmp/cache
        tmpfs:
          size: 1000000000
    ports:
      - "8971:8971"
      - "5000:5000" # Internal unauthenticated access. Expose carefully.
      - "8554:8554" # RTSP feeds
      - "8555:8555/tcp" # WebRTC over tcp
      - "8555:8555/udp" # WebRTC over udp
    environment:
      - FRIGATE_RTSP_PASSWORD=password
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility,video
      - YOLO_MODELS=yolov4-tiny-416,yolov7-tiny-416
      - USE_FP16=false
```
