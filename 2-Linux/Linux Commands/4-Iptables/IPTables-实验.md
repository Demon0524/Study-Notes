#Linux_commands #Linux_防火墙
[toc]
# 测试环境
```ini
workstation:（内网主机）
	192.168.12.130
servera:（firewall device）
	192.168.12.141
	202.111.112.148
serverb: （外网主机）
	202.111.112.149
```
- 如何实现两个不同网段的联通？
- `202.111.112.0`网段不能ping通网关
## 添加路由
```shell
# [route-网卡]，注意文件的位置和名称
vim /etc/sysconfig/network-scripts/route-ens33
# [目标网段] via [下一条地址] dev [绑定的本地网卡]
202.111.112.0/24 via 192.168.12.141 dev ens33
```
## 使用iptables服务
```shell
yum install -y iptables*
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl start iptables
```

```shell
lsmod | grep iptable
# lsmod 是查看内核中加载了那些模块
# 主要查看是否有相对应的表加载完成
vi /etc/rc.local
# 以下是需要启动的内核模块
# 防火墙是被嵌入到linux内核中的，但是默认是没有激活，需要手动激活
modprobe ip_tables
modprobe iptable_filter
modprobe iptable_nat
modprobe ip_conntrack
modprobe ip_conntrack_ftp
modprobe ip_nat_ftp
modprobe ipt_state
EOF
# modprobe 是加载内核模块
```


```shell
iptables -t filter -I INPUT -p tcp --dport 22 -j DORP
# 添加规则（在filter表里使用INPUT规则拒绝目标端口是22的TCP链接）

iptablse -nL --line-number 
# 显示规则的行号（能够显示已经存在的所有防火墙规则）

iptables -t filter -D INPUT 1
# 删除规则（删除规定行号的规则）
```

**以上提供命令都知识临时操作，永久则需要在`/etc/sysconfig/iptables`配置文件中添加链；语法与临时语法一致**

# 生产环境
> 在生产环境下配置`iptables`规则时需要在非业务期间操作，防止出错到时业务中断。

正常使用模版规则进行匹配即可。
```
iptables -I RH-Firewall-1-INPUT -p tcp --dport 【PORT】 -m comment --comment "allow app access" -j ACCEPT -s 【IP】
sed -i '/EST/i -A RH-Firewall-1-INPUT -p tcp --dport 【PORT】 -m comment --comment "allow app access" -j ACCEPT -s 【IP】' /etc/sysconfig/iptables
# 指定表
# 指定端口及端口类型
# 使用注释‘comment’
# 源IP
# 下一条则为将规则写入配置文件中
```

```shell
awk '
function chuil(ips){
	split(ipss,arr,",")
	len = length(arr)
	for( i=1; i<=len; i++){
		if( index(arr[i], "-")){
			split(arr[i], a, ".")
			split(a[4], range, "-")
			end = range[2]
			for( j=range[1]; j<=end; j++){
			   if( i== len && j == end ){
				  printf "%d.%d.%d.%d ", a[1],a[2],a[3],j
				}
				else{
				printf "%d.%d.%d.%d, ", a[1],a[2],a[3],j
				}
			} 
		}
		else {
			if (i == len ){
				printf arr[i]"
			}
			else {
			printf arr[i]","
			}
		}
	}
}
{
	chuli($1)
	chuli($2)
	gsub(",",";", $3)
	gsub("_",";", $3)
	printf $3
}
	2.txt
```
