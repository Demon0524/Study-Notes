#Linux #Ending_Exam #RHCE #视频笔记
[TOC]

# 6.0 在被管理节点上创建文件或目录

-  课程目标
  - 创建、安装、编辑和删除文件，管理这些文件的权限、所有权、SELinux 上下文和其他特征
  - 使用 **Jinja2 模板自定义文件** ，并部署

### 修改、复制文件

- 查看文件的SELinux标签
  - `ll -Z [filename]` 
  - semanage  可以设置文件的 SELinux 标签

#### 模块

- file

```yaml
- name: SELinux type is set to samba_share_t
	file:
		path: /path/to/samba_file #想要修改文件的路径
		setype: samba_share_t #修改路径文件的 SELinux 标签
		# 此行为与 Linux chcon^[修改文件的安全上下文] 命令类似
		
```



- lineinfile

```yaml
- name: Add a line of text to a file
	# 在远程主机的文件中添加一行 'line' 
	lineinfile:
		path: /path/to/file
		line: 'Add this line to the file'
		state: present/adsent(删除)
		
		insertafter: '^%sudo' # 在“以sudo开头的一行”之后插入 'line' 字段
```

- blockinfile

```yaml
- name: Add additional lines of to a file
	# 在远程主机的文件中添加多行 'block'
	lineinfile:
		path: /path/to/file
		block: | # '|' 表示换行
			First line in the additional block of text
			Second line in the additional block of text
		state: present
```

## 魔法变量---特殊变量

- ansible_files
- 



### JINJA2 模板

