## export to sql功能说明

比如mysqldump，MO需要一个工具或者一个语句，将数据库或者表导出成一个同时具有DDL和DML的SQL文件。然后用户可以运行导出的 SQL 文件来重新创建这些对象。

### 1、使用方法

```
./mo-dump -u ${user} -p ${password} -h ${host} -P ${port} -db ${database} [-tbl ${table}...]
``` 

### 2、参数说明

* u 用户名, 默认dump
* p 密码, 默认111
* h mo部署ip, 默认127.0.0.1
* P mo部署端口, 默认6001
* db 数据库名, 必填
* tbl 表名, 未指定则导出整个数据库;指定多个表名时需要多次指定-tbl参数，参考最后一个样例

### 3、实现原理

#### 3.1 获取待导出元素的ddl

目前使用`show create table`语句获取表的ddl，使用`show create database`语句获取数据库的ddl。
后续会修改为从`mo_catalog`数据库中查询ddl。

#### 3.2 获取数据生成insert语句

使用`select * from table`语句获取数据

#### 3.3 将ddl和insert语句写入文件

### 4、使用示例

```sql
DROP DATABASE IF EXISTS `t`;
CREATE DATABASE `t`;
USE `t`;
create table t1
(
    c1  int primary key auto_increment,
    c2  tinyint not null default 4,
    c3  smallint,
    c4  bigint,
    c5  tinyint unsigned,
    c6  smallint unsigned,
    c7  int unsigned,
    c8  bigint unsigned,
    c9  float,
    c10 double,
    c11 date,
    c12 datetime,
    c13 timestamp on update current_timestamp,
    c14 char,
    c15 varchar,
    c16 json,
    c17 decimal,
    c18 text,
    c19 blob,
    c20 uuid
);
insert into t1
values (1, 1, 1, 1, 1, 1, 1, 1, 1, 1, '2019-01-01', '2019-01-01 00:00:00', '2019-01-01 00:00:00', 'a', 'a', '{"a":1}',
        '1212.1212', 'a', 'aza', '00000000-0000-0000-0000-000000000000');
select *
from t1;
+----+----+----+----+----+----+----+----+----+----+------------+---------------------+---------------------+----+----+--------+----------+----+----+--------------------------------------+
| c1 | c2 | c3 | c4 | c5 | c6 | c7 | c8 | c9 | c10 | c11        | c12                 | c13                 | c14 | c15 | c16    | c17      | c18 | c19 | c20                                  |
+----+----+----+----+----+----+----+----+----+----+------------+---------------------+---------------------+----+----+--------+----------+----+----+--------------------------------------+
|  1 |  1 |  1 |  1 |  1 |  1 |  1 |  1 |  1 |  1 | 2019-01-01 | 2019-01-01 00:00:00 | 2019-01-01 00:00:00 | a  | a  | {"a":1} | 1212.1212 | a  | aza | 00000000-0000-0000-0000-000000000000 |
+----+----+----+----+----+----+----+----+----+----+------------+---------------------+---------------------+----+----+--------+----------+----+----+--------------------------------------+
1 row in set
    (0.00 sec)
create table t2
(
    a varchar(10),
    b int
);
insert into t2
values ('a', 1),
       ('b', 2);
```

```shell
./mo-dump -u dump -p 111 -h 127.0.0.1 -P 6001 -db t
DROP DATABASE IF EXISTS `t`;
CREATE DATABASE `t` ;
USE `t`;
DROP TABLE IF EXISTS `t1`;
CREATE TABLE `t1` (
`c1` INT NOT NULL AUTO_INCREMENT,
`c2` TINYINT DEFAULT 4,
`c3` SMALLINT DEFAULT null,
`c4` BIGINT DEFAULT null,
`c5` TINYINT UNSIGNED DEFAULT null,
`c6` SMALLINT UNSIGNED DEFAULT null,
`c7` INT UNSIGNED DEFAULT null,
`c8` BIGINT UNSIGNED DEFAULT null,
`c9` FLOAT DEFAULT null,
`c10` DOUBLE DEFAULT null,
`c11` DATE DEFAULT null,
`c12` DATETIME DEFAULT null,
`c13` TIMESTAMP DEFAULT null ON UPDATE current_timestamp(),
`c14` CHAR(1) DEFAULT null,
`c15` VARCHAR(1) DEFAULT 'qaq',
`c16` JSON DEFAULT null,
`c17` DECIMAL(34,0) DEFAULT null,
`c18` TEXT DEFAULT null,
`c19` BLOB DEFAULT null,
`c20` UUID DEFAULT null,
PRIMARY KEY (`c1`)
);
INSERT INTO `t1` VALUES (1,1,1,1,1,1,1,1,1,1,'2019-01-01','2019-01-01 00:00:00','2019-01-01 00:00:00','a','a','{"a": 1}','1212','a','aza','00000000-0000-0000-0000-000000000000');
DROP TABLE IF EXISTS `t2`;
CREATE TABLE `t2` (
`a` VARCHAR(10) DEFAULT null,
`b` INT DEFAULT null
);
INSERT INTO `t2` VALUES ('a',1),('b',2);
```

```shell
./mo-dump -u dump -p 111 -h 127.0.0.1 -P 6001 -db t > t.sql
cat t.sql
DROP DATABASE IF EXISTS `t`;
CREATE DATABASE `t` ;
USE `t`;
DROP TABLE IF EXISTS `t1`;
CREATE TABLE `t1` (
`c1` INT NOT NULL AUTO_INCREMENT,
`c2` TINYINT DEFAULT 4,
`c3` SMALLINT DEFAULT null,
`c4` BIGINT DEFAULT null,
`c5` TINYINT UNSIGNED DEFAULT null,
`c6` SMALLINT UNSIGNED DEFAULT null,
`c7` INT UNSIGNED DEFAULT null,
`c8` BIGINT UNSIGNED DEFAULT null,
`c9` FLOAT DEFAULT null,
`c10` DOUBLE DEFAULT null,
`c11` DATE DEFAULT null,
`c12` DATETIME DEFAULT null,
`c13` TIMESTAMP DEFAULT null ON UPDATE current_timestamp(),
`c14` CHAR(1) DEFAULT null,
`c15` VARCHAR(1) DEFAULT 'qaq',
`c16` JSON DEFAULT null,
`c17` DECIMAL(34,0) DEFAULT null,
`c18` TEXT DEFAULT null,
`c19` BLOB DEFAULT null,
`c20` UUID DEFAULT null,
PRIMARY KEY (`c1`)
);
INSERT INTO `t1` VALUES (1,1,1,1,1,1,1,1,1,1,'2019-01-01','2019-01-01 00:00:00','2019-01-01 00:00:00','a','a','{"a": 1}','1212','a','aza','00000000-0000-0000-0000-000000000000');
DROP TABLE IF EXISTS `t2`;
CREATE TABLE `t2` (
`a` VARCHAR(10) DEFAULT null,
`b` INT DEFAULT null
);
INSERT INTO `t2` VALUES ('a',1),('b',2);
```

```shell
./mo-dump -u dump -p 111 -h 127.0.0.1 -P 6001 -db t -tbl t2
DROP TABLE IF EXISTS `t2`;
CREATE TABLE `t2` (
`a` VARCHAR(10) DEFAULT null,
`b` INT DEFAULT null
);
INSERT INTO `t2` VALUES ('a',1),('b',2);
```

```shell
./mo-dump -u dump -p 111 -db t -tbl t2 -tbl t1
DROP TABLE IF EXISTS `t2`;
CREATE TABLE `t2` (
`a` VARCHAR(10) DEFAULT null,
`b` INT DEFAULT null
);
INSERT INTO `t2` VALUES ('a',1),('b',2);
CREATE TABLE `t1` (
`c1` INT NOT NULL AUTO_INCREMENT,
`c2` TINYINT DEFAULT 4,
`c3` SMALLINT DEFAULT null,
`c4` BIGINT DEFAULT null,
`c5` TINYINT UNSIGNED DEFAULT null,
`c6` SMALLINT UNSIGNED DEFAULT null,
`c7` INT UNSIGNED DEFAULT null,
`c8` BIGINT UNSIGNED DEFAULT null,
`c9` FLOAT DEFAULT null,
`c10` DOUBLE DEFAULT null,
`c11` DATE DEFAULT null,
`c12` DATETIME DEFAULT null,
`c13` TIMESTAMP DEFAULT null ON UPDATE current_timestamp(),
`c14` CHAR(1) DEFAULT null,
`c15` VARCHAR(1) DEFAULT 'qaq',
`c16` JSON DEFAULT null,
`c17` DECIMAL(34,0) DEFAULT null,
`c18` TEXT DEFAULT null,
`c19` BLOB DEFAULT null,
`c20` UUID DEFAULT null,
PRIMARY KEY (`c1`)
);
INSERT INTO `t1` VALUES (1,1,1,1,1,1,1,1,1,1,'2019-01-01','2019-01-01 00:00:00','2019-01-01 00:00:00','a','a','{"a": 1}','1212','a','aza','00000000-0000-0000-0000-000000000000');
```