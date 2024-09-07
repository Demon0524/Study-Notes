#Linux #Ending_Exam #RHCE #视频笔记
[TOC]

# 5.1_when条件-hanlder程序处理-block简介

## 编写条件任务

### 有条件的运行任务

- Ansible 可使用 conditionals 在 **符合特定条件** 是 **执行任务或play** 。
- 可以利用条件来区别不同的受管主机，并根据其符合的条件来分配功能角色。 **Playbook变量** 、 **注册的变量** 和 **Ansible事实** 都是可以通过 **条件来进行测试**
  - Anaible 可以 **捕获并评估命令的输出** ，以确定一项任务在执行进一步操作前是否已经完成
  - 利用 Ansible 事实管理 **受管主机网络配置** 
  - 可以 **评估CPU的数量** ，来确定如何正确调节某一 web服务器
  - 将 **注册的变量与预定义的变量进行比较** ，以确定服务是否已更改。
- 处理条件是的运算符：
  - 基本运算符
  - 变量（不）存在
    - （not）defined
  - 布尔值为true（false---取反）
    - （not）
  - 第一个变量的值存在，作为第二个变量的列表中的值
    - [] in [] ——通过“in“

#### 示例文件

- 查看当前系统中有哪些事实可用

```yaml
---
- hosts: localhost
	tasks: 
		- name: show fact available on the system
			debug:
				var: ansible_facts
```

- 简单的基于事实条件判断的例子

```yaml
tasks:
	- name: Shut down Debian flavored systems
		command: /sbin/shutdown -t now
		when: ansible_facts['os_family'] == "Debian" # 对事实变量进行判断
```

- when 语句用于有条件地运行任务。满足则运行；不满足则跳过

  - 通过测试布尔变量的值；一下 **任务仅在 run_my_task 为true 时运行（when 调用 run_my_task 变量的时候，不需要加 {{}} ）**  

  - when 模块与需要进行判定的模块进行对其

    ```yaml
    - name: Simple Boolean Task Demo
    	hosts: all
    	vars: # 定义变量
    		run_my_task: true
      tasks:
      	- name: httpd package is installed
      		yum:
      			name: httpd
          when: run_my_task
    ```

-  **ansible_distribution** 变量是在 **Gather_facts** 任务期间确定的事实

-  **supported_distros** 由 **playbook** 作者创建，包含改 playbook 支持的操作系统方法列表

```yaml
--- 
- name: Demonstrate the "in" keyword
	hosts: all
	gather_fasts: yes
	vars:
		supported_distros
		- RedHat
		- Fedroa
	tasks：
		- name: Install httpd using yum, where supported
			yum:
				name: httpd
				state: present
			when: ansible_distribution in supported_distros
```

### 测试多个条件

- 同时满足设定的条件才能够进一步的执行，使用 ”and“ 和 ”or“ 关键字进行组合

- 通过 **使用括号分组条件** ，可以表达等复杂的条件语句

```yaml
when: >
	(ansible_distribution == "RedHat" and ansible_distribution_major_version == "7")
	(ansible_distribution == "Fedora" and ansible_distribution_major_version == "28")
```

### 组合循环和有条件任务

- 安装 mariadb-server 软件包，只要 / 上挂载的文件系统超多 300M 的可用空间。 **ansible_mounts 事实是一组字典** ，各代表一个已挂载文件系统的相关事实（可以判定设备是否由足够的空间）；循环迭代列表中的每一个字典

```yaml
- name: install mariadb-server if enough space on root
	yum:
		name: mariadb-server
		state: present
	loop: "{{ ansible_mounts }}" # 循环这个事实变量---字典结构
	when: item.mount == "/" and item.size_available > 30000000 # 循环其自建---调用 key
```

- 组合使用条件和注册变量的示例

```yaml
--- 
- name: Restart HTTPD if Postfix is Running
	hosts: all
	tasks:
		- name: Get Postfix server status
			command: /usr/bin/systemctl is-active postfix #Postfix 是否运行
			ignore_ errors: yes # 忽略报错
			register: result # 将收集到的变量放置在 result 中；字典的数据结构
		- name: Restart Apache HTTPD based on Postfix status
			service:
				name: httpd
				state: restarted
			when: result.rc == 0 # 如果收集到的 rc值 为0，则重启 httpd 服务
```

### when示例




## 实施处理程序

### ANSIBLE 处理程序——hanlers

- Ansible 模块具有幂等性。再更改任务配置时需要 **重新加载程序** 进而实现 **更改的配置生效**
-  **处理程序---handler** 是 **响应由其他任务触发的通知的任务---notify** 。每个处理程序具有全局唯一的名称，在 playbook 中 **任务块的末尾触发** 。
-  **不通知，不运行** 
- 如果 notify **没有报告changed 结果** ，则程序 **不会收到通知** ， **handlers 便不能执行** 

#### 示例文件

- 只有更新了配置文件才能够进行生效， **restart apache** 处理程序才会重启 Apache 服务器

```yaml
tasks:
	- name: copy demo.example.conf configuration template0
		template:
			src:
			dest:
		notify: # notify 语句指出该任务需要出发的一个处理程序
			- restart apache # 要运行的处理程序的名称；与 handlers中的名称一一对应
handlers: # 与 tasks 同级；
  -	name: restart apache # 被任务调用的处理程序的名称；
   	service: # 用于该处理程序的模块
    	name: httpd
   		state: restarted
```

- 一个任务可以在其 notify 部分中调用对各处理程序
- ansible 将 notify 语句视为数组，并迭代进行处理程序名称
  - 分别调用一个 handlers 的处理程序


### Handler 处理程序示例

- ![image-20220707152858982](C:\Users\Demon\AppData\Roaming\Typora\typora-user-images\image-20220707152858982.png)

- 处理程序也可以 ”listen“ 一般的主题，添加一个 ”listen“ 的key，监听 notify 的通知状况

```yaml
handlers:
	- name: Restart memcached
		service: 
			naem: memcached
			state: restarted
		listen: "restart web services"
  - name: restart apache 		
  	service: 
			naem: apache
			state: restarted
		listen: "restart web services"
		
tasks:
	- name: Restart everything
		command: echo "This task will restart the web serviced"
		notify: "restart web service" # 监听此处
```

### 处理任务失败

#### 管理PLAY中的错误

- 忽略任务失败

```yaml
- name: 
	yum:
		name:
		state:
	ignore_errors: yes # "yes" 忽略报错，接下来的任务继续执行
```

#### 任务失败后强制执行处理程序

-  **force_handlers: yes** 关键字，即使 play 因为 **后续任务失败** 为中止 **也会调用被通知的处理程序** 

```yaml
--- 
- hosts: servera
  force_handlers: yes
  tasks:
    - name: a 
      command: /bin/true
      notify: restart the database
    - name: a task which fails 
      yum: 
        name: notapkg
        state: latest
  handlers:
    - name: restart the database
      service:
        name: mariadb
        state: restarted
```

#### 指定任务失败的条件

-  **failed_when** 关键字来 **指定表示任务已失败的条件** 。**通常与命令模块搭配使用** 

```yaml
tasks:
	- name: Run user 
    shell: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
```

- fail 模块也可用于强制任务失败。

```yaml
tasks:
  - name: Run user 
    shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: yes
  - name: Report 
    fail:
      msg: "This password id missing in the output"
    when: "'Password missing' in command_result.stdout"

```

#### 指定何时任务报告 “Changed” 结果

-  **changed_when** 关键字用于控制任务在 **何时报告它以进行了更改** 
- 通过 **register** 收集到的注册变量判断；进行人为控制该任务是否被判定为 *change*
- 可以与 **notify-handler** 配合使用
- 与之前的 when 的语法结构相同
```yaml
- name: run shell command
	shell: hostname
	register: info_name
	changed_when: info_name.rc == 0
```

#### 错误处理 block

- block：定义要运行的主要任务
- rescue：定义要在 block 子句中定义的任务失败时运行的任务
- always：定义始终都独立运行的任务，不论 block 和 rescue 子句中定义的任务是成功还是失败

```yaml
tasks:
	- name: Upgrade DB
		# 同级缩进
		block: # 
			- name:upgrade the database
				shell:
					cmd: /usr/local/lib/upgrade-database	
		rescue: # 当 block 失败才会运行 rescue
			- name:revert the database upgrade
				shell:
					cmd: /usr/local/lib/revert-database
		always: # 总是执行
			- name: always restart the database
				service；
					name: mariadb
					state: restarted
```











