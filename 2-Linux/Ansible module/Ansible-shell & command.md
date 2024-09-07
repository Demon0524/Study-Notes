#Linux #Ansible_Module
### shell & command的使用
>一般使用此命令来删除指定的文件or文件夹

##### shell: 
直接调用`/bin/bash`


- 删除原始的Yum源码，使用中科大Yum源码
```yml
---
name: rm all host Yum_rpeo
  hosts: all
  tasks:
    shell:
	  "rm -rf /etc/yum_rpeos.d/*"
```
- 一段话使用ansible命令执行
```shell
ansible all -m shell 
```




