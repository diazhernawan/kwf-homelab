Frigate LXC with OpenVino

1. Create a Priviledged LXC

	LXC Spec

		OS: Debian 12
		General: Priviledged=1 Nesting=1
		Template:Debian 12
		Disk:32G
		CPU:4 cores
		Memory: 8192
		Network: Vlan Tag = 2 (IOT) DHCP=1
		
	Edit LXC options

		Features: Nesting=1 NFS=1 SMB/CIFS=1 FUSE=1
		

	Start LXC, then Update & Upgrade.

		apt update && apt upgrade -y

2. CONFIGURE SMB
   	Install SMB/CIFS

		apt install cifs-utils -y


   	Create a folder in /mnt to mount the share

		mkdir /mnt/smb/Frigate -p


   	Create a hidden smb credentials file

		nano ~/.credentials.smb
	
   	.credentials.smb content

		user=diaz
		password= 
		domain=katawarnafilm.com

   	Edit the fstab file

		nano /etc/fstab
		
	Mount SMB share for Frigate (at /etc/fstab)

		//10.10.50.40/Frigate /mnt/smb/Frigate cifs uid=0,credentials=/root/.credentials.smb,iocharset=utf8,vers=3.0,noperm 0 0,nobrl

	
   	Reboot


   	Test Mount

		mount -a
		cd /mnt/smb/Frigate

   	Poweroff

   		nano poweroff

3. CONFIGURE INTEL IGPU PASSTHROUGH for OpenVino
   	In Proxmox shell navigate to LXC configuration directory

		cd /etc/pve/lxc/

   	Edit LXC configuration on Proxmox shell

   		nano (lxc-id).conf

   	Add LXC configuration to allow LXC to access iGPU

		lxc.cgroup2.devices.allow: c 226:0 rwm
		lxc.cgroup2.devices.allow: c 226:128 rwm
		lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file 0, 0
		lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
	
   	.credentials.smb content

		user=diaz
		password= 
		domain=katawarnafilm.com

   	Start the LXC
		
	Install intel gpu drivers

		apt install vainfo
		apt install intel-gpu-tools

   	3F. Check the igpu
   
		vainfo
		intel_gpu_top


4. Install Docker & Portainer
	Update & Upgrade the LXC

   		apt update && apt upgrade -y

 
	Install Docker

		apt install curl
		curl -fsSL https://get.docker.com -o get-docker.sh
		sh get-docker.sh

   	Test docker-run
   
   		docker run hello-world

	Create the portainer volume

		docker volume create portainer_data

   	Install Portainer

		docker run -d \
		--name="portainer" \
		--restart on-failure \
		-p 9000:9000 \
		-p 8000:8000 \
		-v /var/run/docker.sock:/var/run/docker.sock \
		-v portainer_data:/data \
		portainer/portainer-ce:latest
	
6. Frigate Installation
	Create Frigate config at /opt/frigate

		mkdir /opt/frigate
		cd /opt/frigate
		nano config.yml

	Content of /opt/frigate/config.yml

		mqtt:
		  enabled: False
		
		cameras:
		  dummy_camera: # <--- this will be changed to your actual camera later
		    enabled: False
		    ffmpeg:
		      inputs:
		        - path: rtsp://127.0.0.1:554/rtsp
		          roles:
		            - detect

	

   	Deploy Frigate using docker-compose in Portainer stacks

		version: "3.9"
		services:
		  frigate:
		    container_name: frigate
		    privileged: true # this may not be necessary for all setups
		    restart: unless-stopped
		    image: ghcr.io/blakeblackshear/frigate:stable
		    shm_size: "256mb" # update for your cameras based on calculation above
		    devices:
		      #- /dev/bus/usb:/dev/bus/usb # passes the USB Coral, needs to be modified for other versions
		      #- /dev/apex_0:/dev/apex_0 # passes a PCIe Coral, follow driver instructions here https://coral.ai/docs/m2/get-started/#2a-on-linux
		      - /dev/dri/renderD128 # for intel hwaccel, needs to be updated for your hardware
		    volumes:
		      - /etc/localtime:/etc/localtime:ro
		      - /opt/frigate/config.yml:/config/config.yml
		      - /mnt/smb/Frigate:/media/frigate
		      - type: tmpfs # Optional: 1GB of memory, reduces SSD/SD Card wear
		        target: /tmp/cache
		        tmpfs:
		          size: 1000000000
		    ports:
		      - "5000:5000"
		      - "8554:8554" # RTSP feeds
		      - "8555:8555/tcp" # WebRTC over tcp
		      - "8555:8555/udp" # WebRTC over udp
		    environment:
		      FRIGATE_RTSP_PASSWORD: "password"



WAIT FRIGATE PULL & INSTALL MIGHT TAKE TWO HOUR+
   
