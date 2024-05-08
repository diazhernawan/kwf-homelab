ZFS ARC MAX fro Truenas Scale

In Truenas Scale Shell in BYTES:

    cat /sys/module/zfs/parameters/zfs_arc_max
    <SIZE_IN_BYTES>
    
    echo SIZE_IN_BYTES >> /sys/module/zfs/parameters/zfs_arc_max
    
    echo 68000000000 >> /sys/module/zfs/parameters/zfs_arc_max
