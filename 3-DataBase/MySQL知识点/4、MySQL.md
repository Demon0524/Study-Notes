[#![🕸️](https://twemoji.maxcdn.com/v/13.1.0/72x72/1f578.png)数据库知识点]

# 学习使用MySQL数据

==**数据库语言**==
- DDL 数据定义语言
- DML 数据操作语言
- DQL 数据查询语言
- DCL 数据控制语言

## 1、安装 MySQL 及 SQLyog

> 下载安装 MySQL 的压缩包，使用压缩包进行安装

```bash
# 在 mysql 文件夹的 bin 目录下
.\mysqld --install
.\mysqld --initialize-insecure --user=mysql # 初始化数据
net start mysql # 启动数据库
mysql -uroot -p123456 -- 链接数据库
update mysql.user set authentication_password('root') where user=('root') and Host=('localhost');  -- 修改用户密码
flush privileges; -- 刷新权限
```

> 安装 sqlyog --- https://github.com/webyog/sqlyog-community/wiki/Downloads


## 2、数据库操作

> 结构化查询语句分类

| 名称                | 解释                                         | 命令                    |
| ------------------- | :------------------------------------------- | ----------------------- |
| DDL（数据定义语言） | 定义和管理数据对象，如数据库、数据表等       | CREATE、DROP、ALTER     |
| DML（数据操作语言） | 用于操作数据库对象中所包含的数据             | INSERT、UPDATE、DELETE  |
| DQL（数据查询语言） | 用于查询数据库数据                           | SELECT                  |
| DCL（数据控制语言） | 用于管理数据库的语言，包括管理权限及数据更改 | GRANT、commit、rollback |

操作数据库 > 操作数据库中的表 > 操作数据库中表的数据

 ==mysql关键字不区分大小写== 

### 2.1、数据库操作

> 命令行操作数据库

创建数据库：create database [if not exists] 数据库名;

删除数据库：drop database [if exists] 数据库名;

使用数据库：use 数据库名;

查看数据库：show database; 



> 对比工具操作数据库

- 对照 sqlyog 可视化历史记录查看 sql 
- 固定的语法或关键字必须强行记住！

### 2.2、创建数据表

属于 DDL 的一种，语法:

```sql
create table [if not exists] `表名`(
   '字段名1' 列类型 [属性][索引][注释],
   '字段名2' 列类型 [属性][索引][注释],
   ...
   '字段名n' 列类型 [属性][索引][注释]
)[表类型][表字符集][注释];
```

**说明:** 反引号用于区别MySQL保留字与普通字符而引入的 (键盘esc下面的键)

### 2.3、数据库的列类型

列类型：规律数据库中该列表存放的数据类型

> 数值类型

| 类型      | 说明               | 存储需求 | 使用范围     |
| --------- | ------------------ | -------- | ------------ |
| tinyint   | 十分小的数据       | 1个字节  |              |
| smallint  | 较小的数据         | 2个字节  |              |
| mediumint | 中等大小的数据     | 3个字节  |              |
| int       | 标准的数据         | 4个字节  | 常用         |
| bigint    | 较大的数据         | 8个字节  |              |
| float     | 浮点数             | 4个字节  |              |
| duoble    | 浮点数             | 8个字节  |              |
| decimal   | 字符串形式的浮点数   | | 用于分式小数 |



> 字符串

| 类型               | 说明                                          | 最大长度  |
| ------------------ | --------------------------------------------- | --------- |
| char[(M)]          | 固定长字符串，**检索快但费空间** 0<= M <= 255 | 0 ~ 255   |
| varchar[(M)]--常用 | 可变字符串 0<= M <= 65535                     | 0 ~ 65535 |
| tinytext           | 微型文本串                                    | 2^8-1     |
| text--保存文本     | 文本串                                        | 2^16-1    |



> 日期和时间型数值类型

java.util.Date

| 类型         | 说明                            | 取值问题                                   |
| ------------ | ------------------------------- | ------------------------------------------ |
| DATE         | YYYY-MM-DD，日期格式            | 1000-01-01~9999-12-31                      |
| TIME         | Hh:mm:ss，时间格式              | -838:59:59~838:59:59                       |
| ==DATETIME== | YY-MM-DD hh:mm:ss               | 1000-01-01 00:00:00 至 9999-12-31 23:59:59 |
| TIMESTAMP    | YYYYMMDDhhmmss 格式表示的时间戳 | 197010101000000~2037年的某个时刻           |
| YEAR         | YYYY 格式的年份值               | 1901~2155                                  |



> null

- 没有值，未知
- ==注意不要使用NULL进行运算，结果为NULL==



### 2.4、数据库的字段属性

> ==UnSigned== 

- 无符号的
- 声明该数据不允许负数



> ==ZEROFILL== 

- 0填充
- 不足位数的用0来填充，如 int(3),5则为005



> ==Auto_InCrement== 

- 自动增长的，每添加一条数据，自动在上一个记录数上加1（默认）
- 通常用于设置 **主键** ，且为整数类型
- 可定义起始值和步长
  - 当前表设置步长（AUTO_INCREMENT=100）：只影响当前表
  - SET @@auto_increment_insrement=5；影响所有使用自增的表（全局）



> ==NULL 和 NOT NULL== 

- 默认为NULL，及时没有插入该列的数值
- 如果设置的为NOT NULL，则该列必须有值



> ==DEFAULT== 

- 默认的
- 用于设置默认值
- 例如：性别字段，默认为“男”，否则为“女”；若无指定该列的值，则默认值为“男”的值



```sql
-- 目标 : 创建一个school数据库
-- 创建学生表(列,字段)
-- 学号int 登录密码varchar(20) 姓名,性别varchar(2),出生日期(datatime),家庭住址,email
-- 创建表之前 , 一定要先选择数据库

CREATE TABLE IF NOT EXISTS `student` (
`id` int(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
`name` varchar(30) NOT NULL DEFAULT '匿名' COMMENT '姓名',
`pwd` varchar(20) NOT NULL DEFAULT '123456' COMMENT '密码',
`sex` varchar(2) NOT NULL DEFAULT '男' COMMENT '性别',
`birthday` datetime DEFAULT NULL COMMENT '生日',
`address` varchar(100) DEFAULT NULL COMMENT '地址',
`email` varchar(50) DEFAULT NULL COMMENT '邮箱',
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

-- 查看数据库的定义
SHOW CREATE DATABASE school;
-- 查看数据表的定义
SHOW CREATE TABLE student;
-- 显示表结构
DESC student;  -- 设置严格检查模式(不能容错了)SET sql_mode='STRICT_TRANS_TABLES';
```

### 2.5、数据表的类型

> 设置数据表的类型

```sql
CREATE TABLE 表名(
   -- 省略一些代码
   -- Mysql注释
   -- 1. # 单行注释
   -- 2. /*...*/ 多行注释
)ENGINE = MyISAM (or InnoDB)

-- 查看mysql所支持的引擎类型 (表类型)
SHOW ENGINES;
```

常见的 MyISAM 与 InnoDB 类型：

| 名称       | MyISAM | InnoDB      |
| ---------- | ------ | ----------- |
| 事务处理   | 不支持 | 支持        |
| 数据行锁定 | 不支持 | 支持        |
| 外键约束   | 不支持 | 支持        |
| 全文索引   | 支持   | 不支持      |
| 表空间大小 | 较小   | 较大，约2倍 |

经验（适用场合）：

- 适合 MyISAM：节约空间及响应速度
- 适用 InnoDB：安全性，事务处理及多用户操作数据表



> 数据表的存储位置

- MySQL数据表以文件方式存放在磁盘中
  - 包括表文件, 数据文件, 以及数据库的选项文件
  - 位置: Mysql安装目录\data\下存放数据表;目录名对应数据库名,该目录下文件名对应数据表

- 注意:
  - \*.frm -- 表结构定义文件
  - \*.MYD -- 数据文件 ( data )
  - \*.MYI -- 索引文件 ( index )
  - InnoDB类型数据表只有一个 *.frm文件 , 以及上一级目录的ibdata1文件
  - MyISAM类型数据表对应三个文件 




> 设置数据表字符集

我们可以为数据库、数据表、数据列设定不同的字符集，设定方法：

- 创建是通过命令来设置。如：DREATE TABLE 表名()CHARSET=utf8;
- 如无设定，则根据MySQL数据库配置文件 my.ini 中的超参数设定



### 2.6、修改数据库

> 修改表（ALTER TABLE）

修改表名：ALTER TABLE 旧表名 REMANE AS 新表名

添加字段：ALTER TABLE 表名 ADD字段名 列属性[属性]

修改字段：

- ALTER TABLE 表名 MODIFY 字段名 列属性[属性]
- ALTER TABLE 表名 CHANGE 旧字段名 新字段名 列属性[属性]

删除字段：ALTER TABLE 表名 DROP 字段名



> 删除数据表

语法： DROP TABLE [IF EXISTS] 表名

- IF EXISTS 为可选，半段是否存在改数据表
- 如删除不存在的数据表会抛出错误



> 其它

```sql
1. 可用反引号（`）为标识符（库名、表名、字段名、索引、别名）包裹，以避免与关键字重名！中文也可以作为标识符！

2. 每个库目录存在一个保存当前数据库的选项文件db.opt。

3. 注释：
  单行注释 # 注释内容
  多行注释 /* 注释内容 */
  单行注释 -- 注释内容       (标准SQL注释风格，要求双破折号后加一空格符（空格、TAB、换行等）)
   
4. 模式通配符：
  _   任意单个字符
  %   任意多个字符，甚至包括零字符
  单引号需要进行转义 \'
   
5. CMD命令行内的语句结束符可以为 ";", "\G", "\g"，仅影响显示结果。其他地方还是用分号结束。delimiter 可修改当前对话的语句结束符。

6. SQL对大小写不敏感 （关键字）

7. 清除已有语句：\c
```

## 3、DML语言--MySQL数据管理

### 3.1、外键

> 外键概念

如果公共关键字在一个关系中是主关键字，那么这个公共关键字被称之为另一个关键的外键。由此可见，==外键表示了两个关系之间的相关联系。以另外一个关系的外键作为主关键字的表被称之为**主表**，具有此外键的表被称为主表的**从表**。==

在实际操作中，讲一个表的值放入第二个表来表示关联，所使用的值是第一个表的主键值（在必要时可包括复合主键值）。此外，第二个表中保存的这些值的属性被称之为**外键（foreign key）**。

- 外键的作用

保持数据的**一致性、完整性**，主要目的是控制存储在外键表中的数据**（约束）**。使两张表形成关联，外键只能引用外表中的列的值或使用空值。



> 创建外键

- 建表时指定外键约束

```sql
-- 创建外键的方式一：创建子表的同时创建外键
USE school;
-- 年级表（id\年级名称）
CREATE TABLE `grade`(
`gradeid` INT (10) NOT NULL AUTO_INCREMENT COMMENT '年级ID',
`gradename` VARCHAR(50) NOT NULL COMMENT '年级名称',
PRIMARY KEY (`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

-- 学生信息表（学号、姓名、性别、年级、手机、地址、出生日期、邮箱、身份证号）
CREATE TABLE IF NOT EXISTS `student`(
`sudent_id` INT (10) NOT NULL COMMENT '学号',
`sudent_name` VARCHAR(30) NOT NULL COMMENT '姓名',
`sex` TINYINT(1) DEFAULT '1' COMMENT '性别',
`gradeid` INT(10) DEFAULT NULL COMMENT '年级',
`phoneNum` VARCHAR(20) NOT NULL COMMENT '手机',
`address` VARCHAR(255) DEFAULT NULL COMMENT '地址',
`borndate` DATETIME DEFAULT NULL COMMENT '生日',
`email` VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
`idCard` VARCHAR(18) DEFAULT NULL COMMENT '身份证号',
PRIMARY KEY (`sudent_id`),
KEY `FK_gradeid`(`gradeid`),
CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade` (`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8
```

- 建表后修改

```sql
-- 创建外键方式二 : 创建子表完毕后,修改子表添加外键
ALTER TABLE `student`
ADD CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade` (`gradeid`);
```



> 删除外键

**注意**: 删除具有主外键关系的表时 , 要先删子表 , 后删主表

```sql
-- 删除外键
ALTER TABLE student DROP FOREIGN KEY FK_gradeid;
-- 发现执行完上面的,索引还在,所以还要删除索引
-- 注:这个索引是建立外键的时候默认生成的
ALTER TABLE student DROP INDEX FK_gradeid;
```



### 3.2、DML语言

**数据库意义：** 数据存储、数据管理

**管理数据库数据方法:** 

- 通过SQLyog等管理工具管理数据库数据
- 通过**DML语句**管理数据库数据

==**DML语言**== ：数据库操作语言

- 用于操作数据库对象中所包含的数据
- 包括：
  - INSERT（添加数据语句）
  - UPDATE（更改数据语句）
  - DELETE（删除数据语句）



### 3.3、添加数据

> INSERT 命令

**语法：** 

```sql
INSERT INTO 表名 (列名1,列名2,...列名N)
values ('字段1','字段2','字段3',...'字段N');
```

**注意：**

- 字段或值之间用英文逗号隔开
- '字段1，字段2...'该部分可省略，但添加的值务必于表结构、数据列、顺序相对应，且数量一致
- 可同时插入多条数据，values后用英文逗号隔开

```sql
-- 使用语句如何增加语句?
-- 语法 : INSERT INTO 表名[(字段1,字段2,字段3,...)] VALUES('值1','值2','值3')
INSERT INTO grade(gradename) VALUES ('大一');

-- 主键自增,那能否省略呢?
INSERT INTO grade VALUES ('大二');

-- 查询:INSERT INTO grade VALUE ('大二')错误代码：1136
Column count doesn`t match value count at row 1

-- 结论:'字段1,字段2...'该部分可省略 , 但添加的值务必与表结构,数据列,顺序相对应,且数量一致.

-- 一次插入多条数据
INSERT INTO grade(gradename) VALUES ('大三'),('大四');
```



### 3.4、修改数据

> UPDATA 命令

**语法：**

```sql
UPDATE 表名 SET column_name=value
[,column_name2=value2,...][WHERE condition];
```

**注意：**

- column_name 为要更改的数据列
- vaule 为修改后的数据，可以为变量；具体指表达式或嵌套的SELECT结果
- condition 为筛选条件，如不指定则修改该表的所有列数据



> where 条件子句

简单理解为：有条件的从表中筛选数据

**常用的运算符**

| 运算符 | 含义 | 范围 | 结果 |
| ------ | ---- | ---- | ---- |
|        |      |      |      |



### 3.5、删除数据

> DELETE 命令

**语法：**

```sql
DELETE FROM 表名 [WHERE condition];
```

**注意：**

```sql
-- 删除最后一条数据
DELETE FROM grade WHERE gradeid = 5;
```



> TRUNCATE 命令

**作用:** 用于完全清空表数据，但表结构、索引、约束等不变

**语法：**

```sql
TRUNCATE [TABLE] table_name;

-- 	清空年级表
TRUNCATE grade;
```

**注意：区别于DELETE命令**

- 相同：都能删除数，不能删除表结构；TRUNCATE速度更快
- 不同：
  - TRUNCATE TABLE 重新设置 AUTO_INCREMENT计数器
  - TRUNCATE TABLE 不会对事务有影响

