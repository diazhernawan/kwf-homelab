CIFS-SMB-MOUNT Debian / Ubuntu

1. update software repositories:

```bash
sudo apt update
```

2. install cifs utilities:

```bash
sudo apt install cifs-utils -y
```

3. create a hidden smb credentials file:

```bash
sudo nano ~/.credentials.smb
```

3. fill smb credentials at ~/.credentials.smb:

```bash
user=diaz
password= 
domain=katawarnafilm.com
```


4 create a folder in /media to mount the share:

```bash
sudo mkdir /mnt/smb/Raptor -p
```

5. edit the fstab file:

```bash
sudo nano /etc/fstab
```

6A. (VM) mount smb on /etc/fstab:

```bash
//10.10.50.40/Raptor /mnt/smb/Raptor cifs uid=0,credentials=/home/diaz/.credentials.smb,iocharset=utf8,vers=3.0,noperm,nobrl 0 0
```

6B. (CT) mount smb on /etc/fstab:

```bash
//10.10.50.40/Raptor /mnt/smb/Raptor cifs uid=0,credentials=/root/.credentials.smb,iocharset=utf8,vers=3.0,noperm,nobrl 0 0
```


7. reload daemon:

```bash
sudo systemctl daemon-reload
```

8.  remount filesystems listed in fstab:

```bash
sudo mount -a
```

9.  change directory to the share mount point:

```bash
cd /mnt/smb/Raptor
```

7.  reboot to test the share is mounted on started:

```bash
sudo reboot now
```

8. troubleshooting:
```bash
sudo journalctl -xe
```


# Guide & Docs

```bash
https://www.youtube.com/watch?v=8lGjHyvXL8Q&t=11s
docs: https://man7.org/linux/man-pages/man8/mount.8.html
```
