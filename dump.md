## export to sql功能说明

比如mysqldump，MO需要一个工具或者一个语句，将数据库或者表导出成一个同时具有DDL和DML的SQL文件。然后用户可以运行导出的 SQL 文件来重新创建这些对象。

### 1、基本语法

```
dump [all|database databaseName|table tableName] into 'filePath';
``` 

### 2、参数说明

* all：导出当前用户所有可见数据库里的所有表 **?可见性是其他地方实现的吧**
* database databaseName：导出指定数据库里的所有表
* table tableName：如果指定了数据库名，导出指定数据库里的指定表，否则导出当前数据库里的指定表
* filePath：导出文件的路径

### 3、实现原理

**直接在前端处理还是走查询计划？**

#### 3.1 获取待导出元素的ddl

1. parser -> frontend -> session -> session.Pu.StorageEngine 获取db和table的def
2. 生成ddl语句

#### 3.2 获取数据生成insert语句

**? 读数据需要考虑事务吗**

1. 获取数据 parser -> frontend -> session -> session.Pu.StorageEngine
2. 生成对应insert语句

#### 3.3 将ddl和insert语句写入文件