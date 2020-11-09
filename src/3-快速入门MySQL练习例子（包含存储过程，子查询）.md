&emsp;&emsp;这篇博客主要是用来展示一个MySQL练习的例子，通过练习这个例子可以达到入门MySQL的效果

### 问题/需求

1. 需求：
	设计数据库表存储：（用户考试信息）
			用户信息、考试时间、考试科目与考试成绩，及所属年级！

2. 测试数据：
	U001, 张三,1985-09-09, 广州天河,  
     java,80,基础班,  考试时间为2014-01-01  
     jsp,90,就业班,  考试时间为2014-03-01
    mysql,90, 就业班, 考试时间为2014-04-04 
	U002,李四,1995-09-09, 广州越秀,
    java,67,基础班, 考试时间为2014-01-01
    mysql,90, 就业班, 考试时间为2014-04-04 
	……….(录入其他记录)

	  提示： 最好是四个表（使用约束）


3. 查询需求：
	- 查询学号是U001的学生参加2014-01-01 “java”课程考试的成绩，要求输出学生姓名和成绩
	- 查询出通过考试（高于60分）的学员所在的姓名、、所属学学习阶段、考试科目名称、学员的成绩。
	- 利用子查询语句，筛选出生日期比“李四”大的学生
	- 查询“java”课程考试成绩为60-80分的学生名单
	- 查询参加最近一次“mysql”考试成绩最高分和最低分
	- 查询出基础班考试的平均成绩；
	- 需求（存储过程）
	- 统计并显示2014-04-04的mysql考试平均分
	- 如果平均分在70以上，显示“考试成绩优秀”
	- 如果在70以下，显示“考试成绩较差”


### 答案

MySQL的 SQL语句如下：
```
-- 创建数据库
CREATE DATABASE IF NOT EXISTS test_message CHARACTER SET utf8;
USE test_message;
DROP TABLE user_test;
-- 创建用户考试信息表
CREATE TABLE user_test(
	id INT PRIMARY KEY AUTO_INCREMENT,
	user_id VARCHAR(8),
	test_date DATE,
	course_id INT,                            -- 外键，考试科目的
	math FLOAT(5,2),                            -- 考试成绩
	grade_id INT
);
DROP TABLE IF EXISTS course ;
-- 考试科目表
CREATE TABLE course(
	id INT PRIMARY KEY,
	course_name VARCHAR(40)
);
DROP TABLE IF EXISTS grade;
-- 年级表
CREATE TABLE grade(
	id INT PRIMARY KEY,
	grade_name VARCHAR(5)
);
DROP TABLE IF EXISTS user_message;
-- 用户信息表
CREATE TABLE user_message(
	id VARCHAR(8) PRIMARY KEY,
	user_name VARCHAR(10),
	user_birthday DATE,
	zone VARCHAR(20)
);
-- 创建用户考试信息表和用户表的外键
ALTER TABLE user_test ADD CONSTRAINT user_test_user_message_fk FOREIGN KEY(user_id) REFERENCES user_message(id);
-- 创建用户考试信息表和年级的外键
ALTER TABLE user_test ADD CONSTRAINT user_test_grade_fk FOREIGN KEY(grade_id) REFERENCES grade(id);
-- 创建用户考试信息表和科目的外键
ALTER TABLE user_test ADD CONSTRAINT user_test_course_fk FOREIGN KEY(course_id) REFERENCES course(id);
-- 插入用户表的数据
INSERT INTO user_message
            (id,
             user_name,
             user_birthday,
             zone)
VALUES ('U001',
        '张三',
        '1985-09-09',
        '广州天河'),
       ('U002',
        '李四',
        '1995-09-09',
        '广州越秀'),
        ('U003',
        '王五',
        '1996-09-08',
        '广州番禺');
-- 插入年级的数据
INSERT INTO grade
            (id,
             grade_name)
VALUES (1,
        '基础班'),
       (2,
        '就业班'),
        (3,
        '进阶版');
-- 插入科目表的数据
INSERT INTO course(id,course_name) VALUES(1,'java'),(2,'jsp'),(3,'mysql'),(4,'ajax'),(5,'html');

-- 插入用户考试信息表的数据
INSERT INTO user_test(
	user_id,
	test_date,
	course_id,                            -- 外键，考试科目的
	math,                            -- 考试成绩
	grade_id
	) VALUES(
	'U001',
	'2014-01-01',
	'1',
	'80.00',
	'1'),(
	'U001',
	'2014-03-01',
	'2',
	'90',
	'2'),(
	'U001',
	'2014-04-04',
	'3',
	'85',
	'2'),(
	'U002',
	'2014-01-01',
	'1',
	'67',
	'1'),(
	'U002',
	'2014-04-04',
	'3',
	'90',
	'2'),(
	'U002',
	'2014-05-04',
	'2',
	'80',
	'2'),(
	'U003',
	'2014-06-04',
	'1',
	'80',
	'1'),(
	'U001',
	'2014-01-01',
	'2',
	'90',
	2),(
	'U001',
	'2014-04-04',
	'3',
	'96',
	2);
-- 查询用户考试信息表
SELECT * FROM user_test;

DELETE FROM user_test;

-- 查询需求
-- 1. 查询学号是U001的学生参加2014-01-01 “java”课程考试的成绩，要求输出学生姓名和成绩
-- 方法一
SELECT user_name,math FROM user_test,user_message,course WHERE user_message.id=user_test.user_id
AND user_test.course_id=course.id
AND user_test.user_id='U001' 
AND user_test.test_date='2014-01-01' 
AND course.course_name='java';
-- 方法二
SELECT user_name,math FROM user_test INNER JOIN user_message INNER JOIN course ON user_test.user_id=user_message.id
AND user_test.course_id=course.id
AND user_test.user_id='U001' 
AND user_test.test_date='2014-01-01' 
AND course.course_name='java';

-- 查询出通过考试（高于80分）的学员所在的姓名、、所属学学习阶段、考试科目名称、学员的成绩。
-- 用别名
SELECT user_name,grade_name,course_name,math FROM user_test t,user_message m,course c,grade g
WHERE t.user_id=m.id
AND t.course_id=c.id
AND t.grade_id=g.id
AND t.math>80;


-- 3利用子查询语句，筛选出生日期比“李四”大的学生
SELECT user_name FROM user_message WHERE user_birthday>(SELECT user_birthday FROM user_message WHERE user_name='李四');
SELECT * FROM user_message;

-- 4查询“java”课程考试成绩为60-80分的学生名单
SELECT user_name FROM user_test t,user_message m,course c WHERE t.course_id=c.id 
AND t.user_id=m.id
AND c.course_name='java'
AND t.math>=60
AND t.math<80;

-- 5查询参加最近一次“mysql”考试成绩最高分和最低分
SELECT MAX(math),MIN(math) FROM user_test t INNER JOIN course c ON t.course_id=c.id 
AND c.course_name='mysql'
AND test_date=(SELECT MAX(test_date) FROM user_test t INNER JOIN course c ON t.course_id=c.id 
AND c.course_name='mysql') 
ORDER BY math DESC;

-- 6.查询出基础班考试的平均成绩；
SELECT AVG(math) FROM user_test t INNER JOIN grade g ON t.grade_id=g.id
AND g.grade_name='基础班';
-- 分组查询
SELECT grade_name,AVG(math) FROM user_test t,grade g  WHERE t.grade_id=g.id GROUP BY g.grade_name; 
-- 7需求（存储过程）
-- 7.1统计并显示2014-04-04的mysql考试平均分

-- 7.2如果平均分在70以上，显示“考试成绩优秀”

-- 7.3 如果在70以下，显示“考试成绩较差”

-- 删除存储过程
DROP PROCEDURE pro_test;
-- 创建存储过程
DELIMITER $
CREATE PROCEDURE pro_test(OUT avg_math DOUBLE,OUT str VARCHAR(6))
BEGIN
  SELECT AVG(math) INTO avg_math FROM user_test t,course c WHERE t.course_id=c.id 
  AND t.test_date='2014-04-04'
  AND c.course_name='mysql';
  
  IF avg_math>=70 THEN
     SET str='考试成绩优秀';
  ELSE
     SET str='考试成绩较差';
  END IF;
END $
-- 执行存储过程
CALL pro_test(@avg_math,@str);
SELECT @avg_math,@str;
```