# SQL转ES的DSL查询语法和SQL的简单解析


之前有个功能需要用到


 ES 官方的X-Pack 插件（后来统一叫做Elastic Stack）
 https://www.elastic.co/cn/subscriptions      ![ES 官方的X-Pack 插件](https://img-blog.csdnimg.cn/18e7d223cc5f43f797c89a55a32e7064.png)
 
 Elasticsearch-SQL 插件
https://github.com/NLPchina/elasticsearch-sql

已经不再维护

>Please note, this project is no longer in active development, and is deprecated, please use official version x-pack-sql and OpenDistro for Elasticsearch SQL supported by AWS and licensed under Apache 2.


GitHub cch123/elastic

![elastic的github的start和frok数](https://img-blog.csdnimg.cn/84c3b45433d843ca83f28beaf37a8667.png)




 自研 Druid SQL解析成AST 转ES的DSL（可控，灵活）

`AST 为抽象语法树（Abstract Syntax Tree）`

`DSL领域特定语言Domain Specific Language`

```sql
select * from test_t 
where 
user_id = 1 
and (  
product_id = 2 
and (star_num = 4 or star_num = 5) 
and banned = 1)
```
![SQL的AST](https://img-blog.csdnimg.cn/f276dde634274d3d8abc8a62baf001ce.png)



参考：
1. AST抽象语法树——最基础的javascript重点知识，99%的人根本不了解 https://segmentfault.com/a/1190000016231512