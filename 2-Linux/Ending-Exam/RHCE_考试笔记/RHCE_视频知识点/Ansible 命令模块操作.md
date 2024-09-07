#Linux #Ending_Exam #RHCE #视频笔记
[toc]

# Ansible 命令模块操作

##  Ansible 清单

> 用于管理目标主机的列表

## 部署 Ansible 清单文件

- 定义清单
  - 清单定义 Ansible 将要管理的一批主机
  -  **这些主机也可以分配到组中，以进行集中管理。组可以包含 ==子组== ，主机也可以是多个组的成员** 。清单还可以设置应用到它所定义的主机和组的变量。
- 可以通过两种方式定义主机清单
  -  **静态** 主机清单：静态主机清单可以通过 **文本文件** 来定义。
  -  **动态** 主机清单：动态主机清单可以根据需要使用 **外部信息提供程序通过脚本或其他程序来生成** 。

### 静态清单

- 静态清单文件是指定 Ansible 目标受管主机的文本文件。可以使用多种不同的格式编写此文件，包括 ==INI 样式或 YAML== 。

  > YAML 格式检测： http://www.yamllint.com/ 

  > **注意** ： Ansible 支持多种静态清单格式。此次仅介绍 **INI** 样式的格式

#### INI 格式

- 节
  - [section]
- 参数
  - key = value
  - 需要注意的时 **key** 在 section 内通常时不能重复的，其行为是不确定的，可能会导致前者被覆盖，或直接报错；但 **key 在不同的 section 中使可以重复的** 。
- 注释
  - 注释使用分号（;）表示。在分号后米娜的文字，知道该行结尾都全部为注释。

```ini
[ServerA]
enable = 0
port = 8080
[ServerB]
enable = 1
port = 8081
; comment text
```

#### 定义嵌套组

- Ansible 主机清单可以包含 **由多个主机构成的组** 。这通过创建后缀为 **:children** 的主机组名称来实现。

```ini
; INI 格式的清单
[dev]
servera.example.com
serverb.example.com
[test]
serverc.example.com
serverd.example.com
[balance:children]
dev
test
```

```yaml
# YAML 格式的清单
all:
  hosts:
    mail.example.com: # 未知主机
  children:
    dev:
      hosts:
        servera.example.com:
        serverb.example.com:
    test:
      hosts:
        serverc.example.com:
        serverd.example.com:
```



#### 静态清单示例

```INI
# 未定义组的主机
servera.lab.example.com
serverb.lab.example.com
serverc.lab.example.com
# 定义组的主机
[webservers]
web1.example.com
web2.example.com
192.168.2.42
# 可以通过主机名，也可以通过IP地址定义
[db-servers]
db1.example.com
db2.example.com
[balance:children]
websevers
db-servers
```

#### 通过规范简化主机规格

- 可以通过指定主机名称或 IP 地址的范围来简化 Ansible 主机清单。可以 **指定数字或字母的范围** 。
  - **[START:END]** 
- 范围匹配从 START 到 END(含) 的所有的值。
  -  **192.168.[4:7].[0:255]** 匹配 192.168.4.0/22 网络中的所有 IPv4 地址
  -  **server[01:20].examlpe.com** 匹配名为 server01.example.com - server20.example.com 的所有主机
  -  **[a:c].dns.example.com** 
  -  **2001:db8:[a:f]** 

#### 验证清单

> ansible [inventory_hostname] --list-hosts

-  **重要** 
  - 如果清单中含有名称相同的主机和主机组， ansible 命令将显示警告并以主机作为其目标、主机组则被忽略。
  - 两个主机组始终存在 all 主机组含邮费清单中明确列出的每一个主机。 ungrouped 主机组含有清单中明确列出、但不属于任何其他组的没一个主机。

#### 覆盖清单的位置

-  ==**/etc/ansible/hosts**== 文件被视为系统的 **默认静态清单文件** 。不过， **通常的做法是不是用该文件** ，而是在 Ansible 配置文件中为清单文件 **定义一个不同位置** 。
- 可以通过 ==**--inventory PATHNAME**== 或==**-i PATHNAME**== 选项命令行中指定清单文件的位置，其中 PATHNAME 是所需清单文件的路径。
-  **在清单中定义的变量** 
  - 可以在主机清单文件中 **指定 palybook 使用的变量值** 。这些变量仅应用到主机或主机组。通常， **最好在特殊目录中定义这些库存变量，而不要直接在清单文件中定义** 。

#### 在清单中定义变量

- **可以在主机清单中指定 palybook 使用的变量值。这些变量经应用到特定的主机或主机组。通常，最好在特殊目录中定义这些库存变量，而不要直接在清单文件中定义** 。

  - 主机变量

```ini
; INI 格式
[atlanta]
host1 http_port=80 maxRequstsPerChild=8080
host1 http_port=303 maxRequstsPerChild=9090
```
```yaml
# YAML 格式
atlanta:
  hosts；
	host1：
	  http_port: 80
	  maxRequstsPerChild: 8080
   host2：
	  http_port: 303
	  maxRequstsPerChild: 9090
```

 
### Ansible 配置文件

> 包括一个名为 **ansible.cfg** 的 **INI 格式文件、环境变量、命令行选项、剧本关键字和变量** 。

- Ansible 提供了 **四个来源控制它的行为** 。按照优先级从 **最低（最容易覆盖）** 到 **最高（覆盖所有其他）** 的顺序

  -  **Configuration settings** 
  -  **Command-line options** 
  -  **Playbook keywords** 
  -  **Variables** 

- 可以在配置文件中进行更改和使用更改，该配置文件将按照一下顺序进行搜索：

  -  **ANSIBLE_CONFIG** (environment variable if set)
  -  ansible.cfg (in the home directory)
  -  ~/.ansible.cfg (in the home directory)
  -  /etc/ansible/ansible.cfg

- 配置文件是 INI 格式的一种变体。当注释开始一行时， hash号(#) 和分号(;) 都可以 **作为注释标记** 。但是如果注释与常规值内联，作为只允许分号引入注释。

```ini
# some basic default values...
inventory = /etc/ansible/hosts ;This points to the file that lisrs your hosts
```

#### 运行临时命令

##### ad-hoc 使用场景

> - 其更注重于 **简单** 或平时工作 **临时遇到的任务** ，相当于 Linux 系统命令行下的 **Shell 命令**
>
> -  **Ansible-playbook** 更适用于解决 **复杂或需固化下来的任务** ，相当于 Linux 系统的 Shell Script。

-  **无需编写 Play-book 即可运行** 
- 临时命令正是 ==**快速执行简单任务时所需要的工具**== 

#### 通过模块来执行任务

> 通过 `ansible-doc-l` 命令查询系统上安装的所有模块

- Ansible **提供了数百个能够完成不同任务的模块** 

### Ansible 模块

| 模块类别 |                 模块                 |
| :------: | :----------------------------------: |
| **文件模块** |**copy** ：将本地文件复制到受管理主机；  **file** ：设置文件的权限和其他属性；   **lineinfile** ：确保特定行是否在文件中；  **synchronize** ：使用 **sync** 同步内容 |
| **软件包模块** | **package** ：使用操作系统本机的自动检测软件包管理器管理软件包； **yum** ：使用 YUM 软件包管理器管理软件包； **apt** ：使用 APT 软件包管理器管理软件包； **dnf** ：使用 DNF 软件包管理器管理软件包； **gem** ：管理 Rudy gem； **pip** ：从 PypI 管理 Python 软件包 |
| **系统模块** | **firewalled** ：使用 firewalld管理任意端 PC 和服务； **reboot** ：重启计算机； **service** ：管理服务； **user** ：添加、删除和管理用户账户 |
| **Net Tools模块** | **get_url** ：通过 HTTP、HTTPS 或 FTP **下载文件** ； **nmcli** ： 管理网络 ；**uri** ：与 web 服务交互 |
