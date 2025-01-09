Date: 08/01/2025

Proxmox Node: Prox01
Virtual Machine: 153
Hostname: docker-3-athena 

OS: Debian 12  
CPU (cores): 6 
Type: x86-64-v2-AES  
RAM (MiB): 12288  
BIOS: Default (SeaBIOS)
Display: Serial terminal 0
SCSI Controller: VirtIO SCSI
HDD (GB):  32 

Network Device (Net0): 
	VLAN 1001
	IP: 10.0.1.53  
	MAC: BC:24:11:43:2A:77

Services:
- portainer agent [[4. guide-install-portainer]]
- homepage [[homepage docker-compose]]
- idrac6 [[idrac6 docker-compose]]
- ntp [[ntp docker-compose]]
- openspeedtest [[openspeedtest docker-compose]]
- speedtest-tracker [[speedtest-tracker docker-compose]]
- traefik [[traefik docker-compose]]
- uptime-kuma [[uptimekuma docker-compose]]

Docker images:
- portainer/agent
- ghcr.io/gethomepage/homepage:latest
- domistyle/idrac6
- cturra/ntp
- openspeedtest/latest
- lscr.io/linuxserver/speedtest-tracker:latest
- traefik:v3.3.1
- louislam/uptime-kuma:1
    
Integrations:
- n/a