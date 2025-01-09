Date: 08/01/2025

Proxmox Node: Prox02
Virtual Machine: 152
Hostname: docker-3-starbuck 

OS: Debian 12  
CPU (cores): 6
Type: x86-64-v2-AES  
RAM (MiB): 6144  
BIOS: Default (SeaBIOS)
Display: Serial terminal 0
SCSI Controller: VirtIO SCSI
HDD (GB):  32 

Network Device (Net0): 
	VLAN 1001
	IP: 10.0.1.52  
	MAC: BC:24:11:6D:03:CE

Services:
- portainer agent [[4. guide-install-portainer]]
- jdownloader-2 [[jdownloader-2 docker-compose-yaml]]
- prowlarr-stash[[prowlarr-stash docker-compose-yaml]]
- qbittorrent-stash [[qbittorrent-stash docker-compose-yaml]]

Docker images:
- portainer/agent
- jlesage/jdownloader-2
- lscr.io/linuxserver/prowlarr:latest
- lscr.io/linuxserver/qbittorrent:latest
    
Integrations:
- n/a