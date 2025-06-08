# **WordPress Docker Compose Tutorial**

## Reminder: Before you start, ensure that:

💡 **Reminder:**  
Install **Docker** & **Portainer**, and **create your bind-mount folders** (`config/`, `wp-app/`, `db-init/`) **before** running `docker compose up`.

```
1. **Host OS**: any modern Linux or macOS (Windows WSL2 also works)  
2. **Docker Engine** (latest stable)  
3. **Docker Compose v2+** (CLI plugin: `docker compose`)  
4. **Portainer** (optional GUI)
```










## Step 2 - Install & configure Tailscale


1. Tailscale dependencies
```
apt install curl ethtool networkd-dispatcher sudo
```

2. Add Tailscale's package signing key and repository
```bash
curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
```

3. Install Tailscale
```bash
sudo apt-get update
sudo apt-get install tailscale
```

4. Configure IP forwarding
```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

5. Firewall allow masquerading
```bash
firewall-cmd --permanent --add-masquerade
```

## Step 3 - Activate Tailscale subnets


6. Connect to Tailscale and authenticate
```bash
tailscale up --advertise-routes=10.0.1.0/24,10.0.2.0/24,10.0.3.0/24,10.0.4.0/24 --advertise-exit-node --accept-routes
```

7. Reboot

## Step 4 - Tailscale Site-to-Site Config

8. Configure GRO / ETHTOOL

```bash
NETDEV=$(ip route show 0/0 | cut -f5 -d' ')
sudo ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro-list off
```

9. Configure autostart on boot
```bash
printf '#!/bin/sh\n\nethtool -K %s rx-udp-gro-forwarding on rx-gro-list off \n' "$(ip route show 0/0 | cut -f5 -d" ")" | sudo tee /etc/networkd-dispatcher/routable.d/50-tailscale
sudo chmod 755 /etc/networkd-dispatcher/routable.d/50-tailscale
```

11. Test the created script
```bash
sudo /etc/networkd-dispatcher/routable.d/50-tailscale
test $? -eq 0 || echo 'An error occurred.'
```

12.  Re-enable Tailscale
```bash
tailscale up --advertise-routes=10.0.1.0/24,10.0.2.0/24,10.0.3.0/24,10.0.4.0/24 --advertise-exit-node -snat-subnet-routes=false --accept-routes
```

13. Approve subnets in tailscale.com machines

14. Configure the static route in UDM PRO / Firewall / Router

