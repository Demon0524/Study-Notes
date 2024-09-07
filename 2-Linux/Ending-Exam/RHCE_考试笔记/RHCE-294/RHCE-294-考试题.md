#Linux #Ending_Exam #RHCE #Practical_exams
# RHCE-249 考试题

### 重要配置信息

在考试期间，除了您就坐位置的台式机之外，还将使用多个虚拟系统。您不具有台式机系统的 root 访问权，但具有对虚拟系统的完整 root 访问权。
#### 系统信息

在本考试期间，您将操作下列虚拟系统：
|  系统   |    IP 地址     |     Ansible 角色     |
| :-----: | :------------: | :------------------: |
| control | 172.25.250.254 | ansible control node |
|  node1  |  172.25.250.9  | ansible managed node |
|  node2  | 172.25.250.10  | ansible managed node |
|  node3  | 172.25.250.11  | ansible managed node |
|  node4  | 172.25.250.12  | ansible managed node |
|  node5  | 172.25.250.13  | ansible managed node |

这些系统的 IP 地址采用静态设置。请勿更改这些设置。
主机名称解析已配置为解析上方列出的完全限定主机名，同时也解析主机短名称。

#### 帐户信息

所有系统的 root 密码是 ***flectrag***。

请勿更改 root 密码。除非另有指定，否则这将是用于访问其他系统和服务的密码。此外，除非另有指定，否则此密码也应用于您创建的所有帐户，或者任何需要设置密码的服务。

为方便起见，所有系统上已预装了 SSH 密钥，允许在不输入密码的前提下通过 SSH 进行 root 访问。请勿对系统上的 root SSH 配置文件进行任何修改。

Ansible 控制节点上已创建了用户帐户 greg。此帐户预装了 SSH 密钥，允许在 Ansible 控制节点和各个 Ansible 受管节点之间进行 SSH 登录。请勿对系统上的 greg SSH 配置文件进行任何修改。您可以从 root 帐户使用 su 访问此用户帐户。


#### 重要信息

除非另有指定，否则您的所有工作（包括 Ansible playbook、配置文件和主机清单等）应当保存在控制节点上的目录 **/home/greg/ansible** 中，并且应当归 **greg** 用户所有。所有 Ansible 相关的命令应当由 greg 用户从 Ansible 控制节点上的这个目录运行。


#### 其他信息

一些考试项目可能需要修改 Ansible 主机清单。您要负责确保所有以前的清单组和项目保留下来，与任何其他更改共存。您还要有确保清单中所有默认的组和主机保留您进行的任何更改。

***考试系统上的防火墙默认为不启用，SELinux 则处于强制模式。***

如果需要安装其他软件，您的物理系统和 Ansible 控制节点可能已设置为指向 content 上的下述存储库：
http://content/rhel8.0/x86_64/dvd/BaseOS
http://content/rhel8.0/x86_64/dvd/AppStream

一些项目需要额外的文件，这些文件已在以下位置提供：
http://materials

产品文档可从以下位置找到：
http://materials/docs

其他资源也进行了配置，供您在考试期间使用。关于这些资源的具体信息将在需要这些资源的项目中提供。

##### 重要信息
请注意，在评分之前，您的 Ansible 受管节点系统将重置为考试开始时的初始状态，您编写的 Ansible playbook 将通过以 ***greg*** 用户身份从控制节点上的目录 ***/home/greg/ansible*** 目录运行来应用。在 playbook 运行后，系统会对您的受管节点进行评估，以判断它们是否按照规定进行了配置。


### 安装和配置 Ansible

按照下方所述，在控制节点 **control** 上安装和配置 Ansible：
- [ ]  安装所需的软件包
- [ ] 创建名为 **/home/greg/ansible/inventory** 的静态清单文件，以满足以下要求：
	- [ ] **node1** 是 **dev** 主机组的成员
	- [ ] **node2** 是 **test** 主机组的成员
	- [ ] **node3** 和 **node4** 是 **prod** 主机组的成员
	- [ ] **node5** 是 **balancers** 主机组的成员
	- [ ] **prod** 组是 **webservers** 主机组的成员
- [ ] 创建名为 **/home/greg/ansible/ansible.cfg** 的配置文件，以满足以下要求：
	- [ ] 主机清单文件为 **/home/greg/ansible/inventory**
	- [ ] playbook 中使用的角色的位置包括 **/home/greg/ansible/roles**

### 创建和运行 Ansible 临时命令
作为系统管理员，您需要在受管节点上安装软件。
请按照正文所述，创建一个名为 **/home/greg/ansible/adhoc.sh** 的 shell 脚本，该脚本将使用 Ansible 临时命令在各个受管节点上安装 yum 存储库：
- [ ] 存储库1：
	- [ ] 存储库的名称为 **EX294_BASE**
	- [ ] 描述为 **EX294 base software**
	- [ ] 基础 URL 为 **http://content/rhel8.0/x86_64/dvd/BaseOS**
	- [ ] GPG 签名检查为**启用状态**
	- [ ] GPG 密钥 URL 为 **http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release**
	- [ ] 存储库为**启用状态**
- [ ] 存储库2：
	- [ ] 存储库的名称为 **EX294_STREAM**
	- [ ] 描述为 **EX294 stream software**
	- [ ] 基础 URL 为 **http://content/rhel8.0/x86_64/dvd/AppStream**
	- [ ] GPG 签名检查为**启用状态**
	- [ ] GPG 密钥 URL 为 **http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release**
	- [ ] 存储库为**启用状态**

### 安装软件包
创建一个名为 **/home/greg/ansible/packages.yml** 的 playbook :
- [ ] 将 **php** 和 **mariadb** 软件包安装到 **dev**、**test** 和 **prod** 主机组中的主机上
- [ ] 将 **RPM Development Tools** 软件包组安装到 **dev** 主机组中的主机上
- [ ] 将 **dev** 主机组中主机上的**所有软件包更新为最新版本**

 
### 使用 RHEL 系统角色

  
安装 RHEL 系统角色软件包，并创建符合以下条件的 playbook **/home/greg/ansible/timesync.yml** ：
- [ ] 在**所有受管节点**上运行
- [ ] 使用 **timesync** 角色
- [ ] 配置该角色，以使用当前有效的 NTP 提供商
- [ ] 配置该角色，以使用时间服务器 **203.107.6.88**
- [ ] 配置该角色，以启用 **iburst** 参数

### 使用 Ansible Galaxy 安装角色

使用 Ansible Galaxy 和要求文件 **/home/greg/ansible/roles/requirements.yml** 。从以下 URL 下载角色并安装到 **/home/greg/ansible/roles** ：
- [ ] **http://materials/haproxy.tar** 此角色的名称应当为 **balancer**
- [ ] **http://materials/phpinfo.tar** 此角色的名称应当为 **phpinfo**

 ### 创建和使用角色

 根据下列要求，在 **/home/greg/ansible/roles** 中创建名为 **apache** 的角色：
- [ ] httpd 软件包已安装，设为在**系统启动时启用**并**启动**
- [ ] **防火墙**已启用并正在运行，并使用允许访问 **Web** 服务器的规则
- [ ] 模板文件 **index.html.j2** 已存在，用于创建具有以下输出的文件 **/var/www/html/index.html** ：
  `Welcome to HOSTNAME on IPADDRESS`
	其中，HOSTNAME 是受管节点的**完全限定域名**，**IPADDRESS** 则是受管节点的 IP 地址。

 

### 从 Ansible Galaxy 使用角色

根据下列要求，创建一个名为 **/home/greg/ansible/roles.yml** 的 playbook ：
- [ ] playbook 中包含一个 play， 该 play 在 **balancers** 主机组中的主机上运行并将使用 **balancer** 角色。
	- [ ] 此角色配置一项服务，以在 **webservers** 主机组中的主机之间平衡 Web 服务器请求的负载。
	- [ ]  浏览到 **balancers** 主机组中的主机（例如 **http://172.25.250.13** ）将生成以下输出：
	      `Welcome to serverb.lab.example.com on 172.25.250.11`
	- [ ] 重新加载浏览器将从另一 Web 服务器生成输出：
		 `Welcome to serverc.lab.example.com on 172.25.250.12`
- [ ] playbook 中包含一个 play， 该 play 在 **webservers** 主机组中的主机上运行并将使用 **phpinfo** 角色。
	- [ ] 请通过 URL **/hello.php** 浏览到 **webservers** 主机组中的主机将生成以下输出：
	      `Hello PHP World from FQDN`
	- [ ] 其中，FQDN 是主机的完全限定名称。
	      `Hello PHP World from serverb.lab.example.com` 
另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。
- [ ] 同样，浏览到 **http://172.25.250.12/hello.php** 会生成以下输出：
`Hello PHP World from serverc.lab.example.com`
另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。

 
### 创建和使用逻辑卷

创建一个名为 **/home/greg/ansible/lv.yml** 的 playbook ，它将在所有受管节点上运行以执行下列任务：
- [ ] 创建符合以下要求的逻辑卷：
	- [ ] 逻辑卷创建在 **research** 卷组中
	- [ ] 逻辑卷名称为 **data**
	- [ ] 逻辑卷大小为 **1500 MiB**
- [ ] 使用 **ext4** 文件系统格式化逻辑卷
- [ ] 如果无法创建请求的逻辑卷大小，应显示错误信息，并且应改为使用大小 **800 MiB**。
      `Could not create logical volume of that size`	    
- [ ] 如果卷组 **research** 不存在，应显示错误信息
      `Volume group done not exist`
- [ ] 不要以任何方式挂载逻辑卷

 

### 生成主机文件

- [ ] 将一个初始模板文件从 **http://materials/hosts.j2** 下载到 **/home/greg/ansibl**
- [ ] 完成该模板，以便用它生成以下文件：针对每个清单主机包含一行内容，其格式与 /etc/hosts 相同
- [ ] 创建名为 **/home/greg/ansible/hosts.yml** 的 playbook ，它将使用此模板在 dev 主机组中的主机上生成文件 **/etc/myhosts**  
该 playbook 运行后， **dev** 主机组中主机上的文件 **/etc/myhosts** 应针对每个受管主机包含一行内容：
```
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6

172.25.250.9    node1.lab.example.com node1
172.25.250.10   node2.lab.example.com node2
172.25.250.11   node3.lab.example.com node3
172.25.250.12   node4.lab.example.com node4
172.25.250.13   node5.lab.example.com node5  
```
 **注：清单主机名称的显示顺序不重要。**

 
### 修改文件内容
按照下方所述，创建一个名为 **/home/greg/ansible/issue.yml** 的 playbook ：
- [ ] 该 playbook 将在**所有清单主机**上运行
- [ ] 该 playbook 会将 **/etc/issue** 的内容替换为下方所示的一行文本：
	- [ ] 在 **dev** 主机组中的主机上，这行文本显示 为：**Development**
	- [ ] 在 **test** 主机组中的主机上，这行文本显示 为：**Test**
	- [ ] 在 **prod** 主机组中的主机上，这行文本显示 为：**Production**

### 创建 Web 内容目录
按照下方所述，创建一个名为 **/home/greg/ansible/webcontent.yml** 的 playbook ：
- [ ] 该 playbook 在 **dev** 主机组中的受管节点上运行
- [ ] 创建符合下列要求的目录 **/webdev** ：
	- [ ] 所有者为 **webdev** 组
	- [ ] 具有常规权限：**owner=read+write+execute ， group=read+write+execute ，other=read+execute**
	- [ ] 具有特殊权限：设置组 ID
- [ ] 用符号链接将 **/var/www/html/webdev** 链接到 **/webdev**
- [ ] 创建文件 **/webdev/index.html** ，其中包含如下所示的单行文件： **Development**
- [ ] 在 **dev** 主机组中主机上浏览此目录（例如 **http://172.25.250.9/webdev/** ）将生成以下输出：
	`Development`

 

### 生成硬件报告
创建一个名为 **/home/greg/ansible/hwreport.yml** 的 playbook ，它将在所有受管节点上生成含有以下信息的输出文件 **/root/hwreport.txt** ：
- [ ] **清单主机名称**
- [ ] 以 **MB** 表示的**总内存大小**
- [ ] **BIOS 版本**
- [ ] 磁盘设备 **vda 的大小**
- [ ] 磁盘设备 **vdb 的大小**
- [ ] 输出文件中的每一行含有一个 key=value 对。

您的 playbook 应当：
- [ ] 从 **http://materials/hwreport.empty** 下载文件，并将它保存为 **/root/hwreport.txt**
- [ ] 使用**正确的值**改为 /root/hwreport.txt
- [ ] 如果硬件项不存在，相关的值应设为 **NONE**

 
### 创建密码库

按照下方所述，创建一个 Ansible 库来存储用户密码：
- [ ] 库名称为 /home/greg/ansible/locker.yml
- [ ] 库中含有两个变量，名称如下：
	- [ ] **pw_developer**，值为 **Imadev**
	- [ ] **pw_manager**，值为 **Imamgr**
- [ ] 用于加密和解密该库的密码为 **whenyouwishuponastar**
- [ ] 密码存储在文件 **/home/greg/ansible/secret.txt** 中

 
### 创建用户帐户
 
 - [ ] 从 **http://materials/user_list.yml** 下载要创建的用户的列表，并将它保存到 /home/greg/ansible
 - [ ] 在本次考试中使用在其他位置创建的密码库 **/home/greg/ansible/locker.yml** 。创建名为 **/home/greg/ansible/users.yml** 的 playbook ，从而按以下所述创建用户帐户：
	 - [ ] 职位描述为 **developer** 的用户应当：
		 - [ ] 在 **dev** 和 **test** 主机组中的受管节点上创建
		 - [ ] 从 **pw_developer** 变量分配密码
		 - [ ] 是补充组 **devops** 的成员
	 - [ ] 职位描述为 **manager** 的用户应当：
		 - [ ] 在 **prod** 主机组中的受管节点上创建
		 - [ ] 从 **pw_manager** 变量分配密码
		 - [ ] 是补充组 **opsmgr** 的成员
 - [ ] 密码采用 **SHA512** 哈希格式。
 - [ ] 您的 playbook 应能够在本次考试中使用在其他位置创建的库密码文件 **/home/greg/ansible/secret.txt** 正常运行。

 
### 更新 Ansible 库的密钥

按照下方所述，更新现有 Ansible 库的密钥：
- [ ] 从 **http://materials/salaries.yml** 下载 Ansible 库到 **/home/greg/ansible**
- [ ] 当前的库密码为 **insecure8sure**
- [ ] 新的库密码为 **bbs2you9527**
- [ ] 库使用**新密码**保持加密状态 

![[../../../../A-Accessories-Note附件/RHCE8辅导1-7.pdf]]


![[../../../../A-Accessories-Note附件/RHCE8辅导8-15.pdf]]





