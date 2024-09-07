#Linux #Ending_Exam #RHSA #Practical_exams
# 试题命令
## Node1配置
1. 配置 IP
* 开启 node1 主机
```
nmtui
配置相关的 IP 地址
```
2. yum 仓库的配置
```ini
vim yum.repo
	[baseos]
	name=baseos yum
	baseurl=
	enabled=1
	gpgcheck=0

	[appstresam]
	name=appstream yum
	baseurl=
	enabled=1
	gpgcheck=0

yum list
yum makechache
```

3. 配置 SE Linux
```shell
systemctl status httpd
systemctl enable httpd
systemctl start httpd
	setenforce 0//报错处理
tail /var/log/messages
semanage port -a -t http_port_t -p tcp 82 #设置selinux规则
restorecon -Rv /var/www/html
systemctl start httpd
systemctl status httpd
curl node1:82 html
```
4. 创建用户账户
```shell
groupadd sysmgrs
useradd -g sysmgrs natasha
useradd -g sysmgrs harry
useradd -s /sbin/nologin sarah
echo flectrag | passwd --stdin natasha
echo flectrag | passwd --stdin harry
echo flectrag | passwd --stdin sarah
id natasha
id harry
id sarah
```
5. 配置计划任务
```shell
systemctl status crond
crontab -u natasha -e
	*/2 * * * * logger "EX200 in progress"
crontab -u natasha -l//检查是否配置成功
```
6. 创建协作目录
```shell
mkdir /home/managers
chgrp sysmgrs /home/managers
chmod 2770 /home/managers
ll -d /home/managers
```
7. 配置 NTP --- 时钟同步
```shell
systemctl status chronyd
vim /etc/chrony.conf
	server materials.example.com iburst
systemctl restart chronyd.service
chronyc source -v//检查
```
8. 配置 autofs 自动挂载
```shell
systemctl status autofs
mkdir /rhome
vim /etc/auto.master
	/rhome /etc/auto.rhome
vim /etc/auto.rhome
	testuser -rw,sync 172.2.254.254:/rhome/testuser
systemctl enable autofs.servise
systemclt start autofs.service
cd /rhome
su - testuser
	exit
ll//此时应有 testuser 文件夹
``` 
9. 配置 /var/tmp/fstab 权限
```shell
cp /etc/fstab /var/tmp/fstab
setfacl -m u:natasha:rw- /var/tmp/fstab
setfacl -m u:harry:--- /var/tmp/fstab
getfacl /var/tmp/fstab
```
10. 配置用户账户 
```shell
useradd -u 3533 manalo
echo flectrag | passwd --stdin manalo
id manalo
```
11. 查找文件
```shell
mkdir /root/findfiles
	find / -user jacques -type f -exec cp -av {} /root/findfiles \;
```  
12. 查找字符串
```shell
cat /use/share/xml/iso-codes/iso_639_3.xml | grep ng > /root/list
cat /root/list
```
13. 创建归档（压缩文件）
```shell
tar cvzf /usr/share/xml/iso-codes/iso_639_3.xml | grep ng > /root/list
cat /root/list //检查
【 z -gzip】
【 -xz】
【 -bzip2】
```
14. 创建容器
```shell

```
15. 为容器配置永久存储
```

```


## Node2配置
1. 重置密码
```shell
e
rd.break
	mount -o rw,remount /sysroot
	chroot /sysroot
		passwd root
		flectrag
		flectrag
		touch /.autorelabel
		exit
	exit
```
2. 配置 yum 仓库
```ini
vim yum.repo
	[baseos]
	name=baseos yum
	baseurl=
	enabled=1
	gpgcheck=0

	[appstresam]
	name=appstream yum
	baseurl=
	enabled=1
	gpgcheck=0

yum list
yum remove all
yum makecache
```
3. 调整逻辑卷大小
```shell
vgs
lvs
lvetend -rL 200M /dev/rhcsa/rhel
df -f //检查
```
4. 创建交换分区 
```shell
fdisk -l
fdisk /dev/vdb
mkswap /dev/vdb1
vim /etc/fstab
	/dev/vdb1 swap swap defaults 0 0
swapon -a
swapon -s
free -m //检查
```
5. 创建逻辑卷
```shell

```
6. 

 


 ![[../../../../A-Accessories-Note附件/RHCSA习题及参考.pdf]]
