#Linux #Ending_Exam #RHCE #Practical_exams
- 创建使用selinux角色

```YML
---
- name: use selinux roles
  hosts: all
  vars:
    - name: 
      selinux_fcontext:
        target: /var/www/html(/.*)?
        setype: 'http_sys_content_t'
        state: present
      selinux_state: enforcing
  roles:
    - rhel-system-roles.selinux
```
- 创建cron计划任务
```yml
---
- name: create cron job
  cron:
    name: 'cron msg'
    job: logger “Example RHEL” #直接复制粘贴指令
    minute: "2"
    user: "natesha"
```

