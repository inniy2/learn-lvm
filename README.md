## Learn LVM  
- - - -  
#### References  
- [Linux Basics LVM Tutorial](https://getpocket.com/redirect?url=https%3A%2F%2Fwww.ostechnix.com%2Flinux-basics-lvm-logical-volume-manager-tutorial%2F&formCheck=fd0395b4ca2f9265e1e85d4812bf4525)  
- [resize2fs issue with xfs](https://stackoverflow.com/questions/26305376/resize2fs-bad-magic-number-in-super-block-while-trying-to-open)  
- [ext4 mount option](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/deployment_guide/s1-filesystem-ext4-mount)  


#### Basic command line  
# 1) Check disk  
```bash
root$>  fdisk -l
```
```bash
[root@testmysql102 ~]# fdisk -l

Disk /dev/sda: 384.1 GB, 384083184640 bytes, 750162470 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk label type: dos
Disk identifier: 0x18dd812a

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048   750162436   375080194+  83  Linux
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

Disk /dev/sdb: 1000.2 GB, 1000204886016 bytes, 1953525168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: gpt
Disk identifier: 7C435765-9590-4894-8E31-BB57FF7269C0


#         Start          End    Size  Type            Name
```
> the host has two disk /dev/sda /dev/sdb  
> /dev/sda has only single partition(physical volumn) /dev/sda1  
> we can use 1000G of /dev/sdb  


```bash
root$>  sfdisk -l
```
```bash
[root@testmysql102 ~]# sfdisk -l

Disk /dev/sda: 46695 cylinders, 255 heads, 63 sectors/track
sfdisk: Warning: The partition table looks like it was made
  for C/H/S=*/116/17 (instead of 46695/255/63).
For this listing I'll assume that geometry.

Units: cylinders of 1009664 bytes, blocks of 1024 bytes, counting from 0

   Device Boot Start     End   #cyls    #blocks   Id  System
/dev/sda1   *      1+ 380406- 380406- 375080194+  83  Linux
sfdisk:                 start: (c,h,s) expected (1,4,9) found (0,32,33)

sfdisk:                 end: (c,h,s) expected (1023,115,17) found (615,115,17)

/dev/sda2          0       -       0          0    0  Empty
/dev/sda3          0       -       0          0    0  Empty
/dev/sda4          0       -       0          0    0  Empty

Disk /dev/sdb: 121601 cylinders, 255 heads, 63 sectors/track
Units: cylinders of 8225280 bytes, blocks of 1024 bytes, counting from 0

   Device Boot Start     End   #cyls    #blocks   Id  System
/dev/sdb1          0+ 121601- 121602- 976762583+  ee  GPT
sfdisk:                 start: (c,h,s) expected (0,0,2) found (0,0,1)

/dev/sdb2          0       -       0          0    0  Empty
/dev/sdb3          0       -       0          0    0  Empty
/dev/sdb4          0       -       0          0    0  Empty
```
> sfdisk will show each partition usage  


# 2) Create Partitions  
```bash
root$>  fdisk /dev/sdb
```
> m: help  
> n: create new partition  
> partition number: 1 ~ 4   
> First sector: Enter  
> Last sectore: +220G (example)  
> p: print partition  
> w: save to disk  
```bash
[root@testmysql102 ~]# fdisk /dev/sdb
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition number (1-128, default 1): 1
First sector (34-1953525134, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-1953525134, default 1953525134): +220G
Created partition 1


Command (m for help):
Partition number (2-128, default 2): 2
First sector (34-1953525134, default 461375488):
Last sector, +sectors or +size{K,M,G,T,P} (461375488-1953525134, default 1953525134): +220G
Created partition 2


Command (m for help): n
Partition number (3-128, default 3): 3
First sector (34-1953525134, default 922748928):
Last sector, +sectors or +size{K,M,G,T,P} (922748928-1953525134, default 1953525134): +220G
Created partition 3


Command (m for help): n
Partition number (4-128, default 4): 4
First sector (34-1953525134, default 1384122368):
Last sector, +sectors or +size{K,M,G,T,P} (1384122368-1953525134, default 1953525134): +270G
Created partition 4


Command (m for help): p

Disk /dev/sdb: 1000.2 GB, 1000204886016 bytes, 1953525168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: gpt
Disk identifier: 7C435765-9590-4894-8E31-BB57FF7269C0


#         Start          End    Size  Type            Name
 1         2048    461375487    220G  Linux filesyste
 2    461375488    922748927    220G  Linux filesyste
 3    922748928   1384122367    220G  Linux filesyste
 4   1384122368   1950353407    270G  Linux filesyste

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```


# 3) Update the kernel to save the changes without restarting the system
```bash
root$>  partprobe
```
> At this point you can see the partition informtaion by following command  
```bash
root$> fdisk -l
```
```bash
[root@testmysql102 ~]# fdisk -l

Disk /dev/sda: 384.1 GB, 384083184640 bytes, 750162470 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk label type: dos
Disk identifier: 0x18dd812a

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048   750162436   375080194+  83  Linux
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

Disk /dev/sdb: 1000.2 GB, 1000204886016 bytes, 1953525168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: gpt
Disk identifier: 7C435765-9590-4894-8E31-BB57FF7269C0


#         Start          End    Size  Type            Name
 1         2048    461375487    220G  Linux filesyste
 2    461375488    922748927    220G  Linux filesyste
 3    922748928   1384122367    220G  Linux filesyste
 4   1384122368   1950353407    270G  Linux filesyste
```

# 4) Create Physical Volumes
> If we do not have lib the instal lvm2 first,  
```bash
root$>  yum -y install lvm2
```
```bash
root$>  pvcreate /dev/sdb1 /dev/sdb2 /dev/sdb3 /dev/sdb4
```
```bash
[root@testmysql102 ~]# pvcreate /dev/sdb1 /dev/sdb2 /dev/sdb3 /dev/sdb4
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdb2" successfully created.
  Physical volume "/dev/sdb3" successfully created.
  Physical volume "/dev/sdb4" successfully created.
```
```bash
root$>  pvdisplay
```
```bash
[root@testmysql102 ~]# pvdisplay
  "/dev/sdb1" is a new physical volume of "220.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               220.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               t9VYIP-VhBO-CpKS-FRzK-Pqip-ZpOk-hK0PEl

  "/dev/sdb4" is a new physical volume of "270.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb4
  VG Name
  PV Size               270.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               eCcGwN-f5hg-QZkG-fMcZ-M3et-PGLw-Yj56ih

  "/dev/sdb2" is a new physical volume of "220.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb2
  VG Name
  PV Size               220.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               RKBcGi-n3nX-gAbS-5XLc-MqQq-13b6-wGS6DX

  "/dev/sdb3" is a new physical volume of "220.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb3
  VG Name
  PV Size               220.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               QagpaD-88mL-wU0w-LYpI-IkN7-i7KE-Weow5w
```
> We can varify the physical volumns  


# 5) Create Volume Groups
```bash
root$>  vgcreate vg1 /dev/sdb1 /dev/sdb2 /dev/sdb3
```
> from vg1 you can see the name from `df -h`  
> we left /dev/sdb4 for the expand example  
```bash
[root@testmysql102 ~]# vgcreate vg1 /dev/sdb1 /dev/sdb2 /dev/sdb3
  Volume group "vg1" successfully created
```
```bash
root$>  vgdisplay
```
```bash
[root@testmysql102 ~]# vgdisplay
  --- Volume group ---
  VG Name               vg1
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <659.99 GiB
  PE Size               4.00 MiB
  Total PE              168957
  Alloc PE / Size       0 / 0
  Free  PE / Size       168957 / <659.99 GiB
  VG UUID               MJnR7X-ucFx-YXSf-FZ3p-c8Pw-yP25-Y3xps8
```
> We can varify volumn group vg1



# 6) Create Logical Volume
```bash
root$>  lvcreate -L 650G vg1 -n lv1
```
> Size can be various  
```bash
[root@testmysql102 ~]# lvcreate -L 650G vg1 -n lv1
  Logical volume "lv1" created.
```
```bash
root$>  lvdisplay
```
```bash
[root@testmysql102 ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg1/lv1
  LV Name                lv1
  VG Name                vg1
  LV UUID                JdS4UN-eeLu-uaPD-XcRT-nXhH-JbFx-iZD8Kk
  LV Write Access        read/write
  LV Creation host, time testmysql102, 2018-11-06 14:04:09 +0900
  LV Status              available
  # open                 0
  LV Size                650.00 GiB
  Current LE             166400
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
> we can varify logical volume  



# 7) Format as ext4  
```bash
root$>  mkfs.ext4 /dev/vg1/lv1
```
```bash
[root@testmysql102 ~]#  mkfs.ext4 /dev/vg1/lv1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
42598400 inodes, 170393600 blocks
8519680 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2319450112
5200 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

# 8) Format as xfs  
```bash
TBD
```

# 9) Mount  
```bash
root$>  mount -o nobarrier /dev/vg1/lv1 /mnt/
```
```bash
root$>  cat /proc/mounts
```
```bash
... ...
/dev/mapper/vg1-lv1 /mnt ext4 rw,relatime,nobarrier,data=ordered 0 0
```
```bash
root$>  df -h
```
```bash
[root@testmysql102 ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs              16G     0   16G   0% /dev
tmpfs                 16G     0   16G   0% /dev/shm
tmpfs                 16G  129M   16G   1% /run
tmpfs                 16G     0   16G   0% /sys/fs/cgroup
/dev/sda1            352G   16G  319G   5% /
/dev/mapper/vg1-lv1  640G   73M  608G   1% /mnt
```
> varify the mount  
```bash
root$>  cd /mnt
        touch file1 file2 file3
```


# 10) Extend Volume Group Size
```bash
root$>  vgextend vg1 /dev/sdb4
```
> Using left over physical volumn (/dev/sdb4), we can extend the size  
```bash
[root@testmysql102 mnt]# vgextend vg1 /dev/sdb4
  Volume group "vg1" successfully extended
```
```bash
root$>  lvresize -L +270G /dev/vg1/lv1 
```
> Logical Volume resize!  
```bash
[root@testmysql102 mnt]# lvresize -L +270G /dev/vg1/lv1
  Size of logical volume vg1/lv1 changed from 650.00 GiB (166400 extents) to 920.00 GiB (235520 extents).
  Logical volume vg1/lv1 successfully resized.
```

# 11) Resize filesystem depends on format (ext4 = resize2fs, xfs = xfs_growfs)
- ext4: resize2fs  
```bash
root$>  resize2fs /dev/vg1/lv1
```
```bash
[root@testmysql102 mnt]# resize2fs /dev/vg1/lv1
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vg1/lv1 is mounted on /mnt; on-line resizing required
old_desc_blocks = 82, new_desc_blocks = 115
The filesystem on /dev/vg1/lv1 is now 241172480 blocks long.
```
```bash
root$>  lvdisplay /dev/vg1/lv1 
```
> we can varify resized filesystem  
```bash
[root@testmysql102 mnt]# lvdisplay /dev/vg1/lv1
  --- Logical volume ---
  LV Path                /dev/vg1/lv1
  LV Name                lv1
  VG Name                vg1
  LV UUID                JdS4UN-eeLu-uaPD-XcRT-nXhH-JbFx-iZD8Kk
  LV Write Access        read/write
  LV Creation host, time testmysql102, 2018-11-06 14:04:09 +0900
  LV Status              available
  # open                 1
  LV Size                920.00 GiB
  Current LE             235520
  Segments               4
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

# 12) Remove Logical Volume  
```bash
root$>  cd ..
        umount /mnt/
```
```bash
root$>  lvremove /dev/vg1/lv1
```
```bash
[root@testmysql102 /]# lvremove /dev/vg1/lv1
Do you really want to remove active logical volume vg1/lv1? [y/n]: y
  Logical volume "lv1" successfully removed
```


# 13) Remove Volume Group  
```bash
root$>  vgremove /dev/vg1
```
```bash
[root@testmysql102 /]#  vgremove /dev/vg1
  Volume group "vg1" successfully removed
```


# 14) Remove Physical Volume  
```bash
root$>  pvremove /dev/sdb1 /dev/sdb2 /dev/sdb3 /dev/sdb4
```
```bash
[root@testmysql102 /]# pvremove /dev/sdb1 /dev/sdb2 /dev/sdb3 /dev/sdb4
  Labels on physical volume "/dev/sdb1" successfully wiped.
  Labels on physical volume "/dev/sdb2" successfully wiped.
  Labels on physical volume "/dev/sdb3" successfully wiped.
  Labels on physical volume "/dev/sdb4" successfully wiped.
```

# 15) Remove Partitions  
```bash
root$>  fdisk /dev/sdb
```
> d: delete partition  
> partition number: 1 ~ 4  
> p: print partitions  
> w: save to disk  
```bash
[root@testmysql102 /]# fdisk /dev/sdb
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d
Partition number (1-4, default 4): 1
Partition 1 is deleted

Command (m for help): d
Partition number (2-4, default 4): 2
Partition 2 is deleted

Command (m for help): d
Partition number (3,4, default 4): 3
Partition 3 is deleted

Command (m for help): d
Selected partition 4
Partition 4 is deleted

Command (m for help): p

Disk /dev/sdb: 1000.2 GB, 1000204886016 bytes, 1953525168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: gpt
Disk identifier: 7C435765-9590-4894-8E31-BB57FF7269C0


#         Start          End    Size  Type            Name

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

# 14) Update the kernel to save the changes without restarting the system
```bash
root$>  partprobe
```







