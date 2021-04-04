---
title: MySQL
date: 2018-09-10 18:27:21
tags: MySQL
---

### 基础语法

#### left jon

 **left join is a short hand for left outer join - they are the same.**

左外链接  left outer join的简写是 left join。会将左边表里的全部查出来，只显示右边表里与左边表里对应有的数据。on为合并条件

右外链接 right (outer) join  ：与左外相反

全外链接：full (outer) join :显示左右两边的数据

```sql
select Person.FirstName,Person.LastName,Address.City,Address.State from Person left join 
Address
on Person.PersonId = Address.PersonId
```

如果将left join 换成join会查询两张表都有的数据

```sql
select p1.id,p1.name,p1.address,p2.password from tb_customer p1  JOIN tb_user p2 ON	p2.id = p1.id;
```

```sql

select  x.p_key,x.p_value,y.p_value
from tb_config_prop as x,tb_config_prop as y,tb_config_prop as z
where x.config_meta_id=7 and y.config_meta_id = 127 and z.config_meta_id = 128
and x.p_key = y.p_key
;
```



#### 176. Second Highest Salary

```sql
SELECT DISTINCT
    Salary AS SecondHighestSalary
FROM
    Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1
```

如果只有一条记录的话就会报错。因为不存在

可以将其作为一个临时表

```sql
SELECT
    (SELECT DISTINCT
            Salary
        FROM
            Employee
        ORDER BY Salary DESC
        LIMIT 1 OFFSET 1) AS SecondHighestSalary
;
```

distinct 去除重复值

desc 降序

或者添加判断

```sql
SELECT
    IFNULL(
      (SELECT DISTINCT Salary
       FROM Employee
       ORDER BY Salary DESC
        LIMIT 1 OFFSET 1),
    NULL) AS SecondHighestSalary
```

#### 595. Big Countries

select name,population,area from World where population > 25000000 
union
select name,population,area from World where area > 3000000;

#### 627. Swap Salary

**explain**：更新数据库表，如果值为多少就改为多少，否则.....

```sql
update tb_address set city = CASE city
	WHEN '北京' THEN
		'上海'
	when '上海' then
		'湖南'
	ELSE
		'南京'
END;
```

上例中，当tb_address表的 city字段，值为'北京' 时，将值设置为'上海'；值为'上海'的，设置为'湖南'。其他的设置为'南京'。可以多加几个when   … then的判断条件。END为结尾，不需要加 END CASE;

#### **TO_DAYS**

**TO_DAYS(wt1.DATE)** return the number of days between from year 0 to date DATE

会将日期变为一个数字，日期越大数字越大

Given a `Weather` table, write a SQL query to find all dates' Ids with higher temperature compared to its previous (yesterday's) dates.

```
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
```

For example, return the following Ids for the above `Weather` table:

```
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```

```sql
SELECT wt1.Id 
FROM Weather wt1, Weather wt2
WHERE wt1.Temperature > wt2.Temperature AND 
      TO_DAYS(wt1.DATE)-TO_DAYS(wt2.DATE)=1;
```

#### 196. Delete Duplicate Emails

Write a SQL query to **delete** all duplicate email entries in a table named `Person`, keeping only unique emails based on its *smallest* **Id**.

```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Id is the primary key column for this table.
```

For example, after running your query, the above `Person` table should have the following rows:

```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```

```SQL
DELETE p1
FROM Person p1, Person p2
WHERE p1.Email = p2.Email AND
p1.Id > p2.Id
```

#### 177. Nth Highest Salary

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
DECLARE M INT;
SET M=N-1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT M, 1
  );
END
```

### MySql遇到的问题

#### sql_mode=only_full_group_by的完美解决方案

```sql
--	1、查看sql_mode
select @@sql_mode

-- 查询出来的值为：

ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

-- 2、去掉ONLY_FULL_GROUP_BY，重新设置值。

set @@sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

-- 3、上面是改变了全局sql_mode，对于新建的数据库有效。对于已存在的数据库，则需要在对应的数据下执行：

set sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

```





