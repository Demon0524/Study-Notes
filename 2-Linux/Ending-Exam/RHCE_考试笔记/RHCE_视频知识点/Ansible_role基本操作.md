#Linux #Ending_Exam #RHCE #视频笔记
[TOC]

# Ansible角色基本操作

##  利用角色简化 PLAYBOOK

### Ansible角色

#### 利用角色构造 **Ansible Playbook** 

- 提供一种方法实现 **重复利用 Ansible 代码** 
- 借助编写良好的角色，可以从 playbook 中向角色传递调整其行为的变量，设置所有站点相关的主机名、IP地址、用户名，或其他你在本地主机需要的详细信息。

#### Ansible 角色具有的优点

- 可以分组内容，轻松共享代码

- 编写角色来定义系统类型的基本要素：Web服务器、数据库服务器、Git存储库

- 

#### Ansible 角色结构

- Ansible 角色有子目录和文件的标准化结构定义

```shell
[student@Workstation ansible_student]$ tree
.
├── defaults # 此中的 main.yml 文件包含角色变量的默认值，使用角色时可以覆盖这些 默认值
├── files # 此目录由角色引用的静态文件
├── handlers # 包含角色的处理程序定义
├── meta # 包含与角色相关的信息；作者、许可证、平台和可选的角色依赖项
├── tasks # 包含角色的任务定义
├── templates # 角色应用的 jinja2 模板
├── tests # 用于测试角色
└── vars # 用于定义角色的变量值；通常用于角色内部用途；优先级较高
```

- 自动生成角色

```bash
ansible-galaxy init [name] -p [path]
```



### 在 PALYBOOK 中使用 Ansible 角色

- 对于每个指定的角色，角色任务、角色处理程序、角色变量和角色依赖项将**按照顺序导入到 Playbook 中** 。角色中的任何 copy、script、template 或 include_tasks/import_tasks **任务都可以应用角色中** 相关的 **文件、模板** 或 **任务文件，且无需相对或绝对路径名称** 。

- Ansible 将 **分别在角色的 files、 template、 ** 或 **tasks 子目录中寻找它们** 。

##### 引用角色示例

```yml
---
- hosts: serverf
  roles: #关键字
   - role1
   - role2
```

- 使用 role2 时，任何 defaults 和 vars 变量都会被覆盖

```yml
---
- hosts: serverf
  roles:
   - role: role1
   - role: role2
     var1: val1 # 通过传递；优先级更高
     var2: val2
```

#### 控制执行顺序

- 以下示例play演示了一个带有 **pre_tasks(前)、roles、tasks、post_tasks(后) 和 handlers** 的示例，一个play中通常不会同时包含所有这些部分。
  - import_task
  - imclude_task

##### 控制执行顺序示例 

```yml
---
- name: Play to illustrate order of execution
  hosts: serverf
  pre_tasks: # 放在整个 palybook 中，实现预处理
    - debug:
        msg: 'pre task'
      notify: my handler
  # roles:
  #   - role1
  # 没有定义 role1 ，暂不使用
# 引用角色，自定义角色内的内容
  tasks: 
    - debug:
        msg: 'first task'
      notify: my handler
  post_tasks:
    - debug:
        msg: 'post task'
      notify: my handler
  handlers:
    - name: my handler
      debug:
        msg: Running my handler
```

- 导入角色
  - include_role---动态
  - import_role---静态

- 一下示例演示了如何通过 **include_role** 模块来利用任务包含角色

```yml
---
- name: Execute a role as a task
  hosts: serverf
  tasks:
    - name: A notmal task
      debug:
        msg: 'First task'
    - name: A task to include role2 here
      include_role: role2
```

## 利用系统角色重用内容



多个文件的集合称之为  `collection` 








