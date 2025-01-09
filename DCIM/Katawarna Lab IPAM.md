
---
IP address management
----
**Main (10.0.0.0/24) VLAN 1**
10.0.0.0/24

**Servers (10.0.1.0/24) VLAN 1001**
10.0.1.0/24

**Katawarna (10.0.2.0/24) VLAN 1002**
10.0.2.0/24

**IOT & Security (10.0.4.0/24) VLAN 1003**
10.0.3.0/24

**Management (10.0.5.0/24) VLAN 4**
10.0.4.0/24

**Guest vlan 1005**
10.0.5.0/24

---
**Main (10.0.0.0/24) VLAN 1**
---
Networking HW
10.0.0.1 Router - UDM Pro
10.0.0.3 10G Switch - USW Aggregation
10.0.0.5 2F Office - UAP LR
10.0.0.6 1F Home - UAP LR
10.0.0.7 2F Corridor - UAP LR


---
**Servers (10.0.1.0/24) VLAN 1001**
----
**Networking SW (10.0.1.10 - 10.0.1.19)**
10.0.1.11 tailscale [[vm 100 (tailscale-core-kwf)]]
10.0.1.12 pihole  [[lxc 101 (pihole-1)]]
10.0.1.14 cloudflared [[lxc 102 (cloudflared)]]

**NAS (10.0.1.20 - 10.0.1.29)**
10.0.1.20 Raptor
10.0.1.21 Arcturus
10.0.1.22 Omega


LXC & VM (10.0.1.30 - 10.0.1.99)
VM (10.0.1.30 - 10.0.1.49)
10.0.1.30 plex [[lxc 105 (plex)]]
10.0.1.31 jellyfin [[lxc 104 (jellyfin)]]

VM (10.0.1.50 - 10.0.1.99)
10.0.1.50 Homeassistant [[vm 250 (homeassistant)]]
10.0.1.51 docker-1-apollo [[vm 151 (docker-1-apollo)]]
10.0.1.52 docker-2-starbuck [[vm 152 (docker-2-starbuck)]]
10.0.1.53 docker-3-athena [[vm 153 (docker-3-athena)]]
10.0.1.54 docker-4-helo [[vm 154 (docker-4-helo)]]

Hypervisors (10.0.1.100 - 10.0.1.120)
10.0.1.101 prox01 [[Prox01]]
10.0.1.102 prox02 [[Prox01]]
10.0.1.120 PDM [[lxc 200 (pdm)]]

Workstations (10.0.1.210 - 10.0.1.220)
10.0.1.201 kwf-ws-1
10.0.1.202 kwf-ws-2
10.0.1.204 diaz-macbook

Reserved:
220 - 254


---
**Management (10.0.2.0/24) VLAN 1002**
---
10.0.2.100 R730 idrac
10.0.2.105 R510 idrac
10.0.2.110 X10ASLL-F


---
IOT & Security (10.0.3.0/24) VLAN 1003
---
10.0.3.9 MQTT
10.0.3.30 Homeassistant [[vm 250 (homeassistant)]]
10.0.3.54 docker-4-helo (1004) [[vm 154 (docker-4-helo)]]


**IoT (WIZ)** [[vm 250 (homeassistant)]]
10.0.3.11 Office 1 (a7526d)
10.0.3.12 Office 2 (a75c8d)
10.0.3.21 Studio 1 (9f7199)
10.0.3.22 Studio 2 (a7531f)
10.0.3.31 Bedroom 1 (a65f01)
10.0.3.32 Bedroom 2 (9860fd)
10.0.3.33 Bedroom 3 (f7cf2e)

**CCTV** [[vm 154 (docker-4-helo)]]
10.0.3.200 CCTV_Carport_Front
10.0.3.205 CCTV_Carport_Front
10.0.3.210 CCTV_Backyard


---
Katawarna (10.0.4.0/24) VLAN 1004
---
Home & WLAN

---
Guest (10.0.5.0/24) VLAN 1005
---
Guest VLAN