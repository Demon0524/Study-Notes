#Linux #Ending_Exam #RHCE #视频笔记
### 批量的进行用户创建

ansible中创建一个普通用户并配置sudo权限
```ini
[/greg]
mkdir playbook
vim ansible.cfg
	[defaults]
	inventory=~/palybook/inventory
	remote_user=greg
	ask_pass=True #在执行时需要输入密码；False：需要免密权限
	roles_path=/root/palybook/roles
	host_key_checking=False #远程登陆不需要进行密钥检查
	[privilege_escalation] #普通用户进行提权操作，首先将用户得在wheel组
	become=True
	become_module=sudo
	become_ask_pass=false
	become_user=root
:wq
```
```ini
vim inventory
	[web]
	web1
	web2
	web3
	[node]
	node1
	node2
	[lamp:children] #嵌套组，要使用children关键字
	web
	node
:wq

```
```yuml
vim 01adduser.yml #利用脚本进行批量设置
	
	---
	- hosts: web,node
	  tasks: 
	    - name: add new user grep
	      user: 
	        name: grep
	        group: #添加用户组
	        groups: wheel #添加附属组
	        append: yes #对添加附加组的确认；相当于'state: present'
```
```shell

ansible all -m ping #查看当前可控的主机

ansible-playbook 01adduser.yml --syntax-check #检查书写时的语法有没有错误
ansible-playbook 01adduser.yml

ansible web -m shell -a 'id grep' #查看web 组中的主机grep用户；ansible命令的简单用法
```
### 结果验证

```shell
[student@Ansible_S playbook]$ ansible-playbook 02_adduser.yml  --syntax-check

playbook: 02_adduser.yml
[student@Ansible_S playbook]$ ansible-playbook 02_adduser.yml  

PLAY [web_servers,node_servers] *********************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [web2.lib.example.com]
ok: [web1]
ok: [web3.example.com]
ok: [192.168.87.131]
ok: [node2.example.com]
ok: [node3.example.com]

TASK [add new user grep] ****************************************************************************************************************
changed: [192.168.87.131]
changed: [web2.lib.example.com]
changed: [web3.example.com]
changed: [web1]
changed: [node2.example.com]
changed: [node3.example.com]

PLAY RECAP ******************************************************************************************************************************
192.168.87.131             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node2.example.com          : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node3.example.com          : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2.lib.example.com       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web3.example.com           : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[student@Ansible_S playbook]$ ansible all -m shell -a 'id grep'
192.168.87.131 | CHANGED | rc=0 >>
uid=1025(grep) gid=1025(grep) 组=1025(grep),10(wheel)
web1 | CHANGED | rc=0 >>
uid=1025(grep) gid=1025(grep) 组=1025(grep),10(wheel)
web2.lib.example.com | CHANGED | rc=0 >>
uid=1025(grep) gid=1025(grep) groups=1025(grep),10(wheel)
web3.example.com | CHANGED | rc=0 >>
uid=1025(grep) gid=1025(grep) groups=1025(grep),10(wheel)
node2.example.com | CHANGED | rc=0 >>
uid=1025(grep) gid=1025(grep) groups=1025(grep),10(wheel)
node3.example.com | CHANGED | rc=0 >>
uid=1025(grep) gid=1025(grep) 组=1025(grep),10(wheel)
[student@Ansible_S playbook]$ 
```

#### Tab进制
vim ~/.vimrc

```
set ai
set tabstop = 2 #强制两个缩进
```


