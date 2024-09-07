#ComputerNetwork
# 一、MSTP
> 快速生成树协议MSTP（Multiple Spanning Tree Bridge Protocol）

- 详细信息通过[[../../报文格式地址地图/数据链路层/生成树协议]]了解

# 二、VRRP

- [[../../报文格式地址地图/网络层/VRRP报文格式]]

> 虚拟链路冗余协议VRRP（Virtual Router Redundancy Protocol）

- 是一种提高网络可靠性的容错协议
- 通过VRRP可以在主机的下一跳设备出现故障时，及时将业务切换到备份设备，保障网络通信的连续性和可靠性

## 1、工作原理

- 为了解决主机使用缺省网关通信时，出现Gateway出现故障，从而导致主机失联，业务中断

> VRRP的三种状态

-tx-
| 状态       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Initialize | VRRP不可用。设备不处理VRRP通告的任何报文                     |\
|            | 设备在启动或检测到故障时会进入此状态                         |
| Master     | ==承担虚拟路由设备的所有转发功能==，并定期向这个虚拟内发送VRRP通告报文 |
| Backup     | ==不会承担虚拟路由设备的转发工作==，并定期接受Master设备的VRRP通告报文，判断Master的工作状态是否正常 |



> VRRP选举机制

- Master设备选举过程

![Master设备选举过程](https://download.huawei.com/mdl/image/download?uuid=090b0e71d45942fcbf40641b87ddb6a3)



> 工作原理

- 当Master设备出现故障时，备份路由会选举出新的Master设备。新的Master设备考试响应对虚拟IP地址的ARP响应，并定期发送VRRP通告报文

  详细工作过程：

  - VRRP备份组中的设备根据优先级选举出Master。Master设备通过发送免费ARP报文，将虚拟MAC地址通知给与它连接的设备或主机，从而承担报文转发任务
  - Master设备周期性向备份组内所有Backup设备发送VRRP通告报文，通告其配置信息（优先级等）和工作状况
  - 如果Master设备出现故障，VRRP备份组中的Backup设备将根据优先级重新选举新的Master
  - VRRP备份组状态切换时，Master设备由一台设备切换至了一台设备，新的Master设备会立即发送携带虚拟路由器的虚拟MAC地址和虚拟IP地址信息的免费ARP报文，刷新它连接的设备或主机的MAC表项，从而把用户流量引导新的Master设备上来，整个过程对用户完全透明
  - 原Master设备故障恢复时，若该设备为IP地址拥有者（优先级为255），将直接切换至Master状态。若该设备优先级小于255，将首先切换为Backup状态，且其优先级恢复为故障前的优先级
  - Backup设备的优先级高于Master设备时，由Backup设备的工作方式（抢占方式和非抢占方式）决定是否重新选举Master

## VRRP应用场景

> 在网络中，VRRP不仅仅在设备出现故障时触发Master设备的切换，它也能感知某个接口、某条路由的状态

### 1、与接口状态联动

- VRRP可以与上行接口的状态绑定在一起，当承担转发的任务的Master设备的上行接口出现异常时，Master设备将降低一定的优先级，当优先级低于Backup设备的优先级时，Backup设备就会切换为Master设备，从而防止因为上行接口的异常导致业务受损

![VRRP与接口联动](https://download.huawei.com/mdl/image/download?uuid=e486b0827c6c4c3eba814200802f62aa)

### 2、与路由状态联动

- VRRP可以与上行的路由状态绑定在一起，当上行路由出现异常时，Master设备可以降低一定的优先级，当优先级低于Backup设备额优先级时，Backup设备就会切换为Master设备，从而防止因为上行路由的异常导致业务受损

![VRRP与路由联动](https://download.huawei.com/mdl/image/download?uuid=6ef8f7de90c64054a6e78a498384bb3a)
