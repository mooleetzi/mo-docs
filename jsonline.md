## load jsonline功能说明

支持jsonline格式数据导入

### 1、基本语法

```
load data infile {'filepath'='data.txt', 'compression'='BZIP2','format'='jsonline','jsondata'='object'} into table db.a

load data infile {'filepath'='data.txt', 'format'='jsonline','jsondata'='object'} into table db.a
```

### 2、参数说明

* filepath：文件路径
* compression：压缩格式，支持BZIP2、GZIP、LZO、SNAPPY、ZLIB(参考csv格式load)
* format：文件格式，支持csv和jsonline
* jsondata：json数据格式，支持object和array，如果`format`为`jsonline`，则**必须**指定`jsondata`

## 3、实现原理

1. 使用simdcsv读入一行jsonline
2. 将jsonline转换成json对象
3. 将json对象转换为一行数据
4. 走默认csv流程

## 4、示例

```shell
> head jsonline_object.jl
{"col1":true,"col2":1,"col3":"var","col4":"2020-09-07","col5":"2020-09-07 00:00:00","col6":"2020-09-07 00:00:00","col7":"18","col8":121.11}
{"col1":"true","col2":"1","col3":"var","col4":"2020-09-07","col5":"2020-09-07 00:00:00","col6":"2020-09-07 00:00:00","col7":"18","col8":"121.11"}
{"col6":"2020-09-07 00:00:00","col7":"18","col8":"121.11","col4":"2020-09-07","col5":"2020-09-07 00:00:00","col1":"true","col2":"1","col3":"var"}
{"col2":1,"col3":"var","col1":true,"col6":"2020-09-07 00:00:00","col7":"18","col4":"2020-09-07","col5":"2020-09-07 00:00:00","col8":121.11}

> head jsonline_array.jl
[true,1,"var","2020-09-07","2020-09-07 00:00:00","2020-09-07 00:00:00","18",121.11]
["true","1","var","2020-09-07","2020-09-07 00:00:00","2020-09-07 00:00:00","18","121.11"]
```

```sql
drop table if exists t1;
create table t1(col1 bool,col2 int,col3 varchar, col4 date,col5 datetime,col6 timestamp,col7 decimal,col8 float);
load data infile {'filepath'='$resources/load_data/jsonline_object.jl','format'='jsonline','jsondata'='object'} into table t1;
select * from t1;
col1	col2	col3	col4	col5	col6	col7	col8
true	1	var	2020-09-07	2020-09-07 00:00:00	2020-09-07 00:00:00	18	121.11
true	1	var	2020-09-07	2020-09-07 00:00:00	2020-09-07 00:00:00	18	121.11
true	1	var	2020-09-07	2020-09-07 00:00:00	2020-09-07 00:00:00	18	121.11
true	1	var	2020-09-07	2020-09-07 00:00:00	2020-09-07 00:00:00	18	121.11

delete from t1;
load data infile {'filepath'='$resources/load_data/jsonline_array.jl','format'='jsonline','jsondata'='array'} into table t1;
select * from t1;
col1	col2	col3	col4	col5	col6	col7	col8
true	1	var	2020-09-07	2020-09-07 00:00:00	2020-09-07 00:00:00	18	121.11
true	1	var	2020-09-07	2020-09-07 00:00:00	2020-09-07 00:00:00	18	121.11
```