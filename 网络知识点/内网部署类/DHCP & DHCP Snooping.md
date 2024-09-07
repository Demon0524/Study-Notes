#ComputerNetwork
## 4、DHCP & DHCP Snooping

### 4.1、DHCP
>[什么是DHCP？为什么要用DHCP？- 华为](https://info.support.huawei.com/info-finder/encyclopedia/zh/DHCP.html)

> DHCP（Dynamic Host Configuration Protocol）动态主机配置协议

- DHCP是一种网络管理协议，运行在应用层，用于集中对用户IP地址进行动态管理和配置

> 优势

- **准确的IP配置**：IP地址配置参数必须准确；使用DHCP服务器可以最大程度地降低IP地址配置出错的风险
- **减少IP地址冲突**：每个连接的设备都必须有一个IP地址。DHCP的使用可以确保每个地址仅使用一次
- **IP地址管理的自动化**：DHCP允许将其自动化和集中化管理；
- **高效的变更管理**：DHCP的使用使更改地址、范围和端点变得非常简单

#### 4.1.1、DHCP是怎样工作的

- [[../../报文格式地址地图/应用层/DHCP报文格式]]

> DHCP协议采用 **UDP** 作为传输协议
>
> 客户端使用68号端口；服务器端使用67号端口

- 只有客户端与服务端在同一网段下才能接收客户端广播的DHCP Discover报文。
- 当客户端与服务端不在同一网段时，必须部署DHCP中继来转发两者之间的报文。在客户端看来，DHCP中继就像是服务器；在服务器看来，DHCP中继就像是客户端。



##### 一、无中继场景时DHCP客户端首次接入网络的工作原理

![无中继场景时DHCP客户端首次接入网络的报文交互示意图](https://download.huawei.com/mdl/image/download?uuid=8816db1db82148bb98166b2d3509dc47)

1. **发现阶段**

- 首先接入网络的客户端不知道服务器的IP地址，客户端通过发送DHCP Discover^[DHCP DISCOVER报文中携带了客户端的MAC地址（chaddr字段）、需要请求的参数列表选项（Option55）、广播标志位（flags字段）等信息。]报文（目的IP地址为255.255.255.255）给同一网段内的所有设备（包括DHCP服务器或中继）。



2. **提供阶段**

- 与客户端位于同一网段的服务器都会接收到DHCP Discover报文，服务器选择接收DHCP Discover报文接口的IP地址处于同一网段的地址池，并且从中选择一个可用的IP地址，通过DHCP Offer报文发送给客户端
- 通常，服务器的地址池中会指定IP地址的租期，如果客户端发送的DHCP Discover报文中携带了期望的租期，服务器会将客户端青雀的期望租期与其指定的进行比较，**选择其中时间较短的租期**分配给客户端。DHCP服务器在地址池中为客户端分配IP地址的顺序如下：
  - 服务器上已配置的与客户端MAC地址静态绑定的IP地址
  - 客户端发送的DHCP Discover报文中Option50（请求IP地址选项）指定的地址
  - 地址池内查找“Expired”状态的IP地址，即曾经分配给客户端的超租期的地址
  - 在地址池内随机查找一个“Idle”状态的IP地址
  - 如果未找到可提供分配的IP地址，则地址池依次自动回收超过租期（“Expired”状态）和处于冲突状态（“Conflict”状态）的IP地址。回收后如果找到可用的IP地址，则进行分配；否则，客户端等待回应超时，重新发送DHCP Discover报文来事情IP地址。
- 设备支持在地址池中排出一些不能通过DHCP机制进行分配的IP地址。需要将手工配置的IP地址从DHCP池中排除掉，否则会导致地址冲突
- 为了防止分配出去的IP地址出现冲突，服务器在发送DHCP Offer报文前会通过发送源地址为服务器IP地址、目的地址为预分配出去的IP地址的ICMP Echo Request报文对预分配IP地址进行地址冲突探测。
  - 指定时间内未收到应答报文，不是该网络中没有客户端使用该IP地址
  - 收到应答报文，表示该网络中存在此IP地址的客户端，将改地址列为冲突地址；然后等待重新接收DHCP Discover报文重新分配

- DHCP Offer报文发送给客户端等待16s后，如果没有收到客户端的响应，此地址可以继续分配给其他客户端



3. **选择阶段**

- 如果多个服务器想客户端回应DHCP Offer报文，客户端一般只接受第一个收到的DHCP Offer报文，然后通过广播的方式发送DHCP Request报文

- 客户端广播发送DHCP Request报文通知所有的服务器，其会选择一个服务器提供IP地址



4. **确认阶段**

- 服务器收到客户端发送的DHCP Request报文后，服务器回应DHCP Ack报文，表示DHCP Request报文中请求的IP地址分配给客户端使用
- 客户端收到DHCP ACK报文，会广播发送免费ARP报文，探测本网段是否有其他终端使用服务器分配的IP地址
  - 如果在指定时间内没有收到回应，表示客户端可以使用此地址。
  - 如果收到了回应，说明有其他终端使用了此地址，客户端会向服务器发送DHCP DECLINE报文，并重新向服务器请求IP地址，同时，服务器会将此地址列为冲突地址。当服务器没有空闲地址可分配时，再选择冲突地址进行分配，尽量减少分配出去的地址冲突。

- 当服务器收到客户端发送的DHCP REQUEST报文后，如果服务器由于某些原因（例如协商出错或者由于发送REQUEST过慢导致服务器已经把此地址分配给其他客户端）无法分配DHCP REQUEST报文中Option50填充的IP地址，则发送DHCP NAK报文作为应答，通知客户端无法分配此IP地址。客户端需要重新发送DHCP DISCOVER报文来申请新的IP地址。



##### 二、有中继场景时DHCP客户端首次接入网络的工作原理

- 首次接入DHCP网络的客户端与服务器的==工作原理与上文相同==。主要差异是DHCP中继在服务器和客户端之间转发的DHCP报文，以保证服务器和客户端可以正常交互

![有中继场景时DHCP客户端首次接入网络的工作原理](https://download.huawei.com/mdl/image/download?uuid=df77e017b2214850a724550ce0b2c4fd)

- 

1. **发现阶段**

- 中继接收到客户端广播发送的DHCP Discover报文后，进行如下处理
  - 检查DHCP报文中的hops字段。如果大于16，则丢弃报文；否则将hops字段加1（表示进过一次DHCP中继），并继续以下操作
  - 检查DHCP报文中的giaddr字段。如果是0，将gaiddr字段设置为接收DHCP Discover报文的接口IP地址；如果不是0，则不修改该字段，继续下面的操作
  - 将DHCP报文的目的IP地址改为服务器或下一跳中继的IP地址，源地址改为中继连接客户端的接口地址，通过路由转发将DHCP报文单播发送到服务器或下一条中继
- 如果客户端与服务器之间存在多个中继，后面的中继接收DHCP Discover报文的处理流程同上所述



2. **提供阶段**

- 服务器接收到DHCP Discover报文后，选择与报文中gaiddr字段为同一网段的的地址池，并为客户端分配IP地址等参数，然后向gaiddr字段标识的DHCP中继单播发送DHCP Offer报文
  - 检查报文中的gaiddr字段，如果不是接口的地址，则丢弃；否则进行下面的操作
  - 中继检查报文的广播标志位。如果广播标志位为1，则将DHCP Offer报文广播发送到客户端；否则将DHCP Offer报文单播发送给客户端



3. **选择阶段**

- 中继接收来自客户端的DHCP Request报文的过程同无中继场景下的选择阶段。



4. **确认阶段**

- 中继接收到来自服务器的DHCP ACK报文的处理过程同无中继场景下的确认阶段。

 

##### 三、DHCP客户端重用曾经使用过的地址的工作原理

- 客户端非首次接入网络时，可以重用曾经使用过的地址

![DHCP客户端重用曾经使用过的地址的工作原理](https://download.huawei.com/mdl/image/download?uuid=e84dcf9333d942209dc7f5a95c235f52)

1. 选择阶段

- 客户端广播发送包含前一次分配的IP地址的DHCP REQUEST报文，报文中的Option50（请求的IP地址选项）字段填入曾经使用过的IP地址。

2. 确认阶段

- DHCP服务器收到DHCP REQUEST报文后，根据DHCP REQUEST报文中携带的MAC地址来查找有没有相应的租约记录，如果有则返回DHCP ACK报文，通知客户端可以继续使用这个IP地址。否则，保持沉默，等待客户端重新发送DHCP DISCOVER报文请求新的IP地址。

##### 四、DHCP客户端更新租期的工作原理

- 服务器采用动态分配机制给客户端分配IP地址时，分配出去的IP地址有租期限制。

- 客户端向服务器申请地址时可以携带期望租期，服务器在分配租期时把客户端期望租期和地址池中租期配置比较，分配其中一个较短的租期给客户端。

- 租期到期或者客户端下线释放地址后，服务器会收回该IP地址，收回的IP地址可以继续分配给其他客户端使用。这种机制可以提高IP地址的利用率，避免客户端下线后IP地址继续被占用。如果客户端希望继续使用该地址，需要更新IP地址的租期（如延长IP地址租期）。

  DHCP客户端更新租期的过程如下图所示

![DHCP客户端更新租期示意图](https://download.huawei.com/mdl/image/download?uuid=83fc3a71b976489b8f74e7d725248d95)

- 当租期达到50%（T1）时，DHCP客户端会自动以单播的方式向服务器发送DHCP REQUEST报文，请求更新IP地址租期。如果收到服务器回应的DHCP ACK报文，则租期更新成功（即租期从0开始计算）；如果收到DHCP NAK报文，则重新发送DHCP DISCOVER报文请求新的IP地址。

- 当租期达到87.5%（T2）时，如果仍未收到服务器的应答，客户端会自动以广播的方式向服务器发送DHCP REQUEST报文，请求更新IP地址租期。如果收到服务器回应的DHCP ACK报文，则租期更新成功（即租期从0开始计算）；如果收到DHCP NAK报文，则重新发送DHCP DISCOVER报文请求新的IP地址。

- 如果租期时间到时都没有收到服务器的回应，客户端停止使用此IP地址，重新发送DHCP DISCOVER报文请求新的IP地址。

  客户端在租期时间到之前，如果用户不想使用分配的IP地址（例如客户端网络位置需要变更），会触发客户端向服务器发送DHCP RELEASE报文，通知服务器释放IP地址的租期。

  服务器会保留这个客户端的配置信息，将IP地址列为曾经分配过的IP地址中，以便后续重新分配给该客户端或其他客户端。

  客户端可以通过发送DHCP INFORM报文向服务器请求更新配置信息。

#### 4.1.2、DHCP的使用场景

> DHCP提供了两种地址分配机制，网络管理员可以根据网络需求为不同的主机选择不同的分配策略。

- **动态分配机制：**通过DHCP为主机分配一个有使用期限的IP地址。

  DHCP使用了租期的概念，或称为设备IP地址的有效期。租用时间是不定的，主要取决于用户在某地连接Internet需要多久，这种分配机制适用于主机需要临时接入网络或者空闲地址数小于网络主机总数且主机不需要永久连接网络的场景。

- **静态分配机制：**网络管理员通过DHCP为指定的主机分配固定的IP地址。

  相比手工静态配置IP地址，通过DHCP方式静态分配机制避免人工配置发生错误，方便管理员统一维护管理。



### 4.2、DHCP Snooping

> [什么是DHCP Snooping？](https://info.support.huawei.com/info-finder/encyclopedia/zh/DHCP+Snooping.html)

> DHCP Snooping是DHCP的一种安全特性，用于保证DHCP客户端从合法的DHCP服务器获取IP地址，并记录DHCP客户端IP地址与MAC地址等参数的对应关系
>
> DHCP Snooping可以抵御网络中针对DHCP的各种攻击，为用户提供更安全的网络环境和更稳定的网络服务。

#### 4.2.1 为什么需要DHCP Snooping？

- 目前DHCP协议（RFC2131）在应用的过程中遇到很多安全方面的问题，网络中存在一些针对DHCP的攻击，如DHCP Server仿冒者攻击、DHCP Server的拒绝服务攻击、仿冒DHCP报文攻击等。为了保证网络通信业务的安全性，引入DHCP Snooping技术。在DHCP Client和DHCP Server之间建立一道防火墙，以抵御网络中针对DHCP的各种攻击。



#### 4.2.2、DHCP Snooping应用场景有哪些？

##### 一、防止DHCP Server仿冒者攻击导致用户获取到错误的IP地址和网络参数

> 攻击原理：

- Server与Client之间还有认证机制，网络中随意添加一台DHCP Server就能够为客户端分配其他的IP地址。DHCP Discover报文是以广播的形式发送的，无论合不合法的Server都能够接收Client发送的报文
- Server仿冒者回应给Client仿冒信息，Clien将无法接收准确的IP地址等消息，导致客户端无法正常访问网络或信息安全收到严重威胁

> 解决方法：

- 配置端口在“信任（Trusted）/非信任（Untrusted）”工作模式
- 将域合法Server直接或间接连接的接口设置为信任接口，其他接口设置为非信任接口。



##### 二、防止非DHCP用户攻击导致合法用户无法正常使用网络

> 攻击原理：

- DHCP网络中，静态获取IP地址的用户（非DHCP用户）对网络可能存在多种攻击
  - 仿冒DHCP Server
  - 构建虚假DHCP Request报文

> 解决方法：

- 开启设备根据DHCP Snooping绑定表生成接口的静态MAC表项功能
- 之后，将根据接口下所有的DHCP用户对应的DHCP Snooping绑定表项自动执行命令生成这些用户的静态MAC表项，并同时关闭接口学习动态MAC表项的能力。此时，只有源MAC与静态MAC表项匹配的报文才能够通过该接口，否则报文会被丢弃
- 因此对于该接口下的非DHCP用户，只有管理员手动配置了此类用户的静态MAC表项其报文才能通过，否则报文将被丢弃
- 动态MAC表项是设备自动学习并生成的，静态MAC表项则是根据命令配置而成的。MAC表项中包含用户的MAC、所属VLAN、连接的接口号等信息，设备可根据MAC表项对报文进行二层转发。



##### 三、防止DHCP报文泛洪攻击导致设备无法正常工作

> 攻击原理：

- 在DHCP网络环境中，若攻击者短时间内向设备发送大量的DHCP报文，将会对设备的性能造成巨大的冲击以致可能会导致设备无法正常工作。

> 解决方法：

- 开启DHCP Snooping，设备将会检测DHCP报文的上送速率，并仅允许咋规定的速率内的报文上送至DHCP报文处理单元



##### 四、防止仿冒DHCP报文攻击导致合法用户无法获得IP地址或异常下线

> 攻击原理：

- 若攻击者冒充合法用户不断向Server发送DHCP Request报文来续租IP地址，会导致这些到期的IP地址无法正常回收，导致一些合法用户不能获取IP地址
- 若攻击者仿冒合法用户的DHCP Release报文发往Server，将会导致用户异常下线

> 解决方法：

- 利用DHCP Snooping绑定表的功功能
- 设备将通过DHCP Request续租报文和DHCP Release报文与绑定表进行匹配操作，能够有效的判别报文是否合法
  - 主要检查报文中的VLAN、IP、MAC、接口信息是否匹配动态绑定表



##### 五、防止DHCP Server服务拒绝攻击导致部分用户无法上线

> 攻击原理：

- 设备接口下存在大量攻击者恶意申请IP地址，导致DHCP Server中IP地址快速耗尽而不能为其他合法用户提供IP地址分配服务
- Server通常仅根据DHCP Request报文中的额Chaddr（Client Hardware Address）字段来确认客户端的MAC地址。如果每一个攻击者通过不断改变Chaddr字段向Server申请IP地址，同样会导致Server的IP地址被耗尽

>解决方法：

- 启用DHCP Snooping功能，可配置设备或接口允许的最大DHCP用户数
- 可使用设备检测DHCP Request报文帧头MAC与DHCP数据区中Chaddr字段是否一致功能；若MAC地址与Chaddr值相同则转发，否则丢弃



#### 4.2.3、DHCP Snooping是如何工作的？

- IPv4与IPv6原理相似，一下仅针对IPv4进行描述

> 使用了DHCP Snooping的设备将用户DHCP的DHCP请求报文通过信任接口发送给合法的DHCP服务器。
>
> 之后设备根据DHCP服务器回应的DHCP ACK报文信息生成DHCP Snooping绑定表。
>
> 后续设备再从使用了DHCP Snooping的接口接收用户发来的DHCP报文时，会进行匹配检查，能够有效防范非法用户的攻击。

##### 一、DHCP Snooping信任功能

> DHCP Snooping的信任功能，能够保证客户端从合法额的服务器获取IP（Internet Protocol）地址

- DHCP Snooping信任功能可以控制DHCP服务器应答报文的来源，以防止之网络中可能存在的DHCP Server仿冒者为DHCP Client分配IP地址及其他配置信息
- DHCP Snooping信任功能将接口分为信任接口和非信任接口：
  - **信任接口：**正常接收DHCP Server响应的DHCP Ack、DHCP Nak和DHCP Offer报文
  - **非信任接口：**正常接收DHCP Server响应的DHCP Ack、DHCP Nak和DHCP Offer报文后，**丢弃该报文**

- 一般将与合法DHCP服务器直接或间接连接的接口设置为信任接口，其他接口设置为非信任接口



##### 二、DHCP Snooping绑定表

- PC作为DHCP客户端通过广播形式发送DHCP请求报文，使用了DHCP Snooping功能的二层接入设备将其通过信任接口转发给DHCP服务器。
- 最后DHCP服务器将含有IP地址信息的DHCP ACK报文通过单播的方式发送给PC。
- 在这个过程中，二层接入设备收到DHCP ACK报文后，会从该报文中提取关键信息（包括PC的MAC地址以及获取到的IP地址、地址租期），并获取与PC连接的使用了DHCP Snooping功能的接口信息（包括接口编号及该接口所属的VLAN），根据这些信息生成DHCP Snooping绑定表。



DHCP Snooping绑定表功能示意图

![DHCP Snooping绑定表功能示意图](https://download.huawei.com/mdl/image/download?uuid=1b3fd3a8e1fe41ad8d2c6b84b1d2e274)

- DHCP Snooping绑定表根据DHCP租期进行老化或根据用户释放IP地址时发出的DHCP Release报文自动删除对应表项。
- 由于DHCP Snooping绑定表记录了DHCP客户端IP地址与MAC地址等参数的对应关系，故通过对报文与DHCP Snooping绑定表进行匹配检查，能够有效防范非法用户的攻击。
- 为了保证设备在生成DHCP Snooping绑定表时能够获取到用户MAC等参数，**DHCP Snooping功能需应用于二层网络中的接入设备或第一个DHCP Relay【中继】上**。
- 在DHCP中继使用DHCP Snooping场景中，**DHCP Relay设备不需要设置信任接口**。因为DHCP Relay收到DHCP请求报文后进行源目的IP、MAC转换处理，然后以单播形式发送给指定的合法DHCP服务器，所以DHCP Relay收到的DHCP ACK报文都是合法的，生成的DHCP Snooping绑定表也是正确的。



































