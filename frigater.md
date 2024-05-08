Frigate LXC

#Proxmox Prep
1. PREP THE LXC

	1A. Edit options

		Create LXC Specs
		General: Priviledged=1 Nesting=1
		Template:Debian 12
		Disk:32G
		CPU:4 cores
		Memory: 8192
		Network: Vlan Tag = 2 (IOT) DHCP=1
		
	1B. Edit options

		Features: Nesting=1 NFS=1 SMB/CIFS=1 FUSE=1
		

	1C. Start LXC, then Update & Upgrade.

		apt update && apt upgrade -y


#SMB-CONFIG

2. CONFIGURE SMB
	2A. Install SMB/CIFS

		apt install cifs-utils -y


	2B. Create a folder in /mnt to mount the share

		mkdir /mnt/smb/Frigate -p


	2C. Create a hidden smb credentials file

		nano ~/.credentials.smb
	
   	2D. credentials.smb

		user=diaz
		password= 
		domain=katawarnafilm.com
