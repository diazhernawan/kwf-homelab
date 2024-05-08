nfs mount using autofs (/mnt/nfs/Raptor)


1. Install autofs
        
        sudo apt-get install autofs

2. Create mount for nfs:

        mkdir /mnt/nfs

3. Edit the file /etc/auto.master:

        nano /etc/auto.master

5. Configure auto.master /etc/auto.master

        /mnt/nfs /etc/auto.nfs --ghost --timeout=60

6. Edit the file /etc/auto.nfs:

        nano /etc/auto.nfs

7. Configure the nfs mount:

        Raptor -fstype=nfs4,rw 10.10.50.40:/mnt/ARC07/Raptor/

8. Check the mount with:

        df -h 

9. check parent directory: 

        ls -l /mnt/nfs









Mounts:


Arcturus -fstype=nfs4,rw 10.10.50.20:/mnt/ARC04/Arcturus
Galactica -fstype=nfs4,rw 10.10.50.30:/mnt/ARC01/Galactica
Pegasus -fstype=nfs4,rw 10.10.50.30:/mnt/ARC02/Pegasus
