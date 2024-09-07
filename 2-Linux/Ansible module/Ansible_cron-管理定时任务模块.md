#Linux #Ansible_Module
## cron模块
- 使用此模块管理crontab和环境变量条目。这个模块允许您创建环境变量和命名为crontab的条目、更新或删除它们。
- 当crontab作业被管理时:模块包含一行crontab条目的描述“#Ansible: &lt;name&gt;”’对应于传递给模块的“name”，它被未来的ansible/module调用用来查找/检查状态。
- “name”参数应该是唯一的，并且更改“name”值将导致创建一个新的cron任务(或删除一个不同的任务)。

### 关键字
```ini
name              任务名称（唯一性）
minute            分
hour              分、时
day               分、时、日
mouth             分、时、日、月
weekday           分、时、日、月、周
user              需要指定执行的用户
job               指定执行的命令
state             状态
special_time      特殊时间-循环
									annually, daily, hourly, monthly, reboot, weekly, yearly
```


> 示例
```yml
- name: use cron module
  hosts: all
  tasks:
    - name:
      cron:
        name: "logger msg"
        minute: "2"
        user: "root"
        job: logger "EX294 in progress"
```
- 结果
```shell
[student@Workstation list-ansible]$ ansible all -m shell -a "crontab -l -u root"
serverd | CHANGED | rc=0 >>
#Ansible: logger msg
2 * * * * logger "EX294 in progress"
```