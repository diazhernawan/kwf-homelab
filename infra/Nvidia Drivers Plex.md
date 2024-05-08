Install Nvidia Drivers Debian 12

1: Enable the non-free and contrib repository as an administrative user. 

open the /etc/apt/sources.list and add the non-free repository.

  FROM:
    
        deb http://ftp.au.debian.org/debian/ buster main
  TO:

        deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware

close & save. 

Update the update apt cache

        sudo apt update

2. Install nvidia-detect utility by execution of the below command:

        sudo apt -y install nvidia-detect


3. Detect your Nvidia card model and suggested Nvidia driver.
To do so execute the installed nvidia-detect command:

        nvidia-detect

4. Install Nvidia Driver

        sudo apt install nvidia-driver


5. Reboot

6. Disable Secure Boot

  6A. Run: mokutil --disable-validation or mokutil --enable-validation.
  6B. Choose a password between 8 and 16 characters long. Enter the same password to confirm it. (brabus18)
  6C. Reboot.
  6D. When prompted, press a key to perform MOK management.
  6E. Select "Change Secure Boot state".
  6F. Enter each requested character of your chosen password to confirm the change. Note that you have to press Return/Enter after each character.
  6G. Select "Yes".
  6H. Select "Reboot".

7. To confirm run:
   
        nvidia-smi
        lspci | egrep 'VGA|NVIDIA'
        lscpi -v

####Plex guide:

        https://www.linuxcapable.com/how-to-install-plex-media-server-on-debian-linux/
        https://linux.how2shout.com/install-plex-media-server-on-debian-11-bullseye-with-nginx-reverse-proxy/
        https://wiki.debian.org/NvidiaGraphicsDrivers




