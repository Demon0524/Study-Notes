#Linux #Ending_Exam #RHCE #视频笔记
[TOC]
# 4.0_Ansible事实

- 每次执行脚本时的第一条便是收集事实变量
`ansible [host_name] -m setup`
- debug --- msg 显示模块
```yaml
---
- hosts: all
  tasks:
  - name:
    debug:
	  msg:>
	   The defautle ipv4 address {{ ansible_facts['all_ipv4_addresses'] }} # 调用主机事实变量中的IP地址子变量
	   The host kernel is {{ ansible_facts[''] }} # 获取该主机的内核信息
```
## 可以通过 filter 进行搜索

`ansible node1 -m setup -a 'filter=*kernel*'`

## 关闭事实收集

```yaml
- name: This paly gathers no fact automatically
  hosts: web_servers
  gather_facts: no/yes # 关闭、开启 事实收集
```

### 创建自定义事实

- 管理员可以创建自定义事实，将其本地存储在每个受控的主机上。这些事实 **整合** 到setup模块在受管主机上运行时手机的标准事实列表中。他们让受管主机能够向Ansible提供任意变量，以用于调整play的行为。
-  **自定义事实** 可以在 **静态文件中定义** ，格式可以是 **INI文件** 或者采用 **JSON** 

#### 注意事项

- 默认的 **setup模块** 从各受管主机的 **/etc/ansible/facts.d 目录** 下
	- 各文件的脚本必须以**.fact**结尾才能使用
	- 在被控节点中的 **/etc/ansible/facts.d** 目录下创建以 **.fact**结尾的文件

### 示例

- INI:
```ini
[packages]
web_package=httpd
ssh_packages=sshd
[users]
user1=joe
```
- JSON
```json
{
    "packages": {
        "web_package": "httpd"
    },
    "users": {
        "user1": "joe"
    }
}
```
