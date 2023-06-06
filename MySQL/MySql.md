# 一、SQL

## 1.SQL通用语法

lue

## 2.SQL分类

<img title="" src="file:///C:/Users/Aqi/AppData/Roaming/marktext/images/2023-04-24-12-05-05-image.png" alt="" width="718">

 数据类型：

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-12-28-33-image.png)

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-12-32-17-image.png)

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-12-36-12-image.png)

### 2.1 DDL

数据库操作：

```sql
-- 查询所有数据库
SHOW DATABASES;

-- 查询当前数据库
SELECT DATABASE();

-- 创建数据库
CREATE DATABASE [IF NOT EXISTS] 数据库名 
[DEFAULT CHARSET 字符集] [COLLATE 排序规则];
/* 
    [IF NOT EXISTS] 数据库不存在则创建，反之不创建
    [DEFAULT CHARSET 字符集] 字符集
    [COLLATE 排序规则] 制定排序规则 
   eg：
    CREATE DATABASE IF NOT EXISTS sqltest DEFAULT CHARSET utf8mb4;
*/

-- 删除数据库
DROP DATABASE [IF EXISTS] 数据库名;

-- 使用数据库
USE 数据库名;
```

表操作：

```sql
-- 查询当前数据库所有表
SHOW TABLES；

-- 查询表结构：
DESC 表名;

-- 查询指定表的建表语句
SHOW CREATE TABLE 表名;

-- 表创建
CREATE TABLE 表名(
    字段1 字段1类型 [COMMENT 字段1注释],
    字段2 字段2类型 [COMMENT 字段2注释],
    ...
    字段n 字段n类型 [COMMENT 字段n注释]
) [COMMENT 表注释];

-- 表添加字段
ALTER TABLE 表名 ADD 字段名 类型(长度) [COMMENT 注释] [约束];

-- 表修改字段数据类型
ALTER TABLE 表名 MODIFY 字段名 新数据类型(长度);

-- 表修改字段名和字段类型
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型(长度) [COMMENT 注释] [约束];

-- 删除字段
ALTER TABLE 表名 DROP 字段名;

-- 修改表名
ALTER TABLE 表名 RENAME TO 新表名;

-- 删除表
DROP TABLE [IF EXISTS] 表名;

-- 删除指定表，并重新创建该表（初始化）
TRUNCATE TABLE 表名;
```

### 2.2 DML

添加数据：

```sql
-- 给指定字段添加数据
INSERT INTO 表名 (字段名1，字段名2...) 
VALUES (值1，值2...);

-- 给全部字段添加数据
INSERT INTO 表名
VALUES (值1，值2...);

-- 批量添加数据
INSERT INTO 表名 (字段名1，字段名2...) 
VALUES (值1，值2...),(值1，值2...),(值1，值2...),...;

INSERT INTO 表名
VALUES (值1，值2...),(值1，值2...),(值1，值2...),...;
```

修改数据：

```sql
-- 修改数据
UPDATE 表名 SET 
    字段名1 = 值1,
    字段名2 = 值2,
    ...
    字段名n = 值n
[WHERE 条件]; -- 如果没有条件，则修改整张表的数据
删除
```

删除数据：

```sql
DELETE FROM 表名 [WHERE 条件]
-- 如果没有条件，则删除整张表的数据
```

### 2.3 DQL

查询数据

```sql
-- 查询数据
SELECT 字段列表 FROM 表名列表
WHERE 条件列表
GROUP BY 分组字段列表 HAVING 分组后条件列表
ORDER BY 排序
LIMIT 分页参数

-- 设置别名
SELECT 字段1 [AS 别名1],字段2 [AS 别名2], ... FROM 表名;

-- 去除重复
SELECT DISTINCT 字段列表 FROM 表名;

-- 聚合函数
/*
    概念：将一列数据作为一个整体，进行纵向计算
    包括：count（统计数量）、max（最大值）、min（最小值）、avg（平均值）、sum（和）
    注意：null值不参与聚合函数的计算
*/
SELECT 聚合函数(字段列表) FROM 表名;

-- 分组查询
/*    WHERE和HAVING的区别
    1.执行时机不同: WHERE是分组之前过滤的，不满足WHERE条件，不参与分组；
                  HAVING是分组之后对结果再进行过滤。
    2.判断条件不同: WHERE不能对聚合条件进行判断；
                  HAVING能对聚合条件进行判断。
*/
SELECT 字段列表 FROM 表名 [WHERE 条件] 
GROUP BY 分组字段名 [HAVING 分组后过滤条件]; 
    -- eg1：根据性别分组，统计男员工和女员工的数量
    SELECT gender,count(*) FROM test GROUP BY gender;
    -- eg2: 根据性别分组，统计男员工和女员工的平均年龄
    SELECT gender,avg(age) FROM test GROUP BY gender;
    -- eg3: 查询年龄小于45的员工，并根据工作地址分组，获取员工数量大于等于3的工作地址
    SELECT address,count(*) address_count FROM test 
    WHERE age<45 
    GROUP BY address HAVING address_count  >= 3;

-- 排序查询
/*    排序方式（两种）  
         1.ASC(默认):升序
         2.DESC:降序
*/
SELECT 字段列表 FROM 表名 ORDER BY 字段1 排序方式1,字段2 排序方式2;

-- 分页查询
SELECT 字段列表 FROM 表名 LIMIT 起始索引,查询记录数;
```

DQL实例：

```sql
-- DQL示例
-- eg1:查询年龄为20、21、23、26的女性员工信息
SELECT * FROM test WHERE gender='女' AND age IN(20,21,23,26);
-- eg2:查询性别为男，年龄在20~40且名字为三个字的员工
SELECT * FROM test WHERE gender='男' 
AND age BETWEEN 20 AND 40
AND name LIKE '___';
-- eg3:统计员工表中，年龄小于60岁的，男性员工和女性员工的人数
SELECT gender, count(*) FROM test
WHERE age < 60
GROUP BY gender;
-- eg4:查询所有年龄小于等于35岁员工的姓名和年龄，并对查询结果按年龄升序排序
--     如果年龄相同按入职时间降序排序
SELECT name,age FROM test
WHERE age<=35
ORDER BY age ASC, time DESC;
-- eg5:查询性别为男，且年龄在20~40岁以内的前五个员工的信息
--     对结果按年龄升序排序，年龄相同按入职时间升序排序
SELECT * FROM test
WHERE gender = '男' AND age BETWEEN 20 AND 40
ORDER BY age ASC, time ASC
LIMIT 0,5;
```

条件：

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-14-42-26-image.png)

 DQL执行顺序：

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-16-07-22-image.png)

### 2.4 DCL

管理用户：

```sql
-- 查询用户
USE mysql;
SELECT * FROM user;

-- 创建用户
CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';

-- 修改用户密码
ALTER USER '用户名'@'主机名' 
IDENTIFIED WITH mysql_native_password BY '新密码';

--删除用户
DROP USER '用户名'@'主机名';

-- eg1:创建用户usertest1，只能够在当前主机localhost访问，密码123456
CREAT USER 'usertest1'@'localhost' IDENTIFIED BY '123456';
-- eg2:创建用户usertest2，可以在任意主机访问该数据库，密码123456
CREAT USER 'usertest2'@'%' IDENTIFIED BY '123456';
-- eg3:修改用户usertest1密码为1234
ALTE USER 'usertest1'@'localhost' IDENTIFIED WITH
mysql_native_password BY '1234';
```

权限控制：

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-17-44-37-image.png)

控制用户权限：

```sql
-- 查询权限
SHOW GRANTS FOR '用户名'@'主机名';

-- 授予权限
GRANT 权限列表 ON 数据库名.表名 TO '用户名'@'主机名'; -- 如果是全部则 *.*

-- 撤销权限
REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@'主机名';
```

# 二、函数

## 1.字符串函数

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-18-04-16-image.png)

实例：

```sql
-- CONCAT
SELECT concat('Hello',' Mysql');
-- eg1:由于业务需求变更，企业员工的工号统一为5位数，
--     目前不足5位数的全部在前面补0。1->00001
UPDATE test set workno = lpad(workno, 5, 0);
```

## 2.数值函数

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-18-11-45-image.png)

```sql
-- eg:生成六位数随机数
SELECT lpad(round(rand()*1000000,0),6,0);
```

## 3.日期函数

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-18-15-25-image.png)

## 4.流程函数

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-18-17-07-image.png)

# 三、约束

## 1.概述

概念：约束是作用于表中字段上的规则，用于限制存储在表中的数据；

目的：保证数据库中数据的正确、有效性和完整性；

分类：

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-18-20-13-image.png)

```sql
-- check
...
    age int check(age>0 && age<=120),
...
-- default
...
    status char(1) default '1' ,
...
```

## 2.外键约束

概念：外键用来让两张表的数据之间建立连接，从而保证数据的一致性和完整性。

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-18-49-29-image.png)

```sql
-- 添加外键
-- 建表
CREATE TABLE 表名(
    字段名 数据类型,
    ...
    [CONSTRAINT] [外键名称] FOREIGN KEY(外键字段名) REFERENCES 主表(主表列名);
)

-- 改表
ALTER TABLE 表名 
ADD CONSTRAINT 外键名称 
FOREIGN KEY (外键字段名) REFERENCES 主表(主表字段名);

-- 删除外键
ALTER TABLE 表名 
DROP FOREIGN KEY 外键名称;
```

删除/更新行为：

![](C:\Users\Aqi\AppData\Roaming\marktext\images\2023-04-24-19-27-16-image.png)

```mysql
-- 添加外键约束
ALTER TABLE 表名 
ADD CONSTRAINT 外键名称 
FOREIGN KEY (外键字段名) REFERENCES 主表(主表字段名)
ON UPDATE CASCADE
ON DELETE CASCADE;
/*
    ON UPDATE CASCADE : 更新时执行CASCADE操作
    ON DELETE CASCADE ：删除时执行CASCADE操作
*/
```

# 四、多表查询

## 1.多表关系

​	项目开发中，在进行数据库表结构设计时，会根据业务需求及业务模块之间的关系，分析并设计表结构，由于业务之间相互关联，所以各个表结构之间也存在着各种联系，基本上分为三种:

- 一对多
- 多对多
- 一对一

## 2.多表查询概述

多表查询分类：

- 连接查询
  - 内连接：相当于查询A、B交集部分数据
  - 外连接：
    - 左外连接：查询左表所有数据，以及两张表交集部分数据
    - 右外连接：查询右表所有数据，以及两张表交集部分数据
  - 自连接：当前表与自身的连接查询，自连接必须使用表别名
- 子查询

## 3.内连接

内连接查询语法：

- 隐式内连接

```sql
SELECT 字段列表 FROM 表1,表2 WHERE 条件...;
```

- 显式内连接

```sql
SELECT 字段列表 FROM 表1 [INNER] JOIN 表2 ON 条件...;
```

案例：

- 查询每一个员工的姓名以及关联部门的名称（隐式内连接完成）

```sql
SELECT emp.name,dept.name FROM emp,dept WHERE emp.dept_id = dept.id;

-- 起别名
SELECT e.name,d.name FROM emp e,dept d WHERE e.dept_id = d.id;
```

- 查询每一个员工的姓名以及关联部门的名称（显式内连接完成）

```sql
SELECT e.name,d.name FROM emp e INNER JOIN dept d ON e.dept_id = d.id;
```

## 4.外连接

外连接查询语法

- 左外连接：查询表1（左表）的所有数据以及包含表1表2交集部分数据

```sql
SELECT 字段列表 FROM 表1 LEFT [OUTER] JOIN 表2 ON 条件...;
```

- 右外连接：查询表2（右表）的所有数据以及包含表1表2交集部分数据

```sql
SELECT 字段列表 FROM 表1 RIGHT [OUTER] JOIN 表2 ON 条件...;
```

案例：

- 查询emp表的所有数据，和对应的部门信息（左外连接）

```sql
SELECT e.*,d.name FROM emp e LEFT OUTER JOIN dept d ON e.dept_id = d.id;
```

- 查询dept表的所有数据，和对应的员工信息（右外连接）

```sql
SELECT d.*,e.* FROM emp e RIGHT OUTER JOIN dept d ON e.dept_id = d.id;
```

## 5.自连接

自连接查询语法：

```sql
SELECT 字段列表 FROM 表1 别名1 JOIN 表1 别名2 ON 条件...;
```

案例：

- 查询员工及其领导的名字

```sql
SELECT a.name,b.name FROM emp a, emp b WHERE a.managerid = b.id;
```

## 6.联合查询

联合查询——union、union all：对于联合查询，就是把多次查询的结果合并起来，形成一个新的查询结果集

语法：

```sql
SELECT 字段列表 FROM 表A ...
UNION [ALL]
SELECT 字段列表 FROM 表B ...
```

- UNION ALL：直接查询合并，有可能出现重复
- UNION：查询加去重

案例：

- 将工资低于5000的员工 和 年龄大于50岁的员工全部查询出来

```sql
SELECT * FROM emp WHERE salary < 5000
UNION ALL
SELECT * FROM emp WHERE age > 50
```

注意：

​		对于联合查询的多张表的列数必须保持一致，字段类型也需要保持一致。

## 7.子查询

概念：SQL语句中嵌套SELECT语句，被称为嵌套查询，又称子查询。

```sql
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2);
```

​		子查询外部语句可以是增删改查的任意一个。

根据子查询结果不同，分为：

- 标量子查询（子查询结果为单个值）
- 列子查询（子查询结果为单个列）
- 行子查询（子查询结果为单个行）
- 表子查询（子查询结果为多行多列）

### 7.1标量子查询

标量子查询：

- 子查询返回的值是单个值（数字、字符串、日期等），最简单的形式，这种子查询称为标量子查询
- 常用的操作符：= <> > >= < <=

案例：

- 查询销售部的所有员工信息

```sql
SELECT * FROM emp WHERE dept_id = (SELECT id FROM dept WHERE name = '销售部');
```

- 查询在”东方不败“入职之后的员工信息

```sql
SELECT * FROM emp WHERE entrydate > (SELECT entrydate FROM emp WHERE name = '东方不败');
```

### 7.2列子查询

列子查询：

- 子查询返回的结果是一列（或者多行）
- 常用操作符：IN、NOT IN、ANY、SOME、ALL

![image-20230510185033959](G:\笔记\MySQL\assets\image-20230510185033959.png)

案例：

- 查询销售部和市场部的所有员工信息

```sql
SELECT * FROM emp WHERE dept_id in (SELECT id FROM dept WHERE name = '销售部' OR name = '市场部');
```

- 查询比财务部所有人工资都高的员工信息

```sql
SELECT * FROM emp WHERE salary > ALL (SELECT salary FROM emp WHERE dept_id = (SELECT id FROM dept WHERE name = '财务部'));
```

### 7.3行子查询

行子查询：

- 子查询结果返回的是一行（或多列）
- 常用操作符：=、<>、IN、NOT IN

案例：

- 查询与张无忌的薪资及其直属领导相同的员工信息

```sql
SELECT * FROM emp 
WHERE (salary,managerid) = 
	(SELECT salary,managerid FROM emp WHERE name = '张无忌');
```

### 7.4表子查询

表子查询：

- 返回结果为多行多列
- 操作符：IN

案例：

- 查询与鹿杖客、宋远桥的职位和薪资相同的员工信息

```sql
SELECT * FROM emp
WHERE (job,salary) 
IN (SELECT job,salary FROM emp WHERE name ='鹿杖客' or name ='宋远桥');
```

- 查询入职日期在“2006-01-01”之后的员工信息，及其部门信息

```sql
SELECT * FROM (SELECT * FROM emp WHERE enrtydata > '2006-01-01') e
LEFT JOIN dept d
ON e.dept_id = d.id;
```



# 五、事务

​		事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

## 1.事务操作

- 查看/设置事务提交方式：

```sql
SELECT @@autocommit;
SET @@autocommit = 0;
```

- 提交事务

```sql
COMMIT;
```

- 回滚事务

```sql
ROLLBACK;
```

- 开启事务

```sql
START TRANSACTION;
BEGIN;
```

## 2.四大特性（ACID）

- 原子性（Atomicity）：事务是不可分割的最小操作单元，要么同时成功，要么同时失败
- 一致性（Consistency）：事务完成时，必须是所有的数据都保持一致状态
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的

## 3.并发事务问题

![image-20230512141932190](G:\笔记\MySQL\assets\image-20230512141932190.png)

 ## 4.事务隔离级别

![image-20230512142621498](G:\笔记\MySQL\assets\image-20230512142621498.png)

- 查看事务隔离级别

```sql
SELECT @@TRANSACTION_ISOLATION;
```

- 设置事务隔离级别

```sql
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL
{READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
```



















































