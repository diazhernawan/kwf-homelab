PLEX INSTALL

1: Update the Debian System:

        sudo apt update && sudo apt upgrade -y

2: Install the Required Packages:

        sudo apt install dirmngr ca-certificates software-properties-common apt-transport-https curl -y

3: Import Plex Repository:

        curl -fsSL https://downloads.plex.tv/plex-keys/PlexSign.key | gpg --dearmor | sudo tee /usr/share/keyrings/plex.gpg > /dev/null

4. Add Plex Repo:

        echo "deb [signed-by=/usr/share/keyrings/plex.gpg] https://downloads.plex.tv/repo/deb public main" | sudo tee /etc/apt/sources.list.d/plexmediaserver.list


5. Install Plex Media Server:

        sudo apt update
        sudo apt install plexmediaserver -y

6: Verify Plex Media Server:

        systemctl status plexmediaserver



7: Configure UFW Firewall for Plex Media Server on Debian:
Enable UFW Firewall

        sudo apt install ufw -y
        sudo ufw enable

8. Add Plex Media Server Port Rules:

        sudo ufw allow 32400
        Step 3: Additional UFW Rules for Plex
        sudo ufw allow 1900/udp
        sudo ufw allow 3005/tcp
        sudo ufw allow 5353/udp
        sudo ufw allow 8324/tcp
        sudo ufw allow 32410:32414/udp
        sudo ufw allow ssh

9. Access the WebGUI:
    
        http://127.0.0.1:32400/web



Plex guide:

        https://www.linuxcapable.com/how-to-install-plex-media-server-on-debian-linux/
        https://linux.how2shout.com/install-plex-media-server-on-debian-11-bullseye-with-nginx-reverse-proxy/






