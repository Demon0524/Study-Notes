# MySQL 笔记

## 数据库操作

操作数据库 > 操作数据库中的表 > 操作数据库中表的数据

 ==mysql关键字不区分大小写== 

**数据库：**
- DDL 数据库**定义**语言

- DML 数据库**操作**语言

- DQL 数据库**查询**语言

- DCL 数据库**控制**语言

### 一、增删改查操作

> 一、增：创建数据库和数据表

```sql
-- 创建 [school] 数据库
CREATE DATABASE IF NOT EXISTS school;

-- 使用 [school] 数据库并创建 [sutdent] 表
USE `school`;
CREATE TABLE IF NOT EXISTS `student`(
`student_id` BIGINT(10) NOT NULL AUTO_INCREMENT COMMENT '学号',
`name` VARCHAR(30) NOT NULL DEFAULT '姓名' COMMENT '姓名',
`sex` VARCHAR(10) NOT NULL DEFAULT '男' COMMENT '性别',
`age` INT COMMENT '年龄',
PRIMARY KEY (`student_id`)
)ENGINE = INNODB DEFAULT CHARSET = utf8

-- 创建表格式
/*
create table [if not exists] `（表名）`(
`字段名` 列类型 [属性] [索引] [注释],
primary key (`主键`)
)[表类型] [字符集] [注释]
engine = innodb default charset = utf8
*/
```

> 二、删：删除表、数据库

- 删除数据库&数据表

```sql
USE school;
-- 删除 [school] 数据库
DROP DATABASE school;
-- drop database [if exists] 数据库名;

-- 删除表
DROP TABLE student;
```



> 三、改：修改字段名、属性、

1. 查看命令

```sql
-- 所有的一句都是以 ; 号结尾
show databases; -- 查看所有的数据库

mysql> use school; -- 切换数据库 use 
Database changed；

show tables； -- 查看数据库中的所有的表
describe student； -- 显示数据库中所有的表的信息
exit; -- 退出
-- 单行注释（SQL 的本来的注释）
```

2. 创建命令

```sql
create database westos; -- 创建westos数据库
```

3. 





> 四、查





> 四、创建外键

- **在创建子表时同时创建外键**

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

- **创建子表完成后，修改子表添加外键**

```sql
-- 创建外键方式二 : 创建子表完毕后,修改子表添加外键
ALTER TABLE `student`
ADD CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade` (`gradeid`);
```





```sql
CREATE DATABASE IF NOT EXISTS school;
-- 学生白名册
CREATE TABLE IF NOT EXISTS school.`student`
(
`student_id` INT(10) NOT NULL COMMENT '学号' PRIMARY KEY,
`student_name` VARCHAR(256) NOT NULL COMMENT '姓名',
`sex` TINYINT DEFAULT 1 NOT NULL COMMENT '性别',
`gradeid` VARCHAR(254) NOT NULL COMMENT '年级',
`phoneNum` VARCHAR(256) NOT NULL COMMENT '手机',
`address` VARCHAR(256) NOT NULL COMMENT '地址',
`borndate` DATETIME NOT NULL COMMENT '生日',
`email` VARCHAR(50) NOT NULL COMMENT '邮箱',
`idCard` VARCHAR(100) NOT NULL COMMENT '身份证号'
) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT '学生白名册';
```

```sql
INSERT INTO school.`student` (`student_id`, `student_name`, `sex`, `gradeid`, `phoneNum`, `address`, `borndate`, `email`, `idCard`) VALUES ('1', '陶驰', 1, '大数据科学与技术', '17694079209', '石嘴山', '2022-10-31 02:42:41', 'bette.gleichner@hotmail.com', '8506b114-8117-4fcd-975e-12919cad6155');
...
INSERT INTO school.`student` (`student_id`, `student_name`, `sex`, `gradeid`, `phoneNum`, `address`, `borndate`, `email`, `idCard`) VALUES ('100', '石文轩', 1, '计算机科学与技术', '17364381782', '成都', '2022-10-29 20:21:13', 'diego.nader@hotmail.com', 'e655d637-ed11-458c-a4a7-0fe394440138');

```



