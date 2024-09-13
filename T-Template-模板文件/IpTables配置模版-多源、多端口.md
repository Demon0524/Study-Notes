##### 模版创建时间：{{date}}/{{time}}
- 连续同目的多端口可使用 `:` 连续： `1:100`
- 非连续同目的多端口使用 `,` 相隔：`1,5,100,6565`

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
iptables -I RH-Firewall-1-INPUT -s 【SOURS】 -p tcp -m multiport --dports 【PORT】 -m comment --comment "allow app access" -j ACCEPT

sed -i '/ESTABLISHED/i -A RH-Firewall-1-INPUT -s 【SOURS】 -p tcp -m multiport --dports 【PORT】 -m comment --comment "allow app access" -j ACCEPT' /etc/sysconfig/iptables
```

