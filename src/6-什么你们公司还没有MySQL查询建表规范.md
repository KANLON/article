
&emsp;&emsp;因为之前一直再查找一些比较好的数据库规范，以方便在开发时连接 MySQL 进行查询/建表的时候，能根据规范来执行，达到提高 查询速度 / 执行 SQL 的性能 和提升 MySQL 的整体性能， 这里主要是存放一些比较好的一些数据库设计规范（主要用了`公司某位同事`整理的数据库规范，已得到该大佬的授权），我在此基础上增补了部分规则。目前我们组基本就是使用该规范来开发的。

## 1. 基本规范

1. 使用 innodb 存储引擎 后续使用 TokuDB 存储引擎
2. 表字符集使用 UTF8mb4
3. 所有表和表字段必须添加备注，表备注格式为：`创建人，创建时间，表的业务说明`
4. 所有表必须添加主键和自增 id，自增列无业务意义也需添加
5. 生产 mysql 数据库不允许存储图片和文件等大数据
6. 禁止在线上数据库进行压力测试
7. 数据量较大的表建议建分区表
8. 禁止使用外键 允许逻辑意义上的外键依赖
9. 能用代理账号链接数据库的尽量使用代理账号链接，方便运维那边以后做数据库迁移工作
10. 如果只涉及到查询操作的，使用只查权限的账号建立链接，例如：[Superset](https://github.com/apache/incubator-superset) 等bi系统建立报表的查询
11. 一般情况下线上业务使用的数据库账号为写的账号，因为线上业务一般不涉及到对字段/表定义更改操作，管理权限权限账号一般只用于自己开发本地连接
12. 申请/创建 数据库的时候一般使用主主配置即可，即一个库作为写主库，一个库作为读主库
13. 修改测试库 ddl 级别的 需要提交脚本至升级文档 升级时一起执行脚本 确保测试库和正式库统一



## 2. 命名规范

1. 表名建议带上业务英文名的，第一个字母 +"_" 开头，例如:data_service 业务，表名应该都是以 d_开头，控制在 15 个字符以内
2. 表名字符禁止超过 32 个字符
3. 库名、表名、字段名禁止使用 MySQL 保留字
4. 临时库、表名必须以 `tmp` 为前缀，并以日期为后缀
5. 备份库、表必须以 `bak` 为前缀，并以日期为后缀
6. 操作日志类型的库、表建议以`log`为后缀
7. 普通索引命名：`idx_开头 + 字段名`（单阶字段用全字段名，双阶或者三阶使用前阶首字母加最后阶的全拼，例如：create_time 索引命名为：idx_ctime）
8. 唯一索引命名：`uniq_开头 + 字段名` 方法参考二级索引命名

## 3. 数据表设计规范

1. 自增 id 必须添加 unsigned
2. 字段设置 not null 的必须有默认值
3. 少用 text 类型，尽可能改用 varchar，禁止使用 blob 类型，实在避免不了 blob，请拆表
4. 存储非负整数需添加 unsigned
5. 禁止使用 FLOAT 和 DOUBLE 存储浮点数，改用 DECIMAL 存储
6. 尽可能不使用 `default null`，非必要使用 null，建议改用 `default 0` 或 `default ''` (以免使用查询时候null 值情况查询需要另外处理)，同时也不建议使用 `default ''`
7. 所有 type 和 status 等业务字段使用 TINYINT 类型，禁止使用 varchar，避免不了 varchar 的情况下则使用 enum 类型
8. 使用 datetime 存储时间（mysql5.6 以后）
9. 用好数值类型字段，如果数值字段没那么大，就不要用 bigint
10. 禁止在数据库中存储明文密码，把密码加密后存储
11. 能使用分区表尽量使用分区表
12. 表设计前必须评估字段的字长，避免没必要的空间浪费和性能损耗
13. 建议单表字段数控制在 20 个以内，再多建议垂直分表
14. 外键约束一般不在数据库上创建，只表达一个逻辑的概念，由程序控制
15. 所有表必须有 create_time 和 update_time，并添加触发器（CURRENT_TIMESTAMP 和 CURRENT_TIMESTAMP ON UPDATE）

## 四。索引规范

1. 表必须有主键（聚集索引）
2. 不使用 UUID MD5 HASH 这些作为主键 (数值太离散)
3. 默认使非空的唯一键作为主键
4. 一般自增列为主键，分区表的分区键可以与自增列做复合索引
5. 单索引字段数不超过 5 个
6. 优先考虑覆盖索引
7. 避免冗余和重复索引
8. 避免使用基数低的字段做索引，例如：性别、类型、状态
9. 低基数字段建议根据 where 谓词条件建立复合索引
10. ORDER BY、GROUP BY、DISTINCT、WHERE 后字段需根据实际情况建立索引
11. 索引字段的默认值不能为 NULL
12. 能使用唯一索引就要使用唯一索引，但要求是字长不能超 8 位和不离散的字段，超 8 位的和离散字段，使用前缀索引

## 5. SQL 开发规范

1. 代码中禁止使用 `select *`
2. 不使用标量子查询，尽可能改外连接
3. 表关联控制在 3 个表内
4. 表关联字段尽可能使用主键
5. 禁止大表做子查询
6. 表关联排序字段必须在驱动表内
7. 表关联字段必须有索引
8. 不使用标量子查询，尽可能改为 left join 或者 right join
9. 不使用 `in/not in/exists/not exists` 子查询，尽可能改为 inner join
10. 禁止大表与大表做 join
11. OR 改写为 IN () 或者 UNION
12. 使用 union all 替代 union
13. 分页需改写为如下：`select id,text from test limit 10000,10;` --> `select b.id,b.text from (select id from test a limit 10000,10) left join test b on a.id=b.id`
14. 不建议使用前缀是 % 的 like（无法使用索引）
15. 不建议 `where` 条件里 `in （子查询）`，即使 in 的是索引字段  [参考](https://www.cnblogs.com/xh831213/archive/2012/05/09/2491272.html)

## 6. 建表实例

`1_ddl_新建 demo 表.sql`

```sql
CREATE TABLE `c_demo` (
  `id` bigint (11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar (50) NOT NULL default '' COMMENT ' 名称 ',
  `creator` varchar (50) NOT NULL  default ''  COMMENT ' 创建者 ',
  `is_deleted` tinyint (2) NOT NULL DEFAULT '0' COMMENT ' 是否逻辑删除 0. 否 1. 是 ',
  `create_time` datetime NOT NULL DEFAULT '2020-01-13 00:00:00' COMMENT ' 创建时间 ', -- 这里是为了兼容MySQL 5.6 之前的版本，5.6 之前的版本不能同时建两个 带有 CURRENT_TIMESTAMP 触发的字段
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT ' 更新时间 ',
  PRIMARY KEY (`id`),
  KEY `idx_name_deleted` (`name`,`is_deleted`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='zhangsan-2020.1.3 - 测试案例  '
```


## 参考

1. [知乎上关于“数据库设计时的一些细节的东西如何处理?”的问答](https://www.zhihu.com/question/39967106)
2. [阿里的 Java开发手册（嵩山版）.pdf](https://github.com/alibaba/p3c/blob/master/Java%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C%EF%BC%88%E5%B5%A9%E5%B1%B1%E7%89%88%EF%BC%89.pdf)


![微信公众号图片](https://s3.ax1x.com/2020/11/25/DdddOS.png)