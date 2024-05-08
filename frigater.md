#Frigate LXC

1. PREP THE LXC

	LXC Spec
		General: Priviledged=1 Nesting=1
		Template:Debian 12
		Disk:32G
		CPU:4 cores
		Memory: 8192
		Network: Vlan Tag = 2 (IOT) DHCP=1

1B. Edit options
Features: Nesting=1 NFS=1 SMB/CIFS=1 FUSE=1
		

C. Start LXC, then Update & Upgrade.
	apt update && apt upgrade -y
