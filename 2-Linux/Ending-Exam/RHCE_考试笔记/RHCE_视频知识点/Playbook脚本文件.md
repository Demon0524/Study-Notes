#Linux #Ending_Exam #RHCE #视频笔记
[TOC]

## Playbook脚本文件

### 相关命令：

`ansible web_servers -m copy -a 'src=hosts dest=/opt/hosts'`
将 hosts 拷贝到 web_servers 组下的主机的 /opt/hosts 文件中

- Ansible 中的任务配置文件被称之为 Playbook，可称之为“剧本”。
	- 剧本中可包含多个任务
	- 针对清单中选定的主机，进行一系列的有序任务

 ### Playbook语法格式：
 - 通过缩进实现不同的代码块区分
 - 使用Python中的关键字定义代码段的用处，关键字的**冒号后要有空格**


 - **定义全局的Playbook（hosts 的位置）**

```yaml
--- #表示一份内容的开头
- hosts: # 主机的关键字
	- ##列表格式
  tasks: # 任务的关键字
```
- **局部Playbook脚本**
`ansible web_servers -m user -a 'name=newbie uid=4000 state=present'`

```yaml
---
- name: Cnofigure important user consistently
  hosts: web_servers #可以时单个的主机，也可以是主机组
  tasts：
    - name: newbie exists With UID 4000
      user:
        name: newbie
        uid: 4000
        state: present
```

### YAML基本格式
- 基本语法规则
	- 大小写敏感
	- 使用缩进便是层级关系
	- 缩进时只允许空格，
	- 空格数目不重要，
	- ‘#’ 表示注释
- 支持的主要数据结构
	- 对象：**键值对的集合**，又称映射、哈希、**字典**
		- 可以使用 ‘:’ 或 ‘空格’ 来分开
	- 数组：一组按次序盘列的值，又称序列、列表

#### 列表结构与字典结构的区别

- 列表（list）
	主要有了 “-” 表示每一行的值
```yaml
---
- hosts: all
  vars:
  	- yum: yum list
  	- mkdir: mkdir dircetory
  	- touch: touch file
```
- 字典
	使用 “,” 分割每个值，每个值需要用**引号**引用
```yaml
---
- hosts: all
  tasks:
  vars:
```
### Tab 缩进的改变
vim $HOME/.vimrc
`autocmd FileType yaml setlocal ai ts=2 sw=2 et` 

### 关键字