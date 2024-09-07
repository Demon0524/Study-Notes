#Linux #Ending_Exam #RHCE #视频笔记 
[TOC]
# 5.0_Loop循环-列表-字典-register注册变量

## 实施任务的控制

### 利用循环迭代任务

1. 利用 **loop** （if/while）关键字对一组项目迭代任务
2. 简单循环对 **一组项目**（对象、数组）迭代任务。loop关键字添加到任务中，将对应其 **迭代任务的项目列表取为值** 
3. 循环变量 **item** 保存每个迭代过程中使用的值

- item：

#### 示例

- 无 **loop** 循环

```yaml
- name: Postfix is running # 启动 Postfix 服务
  service:
    name: postfix
    state: started
- name: Dovecot is running # 启动Dovecot 服务
  service: 
    name: dovecot
    state:started
```
+ 有 **loop** 循环

```yaml
# 使用列表结构进行书写
- name: Postfix and Dovecot are running
  service: 
    name: "{{ item }}"
    state: started
  loop: # 迭代调用；每循环一次， loop 中的关键字被 item 调用
    - postfix #列表结构
    - dovecot
```



### loop标准循环基本语法

1. 迭代一个简单的列表，重复的任务可以在一个简单的字符串列表上写成标准循环

```yaml
- name: Add serveral users
  user: 
    name: "{{ item }}"
    state: present
    groups: "wheel"
  loop： #列表/数组 数据结构
    - testuser1 #列表结构
    - testuser2 
```
#### 示例

- 可以在变量文件中 **定义列表** ，或者在play的vars中，引用列表名称：

```yaml
loop: "{{ somelist }}"
```

- 无**loop**循环方法

```yaml
- name: Adduser testuser1
  user:
    name: "testuser1"
    state: present
    groups: "wheel"
- name: Adduser testuser2
  user: 
    name: "testuser2"
    state: present
    groups: "wheel"
```
- **loop**循环方法
  - 对于某些插件，可以**将列表直接传递给参数**。如果可用，将列表传递给参数比循环遍历任务要好
    - 循环遍历任务——每执行一次都将返回起始点重新进行下一次循环的计算；打印输出每一个数值

```yaml
- name: Option yum
	yum:
		name: "{{ list_of_packages }}" # 先定义变量
		state: present
- name: Non-option yum,dlower and may cause issues with interdependencies
	yum:
		name: "{{ item }}"
		state: present
  loop: "{{ list_of_packages }}" #调用变量
```





### 字典形式

- loop 列表不一定是简单的值列表。在以下的示例中，列表中的每个项实际上是 **散列** 或 **字典** 
- 示例中的 **每个散列** 或 **字典具有两个键** ，即 **name 和 groups** ，当前 **item** 循环变量中每一个键的值可以分别通过 **item.name** 和 **item.groups** 变量来 **检索** 
  - 对象的结构： **key: value** 

```yaml
- name:
  user:
    name: "{{ item.name }}" 
    # 循环调用loop中的 name 变量；“ . ”&“ [] ” ---表示引用
    state: present
    groups: "{{ item.groups }}"
loop: #列表 & 字典的结构
  - name: jane # key: 【name】; value：【jane】
    groups: wheel
  - name: joe
    groups: root
```
通过引用 key 的值进行调用

### loop遍历一组字典

- 如果有一组 **字典对象** ，可以在循环中 **引用子健** 
```yaml
- name: Adduser serveral users
  user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
	loop:
		- { name: 'testuser1', groups: 'wheel' } #平铺格式
		- { name: 'testuser2', groups: 'root' }
```

- 当条件语句与循环组合使， **when** 语句将针对每一个项分别处理
  -  **遍历字典** ；这里遍历 tag_data 并打印其中的键和值

```yaml
- name: Usering dict2items
	debug:
		msg: "{{ item.key }} - {{ item.value }}"
	loop: "{{ tag_data | dict2items }}" # " | "过滤；将 tag_data 当成字典去处理，
	vars: # 定义变量
		tag_data:
			Environment: dev
			Application: payment
```

### Register变量与Loop

- Register关键字也可以 **捕获循环任务的输出** 

  ```yaml
  ---
  - name: Loop Register Test
  	gather_facts: no
  	hosts: localhost
  	tasks:
  		- name: Looping Echo Task
  			shell: "echo This is my item: {{ item }}" #“item”是内置的循环变量
  			loop:
  				- one
  				- two
        register: echo_results # 注册变量；保存记录输出的信息；引用其字典的值——“['  ']”
      - name: Show echo_results variable
      	debug:
      		var: echo_results.results[0].rc # 输出item第0个循环的值的rc
  
  ## 以下时演示部分
  #调用输出的信息
      - name: Show echo_results variable
      	debug:
      		var: echo_results.results[0].rc    
          echo_registers={"results": [{"rs","0"}，{"start","2022-07-07"}]} 
          echo_regists['results'][0]['rc']
          echo_regists['results'][0]['start']
          # 此时输出的数据结构是【字典-列表-字典】
          # **调用字典时仅需要通过“.”或“['']”的方式调用自建即可；**
          # **调用列表时时可以通过下标索引进行调用。从 0 开始**
          # 一直进行重复的调用
          
          list1=[one,two,three]   -->   list1[1] #输出的值是“two”  
```

-  **调用字典时仅需要通过“.”或“['']”的方式调用自键即可；**
-  **调用列表时时可以通过下标索引进行调用。从 0 开始**

- 注册变量及事实变量都是要掌握的
