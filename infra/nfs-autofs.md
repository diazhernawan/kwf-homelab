#AUTOFS (/mnt/nfs/Raptor)

1. Create mount for nfs:

        mkdir /mnt/nfs

2. Edit the file /etc/auto.master:

        nano /etc/auto.master

4. Contents of /etc/auto.master

        /mnt/nfs /etc/auto.nfs --ghost --timeout=60

5. Edit the file /etc/auto.nfs:

        nano /etc/auto.nfs

6. Edit the nfs mount:

        Raptor -fstype=nfs4,rw 10.10.50.40:/mnt/ARC07/Raptor/

7. Check the mount with:

        df -h 

8. check parent directory: 

        ls -l /mnt/nfs









Mounts:


storage -fstype=nfs4,rw 10.10.50.40:/mnt/Tank/Raptor-Data/7_storage/medialibrary
