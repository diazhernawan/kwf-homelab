# kwf-homelab


https://www.youtube.com/watch?v=8lGjHyvXL8Q&t=11s

# update software repositories
sudo apt update

# install cifs utilities
sudo apt install cifs-utils -y

# create a hidden smb credentials file
sudo nano ~/.credentials.smb

user=diaz
password= 
domain=katawarnafilm.com


# create a folder in /media to mount the share
sudo mkdir /mnt/smb/Raptor -p

# edit the fstab file
sudo nano /etc/fstab

# On CT - Mount /etc/fstab
//10.10.50.40/Raptor /mnt/smb/Raptor cifs uid=0,credentials=/root/.credentials.smb,iocharset=utf8,vers=3.0,noperm,nobrl 0 0

# On VM - Mount /etc/fstab
//10.10.50.40/Raptor /mnt/smb/Raptor cifs uid=0,credentials=/home/diaz/.credentials.smb,iocharset=utf8,vers=3.0,noperm,nobrl 0 0



#reload daemon
sudo systemctl daemon-reload

# remount filesystems listed in fstab
sudo mount -a

# change directory to the share mount point
cd /mnt/smb/Raptor

# reboot to test the share is mounted on started
sudo reboot now

# documentation
docs: https://man7.org/linux/man-pages/man8/mount.8.html
