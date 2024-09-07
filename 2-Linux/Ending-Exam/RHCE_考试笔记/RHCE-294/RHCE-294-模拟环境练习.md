#Linux #Ending_Exam #RHCE #Practical_exams
[TOC]

# RHCE_294练习

## 1. 安装和配置 ansible

```ini
serverf
[dev]
servera
[test]
serverb
[prod]
serverc
serverd
[balancers]
servere
[webservers:children]
prod
```

```ini
[defaults]
inventory = /home/greg/ansible/inventory
remote_user = greg
ask_pass = false
roles_path = /home/greg/ansible/roles
host_key_checking = false
vault_password_file = /home/greg/ansible/secret.txt #不建议添加此键值对，会对之后的题目有影响。但是需要了解该键值对的作用——默认使用的密钥的文件地址

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```



## 2. 创建和运行 Ansible 临时命令

```bash
#! /bin/bash
ansible all -m yum_repository -a \
'name=EX294_BASE description="RH294 base software" \
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/BaseOS \
gpgcheck=yes gpgkey=http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release \
enabled=yes' 

ansible all -m yum_repository -a \
'name=EX294_STREAM description="RH294 stream software" \
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/AppStream \
gpgcheck=yes gpgkey=http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release \
enabled=yes'
```

```bash
chmod a+x adhoc.sh
./adhoc.sh
```

## 3. 安装软件包

```yaml
---
- hosts: dev,test,prod
  tasks:
    - name: install php & mariadb
      yum:
        name:
          - php
          - mariadb
        state: latest
    - name: install RPM Development Tools to dev
      yum:
        name: "@RPM Development Tools"
        state: latest
      when: ansible_hostname in groups['dev']
    - name: updata all package
      yum:
        name: '*'
        state: latest
      when: ansible_hostname in groups['dev']
```



## 4. 使用 RHEL system roles

```bash
sudo yum install -y rhel-system-roles

rpm -ql rhel-system-roles

cp -av /usr/share/ansible/roles/rhel-system-roles.timesync /home/student/ansible/roles

ansible-galaxy list
```

``` yaml
---
- name: time sync
  hosts: all
  tasks:
  vars:
    timesync_ntp_servers:
      - hostname: ntp.ntsc.ac.cn 
        ibrurst: yes
    timesync_ntp_provider: chrony
  roles:
    - rhel-system-roles.timesync
```

```bash
ssh serverf
	timedatectl
	cat /etc/chrony.conf
```



## 5. 使用 Ansible Galaxy 安装角色

```bash
cd roles/
vim requirement.yml
```

```yml
- src: 
  name: balancer
- src: 
  name: phpinfo
```

```shell
ansible-galaxy install -r requirement.yml -p .
```


## 6. 创建和使用角色
- 本题创建的脚本未直接使用，需要与下一题结合
- 对应webservers组使用
```bash
cd /home/greg/ansible/roles
ansible-galaxy init apache
vim apache/tasks/main.yml
```

```yaml
--- 
- name: ensure httpd install
  yum:
  	name: httpd
  	state: latest
-name: ensure firewalld enabled & started
  service:
    naem: firewalld
    enabled: yes
    state: started
# 在含有顺序关系时的不建议使用item循环。
# 如同时需要开启firewalld和httpd组件时
- name: open firewall port
  firewalld:
  	service: http
  	premanent: yes
  	state: present
    immidiate: yes
- name: copy template to webservers
  template: 
  	src: index.html.j2
  	dest: /var/www/html/index.html
  	owner: root
  	group: root  	
```

```shell
vim ~/roles/apache/templates/index.html.j2
```

```j2
welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4 }}
```



## 7. 从 Ansible Galaxy 使用角色

```bash
cd /home/greg/ansible
vim roles.yml
```

```yaml
---
- name: use phpinfo roles
  hosts: webservers
  roles:
  	- phpinfo
  	- apache #调用上一题中的apache模板，使webservers组中的主机开启http功能
- name: use haproxy roles
  hosts: balancers
  roles: 
 	- balancer
  tasks:
  	- name: open firewalld port
  	  firewalld:
  	  	service: http
  	  	permanent: yes
  	  	state: enabled # 注意state的值
  	  	immediate: yes
```

```bash
vim /roles/phpinfo/templates/hello.html.j2
```

```html
<!DOCTYPE html>
  <html>
    <body>
	  <h1> 
    	PHP World form {{ ansible_fqdn }}
      </h1>    
      <?
          phpinfo();
      ?>
    </body>
  </html>
```
- 注意使用的是【**hostname全写**】其template/main.yml中有模板可以引用
```bash
vim roles/balancer/tasks/main.yml
```

```yaml
- name: open haproxy server
  	  lineinfile:
  	  	path: /etc/haproxy/haproxy.cfg
  	  	state: present
  	  	line: "{{ item }}"
  	  with_items:
  	  	- 'server node3.lab.example.com 172.25.250.11:80 check'
  	  	- 'server node4.lab.example.com 172.25.250.12:80 check'
```



## 8. 创建并使用磁盘分区和逻辑卷
- 在使用模块时需要了解其用法
```bash
cd ~/ansible
vim lv.yml
```

```yaml
---
- hosts: all
  tasks:
    - name: show lvm message
      debug:
        msg: 'Volume group done not exist'
      when: "'research' not in ansible_lvm.vgs"
    - name: create lvm data
      block:
        - name: create lvm 1500M
          lvol:
            vg: research
            lv: data
            size: 1500
            state: present
          when: "'research' in ansible_lvm.vgs"
      rescue:
        - name: show error message
          debug:
            msg: "Could not create logical volume of that size"
        - name: create lvm 800M
          lvol: # 使用此模块进行逻辑卷划分
            vg: research
            lv: data
            size: 800
            state: present
          when: "'research' in ansible_lvm.vgs"
      always:
        - name: change filesystem
          filesystem:
            fstype: ext4
            dev: /dev/research/data
```

```bash
ansible all -m setup -a 'filter=*lvm*'
```



## 9. 生成主机文件

```bash
cd ~/ansible
vim hosts.j2
```

```ini
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for host in groups.all %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{ hostvars[host]['ansible_hostname'] }}
{% endfor %}
```
- 注意循环的关键词和结束语句
- 调用的事实变量需要注意写法
```bash
cd ~/ansible
vim hosts.yml
```

```yml
---
- hosts: all
  tasks:
    - name: copy template
      template:
        src: hosts.j2
        dest: /etc/myhosts
      when: inventory_hostname in groups['dev'] #注意要求将文件copy到指定的主机上
```

```bash
ansible dev -m shell -a 'cat /etc/myhosts'
```



## 10. 修改文件内容

```bash
cd ~/ansible
vim issue.yml
```

```yml
---
- hosts: all
  name: copy line file
  tasks:
    - name: copy Dev
      copy:
        content: "Development"
        dest: /etc/issue
      when: "inventory_hostname in groups['dev']"
    - name: copy Test
      copy:
        content: "Test"
        dest: /etc/issue
      when: "inventory_hostname in groups['test']"
    - name: copy Pro
      copy:
        content: "Production"
        dest: /etc/issue
      when: "inventory_hostname in groups['prod']"
```

```bash
ansible all -m shell -a 'cat /etc/issue'
```



## 11. 创建 Web 内容目录

```bash
cd ~/ansible
vim webcontent.yml
```

```bash
---
- hosts: dev
  tasks:
    - name: ensure httpd install
      yum:
        name: httpd
        state: latest
    - name: ensure httpd & firewall enabled
      service:
        name: "{{item}}"
        state: started
        enabled: yes
      with_items:
        - httpd
        - firewalld
    - name: open firewall service
      firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes

    - name: ensure webdev group exist
      user:
        name: webdev
        state: present
    - name: create directory
      file:
        path: /webdev
        group: webdev
        state: directory
        setype: httpd_sys_content_t
        mode: '2775'
    - name: create link /var/www/html/webdev
      file:
        src: /webdev
        dest: /var/www/html/webdev
        state: link
    - name: copy file
      copy:
        content: "Development"
        dest: /webdev/index.html
        group: webdev
        mode: 0644
        setype: httpd_sys_content_t
        
    - name: nodify http.conf file #不了解为什么要执行；
      replace:
        path: /etc/httpd/conf/httpd.conf
        regexp: 'Options Indexes FollowSymLinks'
        replace: 'Options FollowSymLinks'
      notify: restart httpd service
  handlers:
    - name: restart httpd service
      service:
        name: httpd
        state: restarted
```

```bash
curl node1/webdev/ #注意，访问的是主机名下的子目录
```



## 12. 创建硬件报告

```bash
cd ~/ansible
wget http:// #下载hwreport.empt模板进行使用
```
- 建议掌握vim的快捷指令，能够快速的进行替换与定位的操作 
```yaml
---
- name: create hwreport
  hosts: all
  vars:
  tasks:
  	- name: get url
  	  get_url:
  	  	url: http://materials/hwreport.empty
  	  	dest: /root/hwreport.txt
  	- name: replace hostname
  	  replace:
  	  	path: /root/hwreport.txt
  	  	regxep: inventoryhostname
  	  	replace: {{ ansible_hostname }}
  	- name: replace memory_in_MB
  	  replace:
  	  	path: /root/hwreport.txt
  	  	regxep: mumory_in_MB
  	  	replace: {{ ansible_memory_mb }}
  	- name: replace BIOS_version
  	  replace:
  	  	path: /root/hwreport.txt
  	  	regxep: BIOS_version
  	  	replace: {{ ansible_bios_version }}
  	  	
  	- name: replace disk_vda_size
  	  replace:
  	  	path: /root/hwreport.txt
  	  	regxep: disk_vda_size
  	  	replace: {{ ansible_devices.vda.size }}
	  when: "'vda' in ansible_devices"
	- name: replace vda NONE
  	  replace:
  	  	path: /root/hwreport.txt
  	  	regxep: disk_vda_size
  	  	replace: "NONE"
  	   when: "'vda' not in ansible_devices" 
  	- name: replace disk_vdb_size
  	  replace:
  	  	path: /root/hwreport.txt
  	  	regxep: disk_vdb_size
  	  	replace: {{ ansible_devices.vdb.size }}
  	  when: "'vdb' in ansible_devices"
  	- name: replace vdb NONE
  	  replace:
  	  	path: /root/hwreport.txt
  	  	regxep: disk_vdb_size
  	  	replace: "NONE"
  	  when: "'vdb' not in ansible_devices"
```



## 13. 创建密码库

```bash
cd ~/ansible
echo whenyouwishuponastar >> /home/greg/ansible/secret.txt
vim locker.yml
```

```yaml
---
- pw_developer: Imadev
- pw_manager: Imamgr
```

```bash
ansible-vault encrypt --vault-id /home/greg/ansible/secret.txt locker.yml
cat locker.yml
ansible-vault view --vault-passwd-file /home/greg/ansible/secret.txt local.yml
```

> 直接在 ansible.cfg 中设置密码文件路径
>
> vault_password_file=/home/greg/ansible/secret.txt




## 14. 创建用户账户

```bash
cd ~/ansible
vim user_list.yml #题中要求从网站上下载模板
```

```yaml
users:
  - name: bob
    job: developer
  - name: sally
    job: manager
  - name: fred
    job: developer
```

```bash
vim users.yml
```
- 这里需要了解[[../RHCE_视频知识点/loop循环_列表_字典_register注册变量|Loop循环]]的使用 
```yaml
---
- name:
  hosts: dev,test
  vars_files:
    - locker.yml
    - user_list.yml
  tasks:
    - name: ensure devops exist
      group:
        name: devops
        state: present
    - name: verify user_developer exist
      user:
        name: "{{ item.name }}" #调用的自变量
        password: "{{ 'pw_developer' | password_hash('sha512')}}" #使用的关键字需要重新了解一下。
        groups: devops
      loop: "{{ users }}"
      when: item.job == 'developer'

- name:
  hosts: prod
  vars_files:
    - locker.yml
    - user_list.yml
  tasks:
    - name: ensure opsmgr exist
      group:
        name: opsmgr
        state: present
    - name: verify user_manager exist
      user:
        name: "{{ item.name }}"
        password: "{{ 'pw_manager' | password_hash('sha512')}}"
        groups: opsmgr
      loop: "{{ users }}"
      when: item.job == 'manager'
```

> 与上一题相结合，使用



## 15. 更新 ANSIBLE 库密钥

```bash
wget http://materials/salaries.yml
ansible-vault view salaries.yml
ansible-vault rekey salaries.yml
```











