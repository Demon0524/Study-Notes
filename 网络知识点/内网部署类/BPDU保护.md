#ComputerNetwork
## 3、BPDU保护
>[华为BPDU讲解](https://support.huawei.com/enterprise/zh/doc/EDOC1100090440)
>定义：避免边缘端口受到BPDU报文攻击导致网络拓扑改变及业务流量中断

- 在二层网络中，运行[[../../报文格式地址地图/数据链路层/生成树协议]]（STP/RSTP/MSTP/VBST）的交换机之间通过交互BPDU报文进行生成树的计算，将环形网络修剪成无环路的树形拓扑。
- 攻击者会伪造BPDU恶意攻击交换机，当边缘端口接受BPDU报文时，交换机会自动将边缘端口设置为非边缘端口，并重新对生成树进行计算。当攻击者发送的BPDU报文中的桥优先级高于现有网络中根桥优先级时，会改变当前网络拓扑，可能会导致业务流量的中断。

> 配置命令：

```shell
<HUAWEI> system-view
[HUAWEI] stp bpdu-protection
```