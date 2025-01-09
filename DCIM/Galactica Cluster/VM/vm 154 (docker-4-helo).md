Date: 08/01/2025

Proxmox Node: Prox01
Virtual Machine: 154
Hostname: docker-4-helo 

OS: Debian 12  
CPU (cores): 6 
Type: x86-64-v2-AES  
RAM (MiB): 12288  
BIOS: Default (SeaBIOS)
Display: Serial terminal 0
SCSI Controller: VirtIO SCSI
HDD (GB):  32 

Passthrough: GPU Nvidia P400
	PCI Device: Raw Device
		Device: 0000:01:00.0
		All Functions: Checked
		ROM-Bar: Checked

Network Device (Net0): 
	VLAN 1001
	IP: 10.0.1.54  
	MAC: BC:24:11:71:21:CA

Network Device (Net1): 
	VLAN 1003
	IP: 10.0.3.54
	MAC: BC:24:11:CE:BD:0A

Services:
- portainer agent [[4. guide-install-portainer]]
- frigate [[frigate-openvino docker-compose.yaml]][[frigate-tensor docker-compose.yaml]]
- tdarr [[tdarr docker-compose-yaml]]

Docker images:
- portainer/agent
- ghcr.io/blakeblackshear/frigate:0.14.1-tensorrt
- ghcr.io/haveagitgat/tdarr:latest

Integrations:
- n/a