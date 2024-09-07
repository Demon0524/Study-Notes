#Linux #Ending_Exam #RHCE #Practical_exams 
[TOC]



# RHCE _ 294练习

## 1. 安装和配置 ansible

1. 使用命令将练习机上的受控端 `greg` 用户提权

```bash
ansible all -m shell -a 'chmod -G wheel greg'
```

2. 配置 `inventory` 和 `ansible.cfg` 配置文件

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
[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```
 


## 2. 创建和运行 Ansible 临时命令

1. 编辑 `adhoc.sh` 文件

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

2. 将文件提权 `chmod a+x adhoc.sh` 使文件能够被执行
3.  `./adhoc.sh` 执行临时命令

## 3. 安装软件包

- 方法一：

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

- 方法二：

```yaml
---
- hosts: all
  tasks:
    - name: install php & mariadb
      yum:
        name: "{{ item }}"
        state: latest
      with_items:
        - php
        - mariadb
    - block:
      - name: install RPM Development Tools
        yum:
          name: "@RPM Development Tools"
          state: latest
        when: ansible_hostname in groups['dev']
      - name: uppackages
        yum:
          name: "*"
          state: latest
        when: ansible_hostname in groups['dev']
```

## 4. 使用 RHEL system roles

- 安装相关角色

```bash
sudo yum install -y rhel-system-roles

cp -av /usr/share/ansible/roles/rhel-system-roles.timesync /home/student/ansible/roles

ansible-galaxy list
```

- 调用角色

``` yaml
---
- name: time sync
  hosts: all
  tasks:
  vars:
    timesync_ntp_servers:
      - hostname: ntp.ntsc.ac.cn #[使用提供的服务器]
        ibrurst: yes
  roles:
    - rhel-system-roles.timesync
```

- 验证

```bash
ssh serverf
	timedatectl
	cat /etc/chrony.conf
```

## 5. 使用 Ansible Galaxy 安装角色

-  `vim roles/requirement.yml` 

```yaml
---
- src: http://materials.example.com/phpinfo.tar
  name: phpinfo
- src: http://materials.example.com/haproxy.tar
  name: balancer
```

```bash
ansible-galaxy install -r requirements.yml -p roles/
```

## 6. 创建和使用角色

```bash
cd roles/
ansible-galaxy init apache
vim apache/tasks/main.yml
```

```yaml
---
- name: ensure httpd is install
  yum: 
    name: httpd
    state: latest
- name:  ensure httpd is start and enabled
  service:
    name: httpd
    state: started
    enabled: yes
- name: open firewall port
  firewalld:
    serviec: httpd
    permanent: yes
    state: enabled
    immediate: yes
- name: index html file id installed
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
```

```bash
vim apache/templates/index.html.j2
```

```j2
welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4.address }}
```

```bash
cd /home/greg/ansible
vim newroles.yml
```

```yaml
---
- name: use apache role palybook
  hosts: webservers
  tasks:
  roles:
    - apache
```

```bash
ansible-playbook newroles.yml
curl http://serverc.lab.example.com
	welcome to serverc.lab.example.com on [ipv4_address]
```



## 7. 从 Ansible Galaxy 使用角色

```bash
vim balancer tasks/main/yml
```

```yaml
backend habackend
	serverc serverc.lab.example.com 172.25.250.12:80 check
	serverd serverd.lab.example.com 172.25.250.13:80 check
```

```bash
vim phpinfo templates/hello.php.j2
```

```php
<!DOCTYPE html>
    <html>
    <body>
    <h1>
    PHP World from {{ ansible_fqdn }}
    </h1>
    <?php
        phpinfo();
?>
    </body>
    </html>
```

```bash
vim /home/gerg/ansible/roles.yml
```

```yaml
---
- name: use php role
  hosts: webservers
  roles:
    - phpinfo
- name: use haproxy role
  hosts: balancers
  roles:
    - balancer
  tasks:
    - name: open forewallld port
      firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes
```

```bash
ansible-playbook roles.yml
curl http://servere
```

## 8. 创建并使用磁盘分区和逻辑卷

### 8.1 创建并使用磁盘分区

```bash
vim /home/greg/ansible/partition.yml
```

```yaml
---
- name: create a partition
  hosts: all
  tasks:
    - name: check vdb info
      shell: "ls -l /dev/vdb"
      register: disk_info
      ignore_errors: yes
    - name: show debug message
      debug:
        msg: "disk does not exist"
      when: disk_inof.rc != 0
    - name: use block create partition
      block:
        - name: create 1500MB partition
          parted:
            device: /dev/vdb
            number: 1
            state: present
            part_end: 1500MiB
      rescue:
        - name: show message
          debug:
            msg: "could not creare partition of that size"
        - name: create 800M partition
          parted:
            device: /dev/vdb
            number: 1
            state: present
            part_end: 800MiB
      always:
        - name: mkfs filesystem
          filesystem:
            fstype: ext4
            dev: /dev/vdb1    
        - name: mount filesystem to mount  point
          mount:
            path: /newpart
            src: /dev/vdb1
            fstype: ext4
            state: mounted
```

```bash
ansible-playbook partion.yml
```

### 8.2 创建并使用逻辑卷













