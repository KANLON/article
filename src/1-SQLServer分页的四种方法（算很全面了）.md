
&emsp;&emsp;这篇博客讲的是SQL server的分页方法，用的SQL server 2012版本。下面都用pageIndex表示页数，pageSize表示一页包含的记录。并且下面涉及到具体例子的，设定查询第2页，每页含10条记录。<br/>
&emsp;&emsp;首先说一下SQL server的分页与MySQL的分页的不同，mysql的分页直接是用limit (pageIndex-1),pageSize就可以完成，但是SQL server 并没有limit关键字，只有类似limit的top关键字。所以分页起来比较麻烦。
<br/>&emsp;&emsp;SQL server分页我所知道的就只有四种：三重循环；利用max（主键）;利用row_number关键字，offset/fetch next关键字（是通过搜集网上的其他人的方法总结的，应该目前只有这四种方法的思路，其他方法都是基于此变形的）。

要查询的学生表的部分记录
![要查询的学生表的部分记录](https://i-blog.csdnimg.cn/blog_migrate/f91c1fccd553cc6d1ebc36824c5f1504.png)

### 方法一：三重循环
##### 思路
&emsp;&emsp;先取前20页，然后倒序，取倒序后前10条记录，这样就能得到分页所需要的数据，不过顺序反了，之后可以将再倒序回来，也可以不再排序了，直接交给前端排序。<br/>
&emsp;&emsp;还有一种方法也算是属于这种类型的，这里就不放代码出来了，只讲一下思路，就是先查询出前10条记录，然后用not in排除了这10条，再查询。
##### 代码实现

```sql
-- 设置执行时间开始，用来查看性能的
set statistics time on ;
-- 分页查询（通用型）
select * 
from (select top pageSize * 
from (select top (pageIndex*pageSize) * 
from student 
order by sNo asc ) -- 其中里面这层，必须指定按照升序排序，省略的话，查询出的结果是错误的。
as temp_sum_student 
order by sNo desc ) temp_order
order by sNo asc

-- 分页查询第2页，每页有10条记录
select * 
from (select top 10 * 
from (select top 20 * 
from student 
order by sNo asc ) -- 其中里面这层，必须指定按照升序排序，省略的话，查询出的结果是错误的。
as temp_sum_student 
order by sNo desc ) temp_order
order by sNo asc
;
```
#### 查询出的结果及时间
![方法一查询出的结果 ](https://i-blog.csdnimg.cn/blog_migrate/212be8e690a771d79321aa07521c550c.png)
![方法一查询出的时间](https://i-blog.csdnimg.cn/blog_migrate/09e88919b60808f9767ad1631e8557e8.png)

### 方法二：利用max（主键）
&emsp;&emsp;先top前11条行记录，然后利用max（id）得到最大的id，之后再重新再这个表查询前10条，不过要加上条件，where id>max(id)。

##### 代码实现

```sql
set statistics time on;
-- 分页查询（通用型）
select top pageSize * 
from student 
where sNo>=
(select max(sNo) 
from (select top ((pageIndex-1)*pageSize+1) sNo
from student 
order by  sNo asc) temp_max_ids) 
order by sNo;


-- 分页查询第2页，每页有10条记录
select top 10 * 
from student 
where sNo>=
(select max(sNo) 
from (select top 11 sNo
from student 
order by  sNo asc) temp_max_ids) 
order by sNo;

```
##### 查询出的结果及时间

![方法二查询出的结果 ](https://i-blog.csdnimg.cn/blog_migrate/ce5a2051c84f231b238c5ddfe44fbe56.png)
![方法二查询出的时间](https://i-blog.csdnimg.cn/blog_migrate/a09440b0bb0f3bf014a17907341e62c4.png)

### 方法三：利用row_number关键字
&emsp;&emsp;直接利用`row_number() over(order by id)`函数计算出行数，选定相应行数返回即可，不过该关键字只有在SQL server 2005版本以上才有。
##### SQL实现

```sql
set statistics time on;
-- 分页查询（通用型）
select top pageSize * 
from (select row_number() 
over(order by sno asc) as rownumber,* 
from student) temp_row
where rownumber>((pageIndex-1)*pageSize);

set statistics time on;
-- 分页查询第2页，每页有10条记录
select top 10 * 
from (select row_number() 
over(order by sno asc) as rownumber,* 
from student) temp_row
where rownumber>10;
```
##### 查询出的结果及时间
![方法三查询结果](https://i-blog.csdnimg.cn/blog_migrate/c50d9ea32a73781d72101a7d5a10615c.png)
![方法三查询时间](https://i-blog.csdnimg.cn/blog_migrate/0886d55e13969227291938f4624dad13.png)

### 第四种方法：offset /fetch next（2012版本及以上才有）
##### 代码实现

```sql
set statistics time on;
-- 分页查询（通用型）
select * from student
order by sno 
offset ((@pageIndex-1)*@pageSize) rows
fetch next @pageSize rows only;

-- 分页查询第2页，每页有10条记录
select * from student
order by sno  
offset 10 rows
fetch next 10 rows only ;
```

offset A rows ,将前A条记录舍去，fetch next B rows only ，向后在读取B条数据。
#####  结果及运行时间
![方法四查询结果](https://i-blog.csdnimg.cn/blog_migrate/b8972413c03f3b0bb6f5f49b979de98b.png)
![方法四查询结果](https://i-blog.csdnimg.cn/blog_migrate/adfe028e3de7f5c7e0ad0f60a62d06c3.png)
### 封装的存储过程
最后，我封装了一个分页的存储过程，方便大家调用，这样到时候写分页的时候，直接调用这个存储过程就可以了。

分页的存储过程

```
create procedure paging_procedure
(	@pageIndex int, -- 第几页
	@pageSize int  -- 每页包含的记录数
)
as
begin 
	select top (select @pageSize) *     -- 这里注意一下，不能直接把变量放在这里，要用select
	from (select row_number() over(order by sno) as rownumber,* 
			from student) temp_row 
	where rownumber>(@pageIndex-1)*@pageSize;
end

-- 到时候直接调用就可以了，执行如下的语句进行调用分页的存储过程
exec paging_procedure @pageIndex=2,@pageSize=10;
```

### 总结
&emsp;&emsp;根据以上四种分页的方法执行的时间可以知道，以上四种分页方法中，第二，第三，第三四种方法性能是差不多的，但是第一种性能很差，不推荐使用。还有就是这篇博客这是测试了小量数据，还没有分页大量数据，所以不清楚在大量数据要分页时哪种方法的性能更加好。我这里推荐第四种，毕竟第四种是SQL server公司升级后推出的新方法，所以应该理论上性能和可读性都会更加好。




