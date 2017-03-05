


# Tuning KVM performance




## I/O Disk

### Scheduler in guest

### Scheduler in host

### Cache

[IBM Reference](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/liaat/liaatbpkvmguestcache.htm)

[SuSE Reference](https://www.suse.com/documentation/sles11/book_kvm/data/sect1_4_chapter_book_kvm.html)

Supported modes:

* **writethrough**

writethrough mode is the default caching mode. With caching set to writethrough mode, the host page cache is enabled, but the disk write cache is disabled for the guest. Consequently, this caching mode ensures data integrity even if the applications and storage stack in the guest do not transfer data to permanent storage properly (either through fsync operations or file system barriers). Because the host page cache is enabled in this mode, the read performance for applications running in the guest is generally better. However, the write performance might be reduced because the disk write cache is disabled.

* **writeback**

With caching set to writeback mode, both the host page cache and the disk write cache are enabled for the guest. Because of this, the I/O performance for applications running in the guest is good, but the data is not protected in a power failure. As a result, this caching mode is recommended only for temporary data where potential data loss is not a concern.

* **none**

With caching mode set to none, the host page cache is disabled, but the disk write cache is enabled for the guest. In this mode, the write performance in the guest is optimal because write operations bypass the host page cache and go directly to the disk write cache. If the disk write cache is battery-backed, or if the applications or storage stack in the guest transfer data properly (either through fsync operations or file system barriers), then data integrity can be ensured. However, because the host page cache is disabled, the read performance in the guest would not be as good as in the modes where the host page cache is enabled, such as writethrough mode.

* **unsafe**

Caching mode of unsafe ignores cache transfer operations completely. As its name implies, this caching mode should be used only for temporary data where data loss is not a concern. This mode can be useful for speeding up guest installations, but you should switch to another caching mode in production environments.

* **directsync**
This mode causes qemu-kvm to interact with the disk image file or block device with both O_DSYNC and O_DIRECT semantics, where writes are reported as completed only when the data has been committed to the storage device, and when it is also desirable to bypass the host page cache. Like cache=writethrough, it is helpful to guests that do not send flushes when needed. It was the last cache mode added, completing the possible combinations of caching and direct access semantics.

Summary:
For local or direct-attached storage, it is recommended that you use writethrough mode, as it ensures data integrity and has acceptable I/O performance for applications running in the guest, especially for read operations. However, caching mode none is recommended for remote NFS storage, because direct I/O operations (O_DIRECT) perform better than synchronous I/O operations (with O_SYNC). Caching mode none effectively turns all guest I/O operations into direct I/O operations on the host, which is the NFS client in this environment.

# Benchmarks / Measurements

## Disk I/O throughput

### dd
The following command can be used
```
time dd if=/dev/zero of=/tmp/test oflag=direct bs=64k count=16000
```
Or this recommended one
```
dd bs=1M count=256 if=/dev/zero of=test conv=fdatasync
```


Example of execution
```
sysadmin@ubuntu:~/vSBC_tests$ time dd if=/dev/zero of=/tmp/test oflag=direct bs=64k count=16000

16000+0 records in
16000+0 records out
1048576000 bytes (1,0 GB, 1000 MiB) copied, 218,058 s, 4,8 MB/s

real	3m38.185s
user	0m0.728s
sys	0m19.964s
```

### hdparm

´´´
sysadmin@ubuntu:~/vSBC_tests$ sudo hdparm -t /dev/vda1

/dev/vda1:
 Timing buffered disk reads:  74 MB in  3.17 seconds =  23.33 MB/sec
sysadmin@ubuntu:~/vSBC_tests$ sudo hdparm -T /dev/vda1

/dev/vda1:
 Timing cached reads:   552 MB in  2.00 seconds = 275.87 MB/sec
 ´´´


# Useful links
