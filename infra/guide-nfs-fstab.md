nfs mount using fstab

1. Install nfs-common:

   apt -y install nfs-common
   
3. Create mount for nfs:

        sudo mkdir /mnt/nfs/Raptor -p
   
5. Edit /etc/fstab:

        10.10.50.40:/mnt/ARC06/Raptor /mnt/nfs/Raptor nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0

6. Mount the Network Drive:

        sudo mount -a

7. reload daemon:

        sudo systemctl daemon-reload

7. Check the mount with:

        df -h

8. check parent directory: 

        ls -l /mnt/nfs







Mounts:

    
    10.10.50.20:/mnt/KWF01/Arcturus /mnt/nfs/Arcturus nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    10.10.50.20:/mnt/Pool-2/Mig2 /mnt/nfs/Mig2 nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    
    10.10.50.30:/mnt/ARC01/Galactica /mnt/nfs/Galactica nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    10.10.50.30:/mnt/ARC02/Pegasus /mnt/nfs/Pegasus nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    
    10.10.50.40:/mnt/ARC06/Raptor /mnt/nfs/Raptor nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    10.10.50.40:/mnt/ARC07/Frigate /mnt/nfs/Frigate nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
