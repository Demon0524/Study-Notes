#ComputerNetwork
[TOC]

# 一、什么是OSPF

> OSPF（Open Shortest Path Frist）-- 最短路径优先

- OSPFv2 - 针对==**IPv4网络**==
- OSPFv3 - 针对==IPv6网络==



# 二、为什么需要OSPF
> LSA - 链路状态信息

包含：
- 接口信息
- IP地址
- 掩码
- cost值

邻居关系

1. 建立邻居关系
	1. 发送 Hello 报文
		1. 心跳机制
		2. 组播
			1. 周期 10s
			2. 224.0.0.5
		3. 打招呼
		4. RID - 不能重复
	2. 修改名字，重启邻居关系
	3. 最大物理地址
	4. 最大环回口IP地址
	5. 手工配置 Ipv4格式
同步LSA，建立 LSDB - 链路状态信息库