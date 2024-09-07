#Linux_commands #正则表达式
# 详解
> awk是一个强大的linux命令，有强大的文本格式化的能力，好比将一些文本数据转化成专业的Excel表的样式

## awk基础
> 语法
```bash
awk [option] 'pattern[action]' file ...
awk  参数     '条件动作'          文件
```


- action值得是动作，awk擅长文本格式化，且输出格式化后的结果，因此最常用的动作就是`print`和`printf`
## awk处理
```bash
[root@servera ~]# cat testfile 
xm 60 70 89 99
myy 80 50 80 98
mt 87 76 64 99

# awk又被称之为流编辑器，通过逐行扫描的方式遍历文本
# 每一行称之为 record
# 每一个字段为 field
# 分隔符为 filed separator；分隔符默认为【空格】，可修改
```

## awk场景
> 动作场景

## 选项
- awk常用参数

| 参数 | 解释                            |
| ---- | ------------------------------- |
| -v   | value，设置变量                 |
| -F   | filed separator，输入字段分隔符 |

- awk常用内置变量

| 变量 | 变量全称                  | 解释               |
| ---- | ------------------------- | ------------------ |
| $[n] | number                    | 第n个字段          |
| $0   |                           | 当前整条记录       |
| FS   | filed separator，         | 输入字段分隔符     |
| RS   | record separator，        | 输入记录分隔符     |
| OFS  | output filed separator，  | 输出字段分隔符     |
| ORS  | output record separator， | 输出记录分隔符     |
| NF   | number of fileds，        | 当前记录包含字段数 |
| NR   | number of reccords，      | 当期已处理记录数   |

- awk内置函数

| 函数   | 解释     |
| ------ | -------- |
| print  |          |
| printf |          |
| length | 字段长度 |
| BEGIN  |          |
| END    |          |

BRGIN语句块--->模式匹配语句块--->END语句块 




# 实验
## 

我们执行的命令是`awk '{print $2}'`，没有使用参数和模式，`$2`表示输出文本的`第二列`信息
awk默认以空格为分隔符，且多个空格也识别为一个空格，作为分隔符
awk是按行处理文件，一行处理完毕，处理下一行，根据用户指定的分隔符去工作，没有指定则默认空格
> 指定了分隔符，awk吧每一行切割后的数据对应到内置变量中

- $0表示整行
- $NF表示当前分割的最后一列
- 倒数第二列可以写成$(NF-1)

```bash
# 输出lsblk的第七列
[root@localhost /]$ lsblk | awk '{print $7}'
MOUNTPOINT

/boot

/
/home
/var

# 配合grep命令输出一个非空白行
[root@localhost /]$ lsblk | awk '{print $7}' |grep -v '^$'
MOUNTPOINT
/boot
/
/home
/var

# 指定文字匹配
```



