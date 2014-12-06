
System
------

Clean install baremetal machine Debian GNU/Linux installation running GRSecurity, PaX, Xen and ZFS/SPL.
System had no other load than the running guest image and standard dom0 Operating system.

Problem
-------

What is the performance impact of running a Xen HVM as flatfile or ZFS zvol (4k, 8k) ?

Tests
-----

- Xen HVM with disk.img and swap.img on /zfs-tank pool
- Xen HVM with /dev/zvol/zfs-tank/bonnie and swap.img on /zfs-tank pool
- Xen HVM with /dev/zvol/zfs-tank/bonnie and /../zfs-tank/bonnie-swap
  o Test on volblocksize 4k and 8k


Result
------

Unconclusive. It seems that flatfile and zvol performance is pretty close in some, and quite
different in other fields. Generally zvol handling makes more sense from a sysops perspective.
Generally I noticed a drop in CPU demand when on zvol, need to investigate further.


Versions & Options used
-----------------------

- Debian GNU/Linux host
- GCC (Debian 4.7.2-5) 4.7.2
- Linux Kernel 3.16.5 (stable)
- GRSecurity/PaX 3.0-3.16.5-201410132000 (testing)
  o On Security Model and max. lock down
- Xen 4.4.1 (testing)
  o GRUB_CMDLINE_XEN="dom0_mem=4096M dom0_max_vcpus=2 dom0_vcpus_pin"
- ZFS 0.6.3
- SPL 0.6.3
- Xen HVM image created with --vcpus 4 --memory 4G --swap 5G --size 30G --dist wheezy --dir /zfs-tank
- Tool used for benchmarking was bonnie 1.97.1 (host) and bonnie 1.96 (guests)

  bonnie++ -u root -d /root -q >> result.csv
  bon_csv2html < result.csv > result.html

- Both 8k and 4k volblocksize on ZVOL was tested.
- Xen flat files were trasnferred to ZVOLs via
  dd if=/zfs-tank/bonnie/domains/bonnie/disk.img | pv | dd of=/dev/zvol/zfs-tank/bonnie

- Two identical SATA Disks

  o hdparam

    Model=ST3000DM001-9YN166, FwRev=CC4B, SerialNo=Z1F16L6F
    Config={ HardSect NotMFM HdSw>15uSec Fixed DTR>10Mbs RotSpdTol>.5% }
    RawCHS=16383/16/63, TrkSize=0, SectSize=0, ECCbytes=4
    BuffType=unknown, BuffSize=unknown, MaxMultSect=16, MultSect=16
    CurCHS=16383/16/63, CurSects=16514064, LBA=yes, LBAsects=5860533168
    IORDY=on/off, tPIO={min:120,w/IORDY:120}, tDMA={min:120,rec:120}

    Model ST3000DM001 explicitly supports Advanced Disk Format.

  o gdisk print

    Disk /dev/sda: 5860533168 sectors, 2.7 TiB
    Logical sector size: 512 bytes
    Disk identifier (GUID): F6445E8B-B329-434D-B2F1-9BA7FDF3701D
    Partition table holds up to 128 entries
    First usable sector is 34, last usable sector is 5860533134
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 6109 sectors (3.0 MiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            4096         1052671   512.0 MiB   FD00  
       2         1052672       545259520   259.5 GiB   FD00  Linux RAID
       3       545261568      5860533134   2.5 TiB     FD00  Linux RAID

  o /dev/sda3 and /dev/sdb3 is used directly by dm-luks its not part of any Linux RAID

  o Last partitions are dm-luks devices which feeds into zpool.
    Since zpool supports advanced format we created it with

    zpool create -o ashift=12 zfs-tank mirror crypto-disk-01 crypto-disk-02

  o ZFS was *not* used with extra SSD cache disk etc. and only limited amount of CPU and RAM was
    given to Dom0 (4GB RAM, 2 CPUs).




