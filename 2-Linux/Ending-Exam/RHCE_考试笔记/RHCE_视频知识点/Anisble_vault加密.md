#Linux #Ending_Exam #RHCE #视频笔记
## 一种加密方式
- Ansible Vault
	- `ansible-vault` 命令行工具
		- 可适用清单文件、playbook中含有的变量文件
- 示例：
通过交互的方式设置密码
```shell
[student@Ansible_S test]$ ansible-vault create secret.yml
New Vault password:  # 隐藏 redhat
Confirm New Vault password: 
```

使用 vault密码文件来存储 vault密码，
`ansible-vault create --vault-password-file=vault-pass secret.yml`

### 相关命令：
- 加密：
	- 创建( create )： `ansible-vault create [filename]`
	- 查看( view )： `ansible-vault view [filename]`
	- 编辑现有的加密文件( edit )： `ansible-vault edit [filename]`
	- 后期加密( encrypt )： `ansible-vault encrypt [filename1] [filename2] --vault-password-file [locker_filename]`
		- 可同时加密多个文件
- 解密：
	- 解密( decrypt )：`ansible-vault decrypt [filename] --vault-password-file [locker_filename]`
