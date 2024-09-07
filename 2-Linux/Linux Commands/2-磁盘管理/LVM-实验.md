#Linux_commands #磁盘管理
# LVM实验
## 实验目标
> 创建 `VgTest` 卷组。
> 创建 `LvTest` 逻辑卷。

> 使用 `/dev/sdb` 和 `/dev/sdc` 两个物理卷【大小分别为`10G`】。

> 挂载到 `/mnt/LvTest/` 目录下，并写入配置文件。

> 使用 `/dev/sdd1` 为 `LVTest` 扩容【`/dev/sdc1` 大小为 `3G`】。

> 逆向缩容，使`/dev/sd{a,b,c}`三块磁盘置空。

## 创建LV
### 创建物理卷和卷组
- 通过 `lsblk` 查看当前磁盘信息
```shell
[root@Servere ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   40G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   39G  0 part 
  ├─cs-root 253:0    0 36.9G  0 lvm  /
  └─cs-swap 253:1    0  2.1G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk 
sdc           8:32   0   10G  0 disk 
sdd           8:48   0   10G  0 disk
```
> 使用 `pvcreate` & `vgcreate` 分别创建物理卷和卷组
```shell
[root@Servere ~]# pvcreate /dev/sdb /dev/sdc 
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.

[root@Servere ~]# vgcreate  VgTest /dev/sdb /dev/sdc 
  Volume group "VgTest" successfully created
```
> 使用 `pvdisplay` & `vgdisplay` 查看创建好的物理卷和卷组
```shell
[root@Servere ~]# pvdisplay 
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               VgTest
  PV Size               10.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               2559
  Allocated PE          0
  PV UUID               j2RqkR-l7uZ-E4zZ-306C-lVXK-L38M-LEge20
   
  --- Physical volume ---
  PV Name               /dev/sdc
  VG Name               VgTest
  PV Size               10.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               2559
  Allocated PE          0
  PV UUID               Wjps5a-fp9w-cHcr-0xTm-2hTw-UsHR-6sgnpE

[root@Servere ~]# vgdisplay 
  --- Volume group ---
  VG Name               VgTest
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               19.99 GiB
  PE Size               4.00 MiB
  Total PE              5118
  Alloc PE / Size       0 / 0   
  Free  PE / Size       5118 / 19.99 GiB
  VG UUID               2SYOGi-73dK-wisb-X6sk-kDxg-hSOv-m8rGsn
```
> 使用 `pvscan` & `vgscan` & `lvscan` 扫描创建好的物理卷和卷组
```shell
[root@Servere ~]# pvscan 
  PV /dev/sdb    VG VgTest          lvm2 [<10.00 GiB / <10.00 GiB free]
  PV /dev/sdc    VG VgTest          lvm2 [<10.00 GiB / <10.00 GiB free]
  PV /dev/sda2   VG cs              lvm2 [<39.00 GiB / 0    free]
  Total: 3 [<58.99 GiB] / in use: 3 [<58.99 GiB] / in no VG: 0 [0]

[root@Servere ~]# vgscan 
  Found volume group "VgTest" using metadata type lvm2
  Found volume group "cs" using metadata type lvm2

[root@Servere ~]# lvscan 
  ACTIVE            '/dev/cs/swap' [2.08 GiB] inherit
  ACTIVE            '/dev/cs/root' [36.91 GiB] inherit
```
### 创建逻辑卷
> 使用 `lvcreate` 创建逻辑卷
```shell
[root@Servere ~]# lvcreate -l 100%FREE -n LvTest /dev/VgTest
  Logical volume "LvTest" created.
```
> 使用 `lvscan` & `lvdisplay`扫描和查看创建好的逻辑卷信息
```shell
[root@Servere ~]# lvscan 
  ACTIVE            '/dev/VgTest/LvTest' [19.99 GiB] inherit
  ACTIVE            '/dev/cs/swap' [2.08 GiB] inherit
  ACTIVE            '/dev/cs/root' [36.91 GiB] inherit
[root@Servere ~]# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/VgTest/LvTest
  LV Name                LvTest
  VG Name                VgTest
  LV UUID                iFsJX5-xJEm-g8S3-43KJ-etcs-F4VR-5hYlkS
  LV Write Access        read/write
  LV Creation host, time Servere.lab.example.com, 2023-05-22 09:51:43 +0800
  LV Status              available
  # open                 0
  LV Size                19.99 GiB
  Current LE             5118
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```
### 挂载
```shell
[root@Servere ~]# mkdir /mnt/LvTest
[root@Servere ~]# mount /dev/VgTest/LvTest /mnt/LvTest/
[root@Servere ~]# echo "/dev/mapper/VgTest-LvTest /mnt/LvTest/ ext4 defaults 0 0" >> /etc/fstab 
[root@Servere ~]# mount -a
[root@Servere ~]# df -Th
Filesystem                Type      Size  Used Avail Use% Mounted on
/dev/mapper/cs-root       xfs        37G  2.1G   35G   6% /
/dev/mapper/VgTest-LvTest ext4       20G   45M   19G   1% /mnt/LvTest
```
## 扩容LV
> 注意事项：
- umount卸载挂载点
- [[#^b0e0a3|重新设置逻辑卷大小]]（lvresize）
- [[#^3542b6|检测磁盘错误]]（e2fsck -f）
- [[#^7160c5|更新逻辑卷信息]]（resize2fs）
### 分配逻辑卷组
> 物理卷 > 卷组 > 逻辑卷
```shell
[root@Servere ~]# fdisk /dev/sdd

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xa9dbbfa0.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-20971519, default 20971519): +3G # 为sdd1分配3G大小的容量

Created a new partition 1 of type 'Linux' and of size 3 GiB.

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e # 使用`8e`格式的
Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): p
Disk /dev/sdd: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa9dbbfa0

Device     Boot Start     End Sectors Size Id Type
/dev/sdd1        2048 6293503 6291456   3G 8e Linux LVM

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
### 扩容卷组

- vgextend 【vg_name】 【device】
```shell
[root@Servere ~]# df -Th
Filesystem                Type      Size  Used Avail Use% Mounted on
/dev/mapper/cs-root       xfs        37G  2.1G   35G   6% /
/dev/mapper/VgTest-LvTest ext4       20G   45M   19G   1% /mnt/LvTest
[root@Servere ~]# vgextend VgTest /dev/sdd1
  Physical volume "/dev/sdd1" successfully created.
  Volume group "VgTest" successfully extended
```
### 扩容逻辑卷

^b0e0a3

- lvextend -l +100%FREE 【vgname】/【lvname】
```shell
[root@Servere ~]# lvextend /dev/mapper/centos  -L +2.99G
  Rounding size to boundary between physical extents: 2.99 GiB.
  Size of logical volume VgTest/LvTest changed from 19.99 GiB (5118 extents) to 22.98 GiB (5884 extents).
  Logical volume VgTest/LvTest successfully resized.
```
### 检测磁盘错误

^3542b6

- 需要解除挂载目录
- 仅适用于 `ext4` 文件系统
```shell
[root@Servere ~]# umount /mnt/LvTest 
[root@Servere ~]# e2fsck -f /dev/mapper/VgTest-LvTest 
e2fsck 1.45.6 (20-Mar-2020)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/mapper/VgTest-LvTest: 11/1966080 files (0.0% non-contiguous), 167442/7858176 blocks
```
### 扩容文件系统

^7160c5

> 方法一：
- resize2fs -f 【/dev/mapper/...】
```shell
[root@Servere ~]# resize2fs /dev/mapper/VgTest-LvTest -f
resize2fs 1.45.6 (20-Mar-2020)
Filesystem at /dev/mapper/VgTest-LvTest is mounted on /mnt/LvTest; on-line resizing required
old_desc_blocks = 3, new_desc_blocks = 3
The filesystem on /dev/mapper/VgTest-LvTest is now 6025216 (4k) blocks long.
```

> 方法二：
- 将`/dev/sdd`剩余空间添加到`/dev/mapper/VgTest-LvTest`内。
- 直接使用`lvextend`中`-r`参数。
```shell
[root@Servere ~]# lvextend -L +6.99G /dev/mapper/VgTest-LvTest /dev/sdd2 -r
  Rounding size to boundary between physical extents: 6.99 GiB.
  Size of logical volume VgTest/LvTest changed from 22.98 GiB (5884 extents) to <29.98 GiB (7674 extents).
  Logical volume VgTest/LvTest successfully resized.
resize2fs 1.45.6 (20-Mar-2020)
Filesystem at /dev/mapper/VgTest-LvTest is mounted on /mnt/LvTest; on-line resizing required
old_desc_blocks = 3, new_desc_blocks = 4
The filesystem on /dev/mapper/VgTest-LvTest is now 7858176 (4k) blocks long.
```


## 缩容
> 注意事项：
- [[#^448f68|卸载]]（umount）
- [[#^1f63e2|检测]]（e2fsck -f）
- [[#^3a2965|更新逻辑卷信息大小]]（resize2fs）
- [[#^94ebaa|重置逻辑卷大小]]（lvresize）
### 卸载

^448f68

```shell
[root@Servere ~]# umount /mnt/LvTest 
```
### 检测磁盘错误

^1f63e2

```shell
[root@Servere ~]# e2fsck -f /dev/mapper/VgTest-LvTest 
e2fsck 1.45.6 (20-Mar-2020)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/mapper/VgTest-LvTest: 11/1966080 files (0.0% non-contiguous), 167442/7858176 blocks
```

### 更新逻辑卷信息

^3a2965

```shell
[root@Servere ~]# resize2fs /dev/mapper/VgTest-LvTest 
resize2fs 1.45.6 (20-Mar-2020)
The filesystem is already 7858176 (4k) blocks long.  Nothing to do!
```
### 重置逻辑卷大小
> 逻辑卷 > 卷组 > 物理卷
^94ebaa

```shell
[root@Servere ~]# lvremove /dev/mapper/VgTest-LvTest 
Do you really want to remove active logical volume VgTest/LvTest? [y/n]: y
  Logical volume "LvTest" successfully removed.
```
### 重置卷组
```shell
[root@Servere ~]# vgremove VgTest 
  Volume group "VgTest" successfully removed
```
### 重置物理卷
- 通过 `pvremove` 删除创建的物理卷
```shell
[root@Servere ~]# pvremove /dev/sdd{1,2}
  Labels on physical volume "/dev/sdd1" successfully wiped.
  Labels on physical volume "/dev/sdd2" successfully wiped.
```
- 通过 `fdisk` 删除创建的物理卷
```shell
[root@Servere ~]# fdisk /dev/sdd
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d
Partition number (1,2, default 2): 

Partition 2 has been deleted.

Command (m for help): d

Selected partition 1
Partition 1 has been deleted.

Command (m for help): p

Disk /dev/sdd: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa9dbbfa0

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
