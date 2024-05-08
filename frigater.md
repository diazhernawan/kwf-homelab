Frigate LXC

#Proxmox Prep
1. PREP THE LXC

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
