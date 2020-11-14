

### 常用查询命令

1. 查询缓存情况 `SHOW VARIABLES LIKE '%cache%';`
2. 查询慢查询情况   `SHOW VARIABLES LIKE '%slow%'; ` 或 `SHOW GLOBAL STATUS LIKE '%slow%';  `
3. 查看最大链接数 `SHOW VARIABLES LIKE 'max_connections';  `
4. 查看最大链接过的用户数 `SHOW GLOBAL STATUS LIKE 'max_used_connections';`
5. 显示用户正在运行的线程
  `SHOW PROCESSLIST;`  或 `SELECT * FROM information_schema.processlist;` 。参考：[mysql: show processlist 详解](https://zhuanlan.zhihu.com/p/30743094)
  
    如果出现有表锁定，无法查询的情况，可以使用  该命令来查询出任务的线程的id，然后`kill id`，即可结束线程(注意不要随便在运行的服务上，做alter table操作，包含更改表备注，不然可能会导致表锁定，深刻体会)。参考：[MySQL出现Waiting for table metadata lock的原因以及解决方法](https://www.cnblogs.com/digdeep/p/4892953.html)

6. 查询出innodb的引擎的情况 `SHOW ENGINE INNODB STATUS` 。参考：[mysql性能监控指标及分析](https://blog.csdn.net/rudygao/article/details/47151033)

7. 在线`alter`表的注意事项，需要先查询事务，是否有事务在锁表了，有则不能`alter`那么快
   
   5.5 版本(不支持在线ddl)
     ```sql
          SELECT
          *
          /**
               a.trx_id,
               a.trx_state,
               a.trx_started,
               a.trx_query,
               b.ID,
               b. USER,
               b. HOST,
               b.DB,
               b.COMMAND,
               b.TIME,
               b.STATE,
               b.INFO
               
          **/
          FROM
               information_schema.INNODB_TRX a
          LEFT JOIN information_schema.PROCESSLIST b ON a.trx_mysql_thread_id = b.id
          WHERE
               b.COMMAND = 'Sleep';
          
     ```
     5.6版本（支持在线ddl）
     ```sql
          SELECT
               a.trx_id,
               a.trx_state,
               a.trx_started,
               a.trx_query,
               b.ID,
               b.USER,
               b.DB,
               b.COMMAND,
               b.TIME,
               b.STATE,
               b.INFO,
               c.PROCESSLIST_USER,
               c.PROCESSLIST_HOST,
               c.PROCESSLIST_DB,
               d.SQL_TEXT
          FROM
               information_schema.INNODB_TRX a
          LEFT JOIN information_schema.PROCESSLIST b ON a.trx_mysql_thread_id = b.id
          AND b.COMMAND = 'Sleep'
          LEFT JOIN PERFORMANCE_SCHEMA.threads c ON b.id = c.PROCESSLIST_ID
          LEFT JOIN PERFORMANCE_SCHEMA.events_statements_current d ON d.THREAD_ID = c.THREAD_ID;
     ```



8. 查询某个表的索引情况，一般查询太慢都是没有命中联合索引.
     ```sql
     SELECT a.TABLE_SCHEMA,
     a.TABLE_NAME,
     a.index_name,
     GROUP_CONCAT(column_name ORDER BY seq_in_index) AS `Columns`
     FROM information_schema.statistics a
     GROUP BY a.TABLE_SCHEMA,a.TABLE_NAME,a.index_name
     having a.TABLE_NAME='dm_yy_kxd_nationnalver_ztsj_xg_day'
     ```

9. Mysql备份数据库：`mysqldump -h 127.0.0.1 -P 3306 -u root -ppsd  --databases  samego > /home/alic/MySQL/samego.sql`
 
   其中，`127.0.0.1 `为数据库地址，`3306`为数据库端口，`root`为数据库账号，`psd` 为数据库账号密码，`samego`为数据库的名称，`/home/alic/MySQL/samego.sql` 为备份的数据库的文件的地址.
   
    参考：[MySQL在线DDL修改表结构的简单经验分享](https://cloud.tencent.com/developer/article/1072214)

10. 如果查询某个表，看起来数量很少，但是实际查询很慢，可能是因为有些数据还没实际删除。一般我们执行 `delete` 操作并不会实际删除表在磁盘上的数据，实际删除表在磁盘上的数据需要执行以下两个命令，`optimize table t`或`alter table t engine=InnoDB`  参考：[mysql删除操作其实是假删除
](https://zhuanlan.zhihu.com/p/66336976)


11. 查询数据库或表的容量
 	  查询数据库的容量
	```sql
	select 
	table_schema as '数据库',
	table_name as '表名',
	table_rows as '记录数',
	truncate(data_length/1024/1024, 2) as '数据容量(MB)',
	truncate(index_length/1024/1024, 2) as '索引容量(MB)'
	from information_schema.tables
	order by data_length desc, index_length desc;
	```
	  查询表的容量
	```sql
	SELECT 
	table_schema AS '数据库',
	table_name AS '表名',
	table_rows AS '记录数',
	TRUNCATE(data_length/1024/1024, 2) AS '数据容量(MB)',
	TRUNCATE(max_data_length/1024/1024,2) AS '最大数据长度(MB)',
    TRUNCATE(data_free/1024/1024,2) AS '空间碎片(MB)',
	TRUNCATE(index_length/1024/1024, 2) AS '索引容量(MB)'
	FROM information_schema.tables
	ORDER BY data_length DESC, index_length DESC;
	```
	

### 参考
1. [MySQL查看数据库表容量大小](https://blog.csdn.net/fdipzone/article/details/80144166)
2. [记一次 mysql 磁盘满解决过程](https://testerhome.com/topics/23049)
	

  
