##### 模版创建时间：{{date}}/{{time}}

> SOURS：
```perl

```
> DEST：
```perl

```
> PORT：
```perl

```
> 模版：
```perl
iptables -I RH-Firewall-1-INPUT -s 【SOURS】 -p tcp --dport 【PORT】 -m comment --comment "allow app access" -j ACCEPT

sed -i '/ESTABLISHED/i -A RH-Firewall-1-INPUT -s 【SOURS】 -p tcp --dport 【PORT】 -m comment --comment "allow app access" -j ACCEPT' /etc/sysconfig/iptables
```

