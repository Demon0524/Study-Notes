##### 模版创建时间：{{date}}/{{time}}
- 多同目的源且同端口使用 `-m iptange` 插件配置连续源；注意IP需完整：`34.95.32.4-134.95.32.6`
- 使用 `-m multiport` 插件配置多个端口，使用 `,` 相隔。可使用单端口
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
iptables -I RH-Firewall-1-INPUT -m iprange --src-range 【SOURS】 -p tcp -m multiport --dports 【PORT】 -m comment --comment "allow app access" -j ACCEPT

sed -i '/ESTABLISHED/i -A RH-Firewall-1-INPUT -m iprange --src-range 【SOURS】 -p tcp -m multiport --dports 【PORT】 -m comment --comment "allow app access" -j ACCEPT' /etc/sysconfig/iptables
```

