#Linux_commands #RAID
# 目标 & 要求
> 在服务器上添加4块硬盘，实现使用`RAID5`级别将5块盘组成磁盘冗余阵列

- 设备添加有`sdb、sdc、sdd、sde、sdf`四块磁盘，分别创建相对应的逻辑分区
- 使用`RAID5`进行创建，四块磁盘作为RAID阵列，一块充当热备盘
- 使用`ext4`格式对RAID进行格式化操作
- 要求挂载到`/mnt/md0`上并设置开机自启


## 创建RAID阵列

> 使用`/dev/sd{b,c,d,e，f}1`创建`RAID5`磁盘冗余阵列
```shell
mdadm -Cv /dev/md0 -n 4 -l 5 -a yes -x 1 /dev/sd{b,c,d,e,f}1
# -C：创建 /dev/md0 的磁盘冗余阵列
# -v：显示信息；可重复使用显示更详细信息
# -n：使用的磁盘数量（小于等于实际的磁盘数量）
# -l：--level；使用的RAID级别
# -a：自动为其创建设备同步文件
# -x：指定空闲盘（热备磁盘）个数，空闲盘（热备磁盘）能在工作盘损坏后自动顶替
# 注意 n+x 等于实际使用的物理磁盘数量

mdadm -Q /dev/md0 
mdadm -D /dev/md0 
# -Q：查看摘要信息
# -D：查看创建RAID的信息
```
> 模拟RAID中磁盘损坏
```shell
mdadm /dev/md0 -f /dev/sdb
# 停用 /dev/md0 中的 /dev/sdb 磁盘，模拟损坏情况
# 观察RAID阵列的情况
mdadm /dev/md0 -a /dev/sdb
# -a：添加磁盘，系统默认会自动开始数据的同步工作
```
> 格式化+挂载
```shell
mkfs.[filesystem] /dev/md0
# 可以直接通过mkfs对RAID阵列进行格式化

mount /dev/md0 /mnt/md0
# 将其挂载到 /mnt/md0 上

df -Th
# 查看挂载情况
echo "/dev/md0 /mnt/md0 ext4 defaults 0 0" >> /etc/fstab
# 可以写入 /etc/fstab 中，开机自动挂载；注意使用 mount -a 检查
```

## 删除RAID阵列
- **注意数据的备份**
```shell
umount /mnt/md0 /dev/md0
# 首先对其进行卸载操作

mdadm -S /dev/md0
mdadm -r /dev/md0
# -S：停止指定RAID运行，释放一切资源
# -r：删除指定RAID阵列
```






-----------------------


```shell
lsblk
mdadm -Cv /dev/md0 -n 3 -l 5 /dev/sdc /dev/sdd /dev/sde /dev/sdf  
# 设置RAID实例

mdadm -Q /dev/md0 
# 查看它是否是一个md设备；它是否是一个md阵列的组件。显示有关所发现内容的信息。

mdadm -D /dev/md0
mkfs.ext4 /dev/md0 
# 分配文件系统

mkdir /Raid
mount /dev/md0 /Raid/ 
# 挂载

df -Th
```



以下是一些常用的 Linux 磁盘相关命令：

1. df：显示磁盘使用情况。可以使用 "df -h" 命令以人类可读的格式查看。
2.  du：显示文件或目录的磁盘使用情况。可以使用 "du -h" 命令以人类可读的格式查看。
3.  fdisk：磁盘分区工具，用于创建、删除和修改磁盘分区表。
4.  mkfs：用于创建文件系统。mkfs 后需要加上文件系统类型（如 ext4、ntfs 等）和设备名称（如 /dev/sda1）。
5.  mount：将一个文件系统挂载到指定的挂载点上。
6.  umount：卸载已经挂载的文件系统。
7.  lsblk：以树形结构查看块设备的信息，包括磁盘、分区和挂载点等。
8.  blkid：显示块设备的 UUID 和文件系统类型。
9.  badblocks：检查并标记坏块。
10.  smartctl：用于读取磁盘的 SMART 信息，从而判断磁盘的健康状态。
