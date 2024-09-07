#Linux #Ansible_Module
[TOC]
# Parted模块
- 这个模块允许使用`parted`命令行工具配置块设备分区。
- 可以进行磁盘的管理

> 关键字
```
device            指定硬盘设备路径 比如 /dev/vdb
label             指定分区表类型 gpt mbr
number            指定分区序号
part_start        分区起始位置
part_end          分区结束位置
state             指定操作方式 present创建 absent删除 info查信息(默认)
flage             指定的分区类型
```

> 在主机中的使用

```yml
- name: create vgm for prod
  parted:
		device: /dev/sdb
		number: 1
		state: present
		part_end: "20%"
		resize: true
		flags: [ lvm ]
  when: inventory_hostname in groups['prod']
```

# Lvg模块

> 关键字
```
vg       卷组名称 
state    present创建(默认) 或者 absent删除
force    在删除时使用。yes表示允许删除带逻辑卷的卷组，默认为false
pvs      指定物理卷 
pesize   设定pe大小。默认为4
```

> 在主机中的使用
```yml
- name: create vg research 100%
  lvg:
		vg: research
		pvs: /dev/sdb1
		pesize: 16
```

# Lvol模块
> 关键字
```
lv        定义逻辑卷名称
vg        逻辑卷的空间来自哪个vg
state     present创建(默认) 或者 absent删除
size      定义逻辑卷大小
force     删除和压缩逻辑卷大小。默认为no。需要时开启，避免磁盘的损坏
```

> 在主机中的使用
```yaml
- name: create lvm 1500M
  lvol:
		vg: research
		lv: data
		size: 1500
```


# Filesystem模块

> 关键字
```
dev      要格式化的分区 
fstype   文件系统类型 比如 ext4 xfs 
force    强制格式化,如果以前分区中有数据
```

> 在主机中的使用
```yml
- name: create filesystem
  filesystem:
		fstype: xfs
		dev: /dev/research/data
```

# Mount模块
> 关键字
```
path:       挂载点。
src:        挂载的文件。
fstype:     挂载的硬盘类型。
opts:       传递给mount命令的参数
						ro：用唯读模式挂上。
						rw：用可读写模式挂上。
						sync：在同步模式下执行。
						auto、noauto：打开/关闭自动挂上模式。
						remount：将一个已经挂下的档案系统重新用不同的方式挂上。例如原先是唯读的系统，现在用可读写的模式重新挂上。
state:      present	    开机时挂载，仅将挂载配置写入/etc/fstab
						mounted  	挂载设备，并将配置写入/etc/fstab
						unmounted	卸载设备，不会清除/etc/fstab写入的配置
						absent		卸载设备，并清理/etc/fstab写入的配置
```

> 在主机中的使用
```yml
- name: Mount up device by labe
  mount:
		path: /dev/research/data
		src: /mnt/research_data
		state: present
```


