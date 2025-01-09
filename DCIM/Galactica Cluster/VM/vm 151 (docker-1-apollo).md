Date: 08/01/2025

Proxmox Node: Prox02
Virtual Machine: 151
Hostname: docker-3-apollo 

OS: Debian 12  
CPU (cores): 4
Type: x86-64-v2-AES  
RAM (MiB): 6144  
BIOS: Default (SeaBIOS)
Display: Serial terminal 0
SCSI Controller: VirtIO SCSI
HDD (GB):  64 

Network Device (Net0): 
	VLAN: 1001
	IP: 10.0.1.51
	MAC: BC:24:11:1D:E3:51

Services:
- portainer [[4. guide-install-portainer]]
- flaresolverr [[flaresolverr docker-compose-yaml]]
- heimdall [[heimdall docker-compose-yaml]]
- immich [[7. immich docker guide.md]] [[immich docker-compose-yaml]]
- 1_jellyseerr [[servarr docker-compose-yaml]]
- 2_prowlarr
- 3_radarr
- 4_sonarr
- 5_sabnzbd
- 6_bazarr
- 7_qbittorrent
- 8_deluge
- 9_overseerr

Docker images:
- portainer/portainer-ce
- ghcr.io/flaresolverr/flaresolverr:latest
- lscr.io/linuxserver/heimdall:latest
- ghcr.io/immich-app
- fallenbagel/jellyseerr:latest
- lscr.io/linuxserver/prowlarr:latest
- lscr.io/linuxserver/radarr:latest
- lscr.io/linuxserver/sonarr:latest
- lscr.io/linuxserver/sabnzbd:latest
- lscr.io/linuxserver/bazarr:latest
- lscr.io/linuxserver/qbittorrent:latest
- lscr.io/linuxserver/deluge:latest
- lscr.io/linuxserver/overseerr:latest
    
Integrations:
- n/a