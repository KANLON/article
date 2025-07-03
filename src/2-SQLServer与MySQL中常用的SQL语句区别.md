
## SQL Server 与 MySQL 的区别


&emsp;&emsp;由于博主之前学过mysql，目前在学习SQL server，原来以为这两个数据库之间的sql语句应该不会有太大区别。但是学sql server（用的版本是2012） 学久之后，发现与mysql 还是有很多不同的地方，有些时候写起来很不适应，所以就打算写这篇博客来记录一下这两个数据库的sql语句的区别，以备方便将来查询。（每隔一段时间会更新）

<hr/>

### 1. 首先说一下sql语句结束标志
mysql以`;`结束一条SQL语句；SQL server 以`;`或`go`或不写结束都可以。不过建议写sql server的时候以`go`结束，因为sql server很多语句要以等一个语句结束后才能写下一个语句，不然会报<font color='red' >批处理错误</font>（深受其害）。<br/><br/>

### 2. 查看表结构数量等
  mysql 语句
  
```sql
-- 查看系统内所有数据库
show databases；
-- 查询数据库内所有表
show tables;
-- 显示表结构
desc 表名;
```

  sql server语句
  

```sql
-- 查看系统内所有数据库
SELECT name, database_id, create_date  FROM sys.databases  ;
-- 查询数据库内所有表
select * from sysobjects where xtype= 'U'  ;
-- 显示表结构
sp_help/sp_columns 表名;
```
  
相比来说，mysql 的更为简洁。
### 3、查询前几条记录

查询前10条记录：

mysql 语句
`select * from student limit 10;`

sql server 语句
`select top 10 * from student ;`

###  4、获取当前时间

MySQL写法：`now()`

SQLServer写法：`getdate()`

###  5、从数据库定位到某张表
mysql写法：库名.表名

```sql
select password from Info.users where userName='boss'
```

Sqlserver写法：库名.dbo.表名 ；或者：库名..表名  （注：中间使用两个点）

```sql
select password from Info.dbo.users where userName='boss'
```

或者
```sql
select password from Info..users where userName='boss'
```

###  6、强制不使用缓存查询
查询temp表

mysql写法：

```sql
select    SQL_NO_CACHE * from temp
```

参考：[https://www.cnblogs.com/eyesfree/p/7232559.html](https://www.cnblogs.com/eyesfree/p/7232559.html)


Sqlserver的没有，它只缓存sql的执行计划，不会缓存结果。

参考：[https://www.cnblogs.com/jeffwongishandsome/p/3235177.html](https://www.cnblogs.com/jeffwongishandsome/p/3235177.html)
[https://codeday.me/bug/20190701/1344787.html](https://codeday.me/bug/20190701/1344787.html)


###  7、查询一个数据库所有的表 和表下的所有列信息
MySQL写法：

写法1（查询选定数据库下所有的表）：
```sql
select table_name tableName, engine, table_comment tableComment, create_time createTime from information_schema.tables where table_schema = (select database())
```

写法2（查询指定数据库名下所有的表）：
```sql
select table_name tableName, engine, table_comment tableComment, create_time createTime from information_schema.tables where table_schema = 'db_name'
```

其中的db_name就是要查询的数据库的名

查询表下的所有列信息：

```sql
select column_name columnName, data_type dataType, column_comment columnComment, column_key columnKey, extra from information_schema.columns
        where table_name = '表名' and table_schema = '数据库名' order by ordinal_position
```


参考：[https://blog.csdn.net/huangbaokang/article/details/78049629](https://blog.csdn.net/huangbaokang/article/details/78049629)

SQL Server写法：

查询数据库中的所有表:
```sql
select name from sysobjects where xtype='u'
```

查询指定表的所有列名
```sql
select name from syscolumns where id=(select max(id) from sysobjects where xtype='u' and name='表名')
```

参考：[https://blog.csdn.net/ANXIN997483092/article/details/78468081](https://blog.csdn.net/ANXIN997483092/article/details/78468081)


### 8.快速查询一个表的行数

Mysql写法：

```sql
SELECT   TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH,
DATA_LENGTH FROM  INFORMATION_SCHEMA.PARTITIONS  WHERE 
TABLE_NAME='table_name_t' ;  
```

其中`table_name_t`是要查询下表名.

这里是通过 `INFORMATION_SCHEMA`数据库中的信息来得到行数，这样会快一点，但是可能会不太准确。

参考：[https://blog.csdn.net/jiaxiaolei19871112/article/details/7161799](https://blog.csdn.net/jiaxiaolei19871112/article/details/7161799)

SqlServer 写法：

```sql
select schema_name(t.schema_id) as [Schema], t.name as TableName,i.rows as [RowCount] 
from sys.tables as t, sysindexes as i 
where t.object_id = i.id and i.indid <=1
```
这种写法可以列出所有表的行数，可以在该查询的基础之上去再去过滤器想要的表名

参考：[https://www.cnblogs.com/kenyang/archive/2013/04/09/3011447.html](https://www.cnblogs.com/kenyang/archive/2013/04/09/3011447.html)

 
### 9. 分页

要获取第12行到第21行的记录可以这样写：

Mysql写法：

```sql
select * from test_t limit 11,10;
```


SqlServer 写法：

```sql
select * from student
order by sno  
offset 11 rows
fetch next 10 rows only ;
```
 
