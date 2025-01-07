Tailscale: https://www.youtube.com/watch?v=QJzjJozAYJo&t=164s

1. Create a Privildged LXC / VM

LXC Spec:
    
       OS: Debian 12
       General: Priviledged=1 Nesting=1
       Template:Debian 12
       Disk:32G
       CPU: 1 cores
       Memory: 2048

Set static ip for tailscale lxc:
  
        ip a

Set static ip on lxc option or UDM PRO

Test the connection: 

        ping 1.1.1.1 or 8.8.8.8

Update the CT:

        apt-get update && apt-get upgrade -y

reboot

2. Tailscale Installation:

Install dependencies:
  
    apt install curl ethtool networkd-dispatcher

Add Tailscale's package signing key and repository:

    curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
    curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list

Install Tailscale:

    sudo apt-get update
    sudo apt-get install tailscale

Configure ip4 forwarding:

    echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
    echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
    sudo sysctl -p /etc/sysctl.d/99-tailscale.conf

Firewall allow masquerading with this command: 

    firewall-cmd --permanent --add-masquerade

Connect your machine to your Tailscale network and authenticate in your browser:
  
    tailscale up --advertise-routes=10.10.50.0/24,192.168.1.0/24 --advertise-exit-node --accept-routes

Reboot

3. Tailscale Site to site Config:

Configure GRO / ETHTOOL:

    NETDEV=$(ip route show 0/0 | cut -f5 -d' ')
    sudo ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro-list off

Configure autostart on boot:

    printf '#!/bin/sh\n\nethtool -K %s rx-udp-gro-forwarding on rx-gro-list off \n' "$(ip route show 0/0 | cut -f5 -d" ")" | sudo tee /etc/networkd-dispatcher/routable.d/50-tailscale
    sudo chmod 755 /etc/networkd-dispatcher/routable.d/50-tailscale

Test the created script to ensure it runs successfully on your machine:

    sudo /etc/networkd-dispatcher/routable.d/50-tailscale
    test $? -eq 0 || echo 'An error occurred.'

Re-enable tailscale:

    tailscale down
    tailscale up --advertise-routes=10.10.50.0/24,192.168.1.0/24 --accept-routes --advertise-exit-node 

```
tailscale up --advertise-routes=10.0.0.0/24,10.0.1.0/24,10.0.2.0/24,10.0.3.0/24,10.0.4.0/24 --accept-routes --advertise-exit-node
```

4. Config the static route in UDM PRO / Firewall / Router
