CIFS-SMB-MOUNT Debian / Ubuntu

1. update software repositories:

        sudo apt update

2. install cifs utilities:

        sudo apt install cifs-utils -y

3. create a hidden smb credentials file:

        sudo nano ~/.credentials.smb

3. fill smb credentials at ~/.credentials.smb:

        user=diaz
        password= 
        domain=katawarnafilm.com


4 create a folder in /media to mount the share:

        sudo mkdir /mnt/smb/Raptor -p

5. edit the fstab file:

        sudo nano /etc/fstab

6A. (VM) mount smb on /etc/fstab:

       //10.10.50.40/Raptor /mnt/smb/Raptor cifs uid=0,credentials=/home/diaz/.credentials.smb,iocharset=utf8,vers=3.0,noperm,nobrl 0 0

6B. (CT) mount smb on /etc/fstab:

       //10.10.50.40/Raptor /mnt/smb/Raptor cifs uid=0,credentials=/root/.credentials.smb,iocharset=utf8,vers=3.0,noperm,nobrl 0 0


7. reload daemon:

        sudo systemctl daemon-reload

8.  remount filesystems listed in fstab:

        sudo mount -a

9.  change directory to the share mount point:

        cd /mnt/smb/Raptor

7.  reboot to test the share is mounted on started:

        sudo reboot now

8. troubleshooting:
        sudo journalctl -xe



# Guide & Docs

        https://www.youtube.com/watch?v=8lGjHyvXL8Q&t=11s

        docs: https://man7.org/linux/man-pages/man8/mount.8.html
