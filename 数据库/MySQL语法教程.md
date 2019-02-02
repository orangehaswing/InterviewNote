# SQL教程

这是 [在线文档](http://www.runoob.com/sql/sql-tutorial.html)

# SQL 快速参考

------

| SQL 语句          | 语法                                       |
| --------------- | ---------------------------------------- |
| AND / OR        | SELECT column_name(s)FROM table_nameWHERE conditionAND\|OR condition |
| ALTER TABLE     | ALTER TABLE table_name ADD column_name datatypeorALTER TABLE table_name DROP COLUMN column_name |
| AS (alias)      | SELECT column_name AS column_aliasFROM table_nameorSELECT column_nameFROM table_name AS table_alias |
| BETWEEN         | SELECT column_name(s)FROM table_nameWHERE column_nameBETWEEN value1 AND value2 |
| CREATE DATABASE | CREATE DATABASE database_name            |
| CREATE TABLE    | CREATE TABLE table_name(column_name1 data_type,column_name2 data_type,column_name2 data_type,...) |
| CREATE INDEX    | CREATE INDEX index_nameON table_name (column_name)orCREATE UNIQUE INDEX index_nameON table_name (column_name) |
| CREATE VIEW     | CREATE VIEW view_name ASSELECT column_name(s)FROM table_nameWHERE condition |
| DELETE          | DELETE FROM table_nameWHERE some_column=some_valueorDELETE FROM table_name (**Note: **Deletes the entire table!!)DELETE * FROM table_name (**Note: **Deletes the entire table!!) |
| DROP DATABASE   | DROP DATABASE database_name              |
| DROP INDEX      | DROP INDEX table_name.index_name (SQL Server)DROP INDEX index_name ON table_name (MS Access)DROP INDEX index_name (DB2/Oracle)ALTER TABLE table_nameDROP INDEX index_name (MySQL) |
| DROP TABLE      | DROP TABLE table_name                    |
| GROUP BY        | SELECT column_name, aggregate_function(column_name)FROM table_nameWHERE column_name operator valueGROUP BY column_name |
| HAVING          | SELECT column_name, aggregate_function(column_name)FROM table_nameWHERE column_name operator valueGROUP BY column_nameHAVING aggregate_function(column_name) operator value |
| IN              | SELECT column_name(s)FROM table_nameWHERE column_nameIN (value1,value2,..) |
| INSERT INTO     | INSERT INTO table_nameVALUES (value1, value2, value3,....)*or*INSERT INTO table_name(column1, column2, column3,...)VALUES (value1, value2, value3,....) |
| INNER JOIN      | SELECT column_name(s)FROM table_name1INNER JOIN table_name2 ON table_name1.column_name=table_name2.column_name |
| LEFT JOIN       | SELECT column_name(s)FROM table_name1LEFT JOIN table_name2 ON table_name1.column_name=table_name2.column_name |
| RIGHT JOIN      | SELECT column_name(s)FROM table_name1RIGHT JOIN table_name2 ON table_name1.column_name=table_name2.column_name |
| FULL JOIN       | SELECT column_name(s)FROM table_name1FULL JOIN table_name2 ON table_name1.column_name=table_name2.column_name |
| LIKE            | SELECT column_name(s)FROM table_nameWHERE column_name LIKE pattern |
| ORDER BY        | SELECT column_name(s)FROM table_nameORDER BY column_name [ASC\|DESC] |
| SELECT          | SELECT column_name(s)FROM table_name     |
| SELECT *        | SELECT *FROM table_name                  |
| SELECT DISTINCT | SELECT DISTINCT column_name(s)FROM table_name |
| SELECT INTO     | SELECT *INTO new_table_name [IN externaldatabase]FROM old_table_name*or*SELECT column_name(s)INTO new_table_name [IN externaldatabase]FROM old_table_name |
| SELECT TOP      | SELECT TOP number\|percent column_name(s)FROM table_name |
| TRUNCATE TABLE  | TRUNCATE TABLE table_name                |
| UNION           | SELECT column_name(s) FROM table_name1UNIONSELECT column_name(s) FROM table_name2 |
| UNION ALL       | SELECT column_name(s) FROM table_name1UNION ALLSELECT column_name(s) FROM table_name2 |
| UPDATE          | UPDATE table_nameSET column1=value, column2=value,...WHERE some_column=some_value |
| WHERE           | SELECT column_name(s)FROM table_nameWHERE column_name operator value |



## 重要的 SQL 命令

- **SELECT** - 从数据库中提取数据
- **UPDATE** - 更新数据库中的数据
- **DELETE** - 从数据库中删除数据
- **INSERT INTO** - 向数据库中插入新数据
- **CREATE DATABASE** - 创建新数据库
- **ALTER DATABASE** - 修改数据库
- **CREATE TABLE** - 创建新表
- **ALTER TABLE** - 变更（改变）数据库表
- **DROP TABLE** - 删除表
- **CREATE INDEX** - 创建索引（搜索键）
- **DROP INDEX** - 删除索引



### SQL SELECT 语法

```
SELECT column_name,column_name FROM table_name;
SELECT * FROM table_name;
```

关键词：

DISTINCT 关键词用于返回唯一不同的值。

```
SELECT DISTINCT column_name,column_name FROM table_name; 
```

WHERE 子句用于提取那些满足指定标准的记录。

```
SELECT column_name,column_name FROM table_name WHERE column_name operator value;

WHERE 子句中的运算符
运算符	描述
=	等于
<>	不等于。注释：在 SQL 的一些版本中，该操作符可被写成 !=
>	大于
<	小于
>=	大于等于
<=	小于等于
BETWEEN	在某个范围内
LIKE	搜索某种模式
IN	指定针对某个列的多个可能值
```

WHERE笔记

```
Where 子句
搜索 empno 等于 7900 的数据：
Select * from emp where empno=7900;

Where +条件（筛选行）
条件：列，比较运算符，值
比较运算符包涵：= > < >= ,<=, !=,<> 表示（不等于）
Select * from emp where ename='SMITH';
例子中的 SMITH 用单引号引起来，表示是字符串，字符串要区分大小写。

逻辑运算
And:与 同时满足两个条件的值。
Select * from emp where sal > 2000 and sal < 3000;
查询 EMP 表中 SAL 列中大于 2000 小于 3000 的值。

Or:或 满足其中一个条件的值
Select * from emp where sal > 2000 or comm > 500;
查询 emp 表中 SAL 大于 2000 或 COMM 大于500的值。

Not:非 满足不包含该条件的值。
select * from emp where not sal > 1500;
查询EMP表中 sal 小于等于 1500 的值。

逻辑运算的优先级：
 not        and         or
 
特殊条件
1.空值判断： is null
Select * from emp where comm is null;
查询 emp 表中 comm 列中的空值。

2.between and (在 之间的值)
Select * from emp where sal between 1500 and 3000;
查询 emp 表中 SAL 列中大于 1500 的小于 3000 的值。
注意：大于等于 1500 且小于等于 3000， 1500 为下限，3000 为上限，下限在前，上限在后，查询的范围包涵有上下限的值。

3.In
Select * from emp where sal in (5000,3000,1500);
查询 EMP 表 SAL 列中等于 5000，3000，1500 的值。

4.like
Like模糊查询
Select * from emp where ename like 'M%';
查询 EMP 表中 Ename 列中有 M 的值，M 为要查询内容中的模糊信息。

 % 表示多个字值，_ 下划线表示一个字符；
 M% : 为能配符，正则表达式，表示的意思为模糊查询信息为 M 开头的。
 %M% : 表示查询包含M的所有内容。
 %M_ : 表示查询以M在倒数第二位的所有内容
 
 不带比较运算符的 WHERE 子句：

WHERE 子句并不一定带比较运算符，当不带运算符时，会执行一个隐式转换。当 0 时转化为 false，1 转化为 true。例如：

SELECT studentNO FROM student WHERE 0
则会返回一个空集，因为每一行记录 WHERE 都返回 false。

SELECT  studentNO  FROM student WHERE 1
返回 student 表所有行中 studentNO 列的值。因为每一行记录 WHERE 都返回 true。
```



## AND & OR 运算符

如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。

如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。



## ORDER BY 关键字

ORDER BY 关键字用于对结果集按照一个列或者多个列进行排序。

ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 DESC 关键字。

```
SELECT column_name,column_name 
FROM table_name 
ORDER BY column_name,column_name ASC|DESC;
```

ORDER BY笔记

ORDER BY 多列的时候，先按照第一个column name排序，在按照第二个column name排序；

**desc** 或者 **asc** 只对它紧跟着的第一个列名有效，其他不受影响，仍然是默认的升序。



## INSERT INTO 语句

INSERT INTO 语句用于向表中插入新记录。

```
第一种形式无需指定要插入数据的列名，只需提供被插入的值即可：

INSERT INTO table_name
VALUES (value1,value2,value3,...);
```

```
第二种形式需要指定列名及被插入的值：

INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);

```



## UPDATE 语句

UPDATE 语句用于更新表中已存在的记录。

```
UPDATE table_name
SET column1=value1,column2=value2,...
WHERE some_column=some_value;
```



## DELETE 语句

DELETE 语句用于删除表中的行。

```
DELETE FROM table_name
WHERE some_column=some_value;
```



# SQL高级用法

## TOP 子句

SELECT TOP 子句用于规定要返回的记录的数目。

SELECT TOP 子句对于拥有数千条记录的大型表来说，是非常有用的。

### MySQL 语法

```
SELECT column_name(s)
FROM table_name
LIMIT number;
```



## LIKE 操作符

LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。

```
SELECT column_name(s)
FROM table_name
WHERE column_name LIKE pattern;
```

```
'%a'    //以a结尾的数据
'a%'    //以a开头的数据
'%a%'    //含有a的数据
‘_a_’    //三位且中间字母是a的
'_a'    //两位且结尾字母是a的
'a_'    //两位且开头字母是a的
```



## 通配符

在 SQL 中，通配符与 SQL LIKE 操作符一起使用。

SQL 通配符用于搜索表中的数据。

```
通配符		描述
%				替代 0 个或多个字符
_				替代一个字符
[charlist]		 字符列中的任何单一字符

[^charlist]		 不在字符列中的任何单一字符
或
[!charlist]		 
```

## 通配符笔记

```
首先说下LIKE命令都涉及到的通配符：

% 替代一个或多个字符
_ 仅替代一个字符
[charlist] 字符列中的任何单一字符
[^charlist]或者[!charlist] 不在字符列中的任何单一字符

其中搭配以上通配符可以让LIKE命令实现多种技巧：

1、LIKE'Mc%' 将搜索以字母 Mc 开头的所有字符串（如 McBadden）。
2、LIKE'%inger' 将搜索以字母 inger 结尾的所有字符串（如 Ringer、Stringer）。
3、LIKE'%en%' 将搜索在任何位置包含字母 en 的所有字符串（如 Bennet、Green、McBadden）。
4、LIKE'_heryl' 将搜索以字母 heryl 结尾的所有六个字母的名称（如 Cheryl、Sheryl）。
5、LIKE'[CK]ars[eo]n' 将搜索下列字符串：Carsen、Karsen、Carson 和 Karson（如 Carson）。
6、LIKE'[M-Z]inger' 将搜索以字符串 inger 结尾、以从 M 到 Z 的任何单个字母开头的所有名称（如 Ringer）。
7、LIKE'M[^c]%' 将搜索以字母 M 开头，并且第二个字母不是 c 的所有名称（如MacFeather）。
```



## IN 操作符

IN 操作符允许您在 WHERE 子句中规定多个值。

```
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...);
```

IN笔记

**in 与 = 的转换**

```
select * from Websites where name in ('Google','菜鸟教程');
```

可以转换成 **=** 的表达：

```
select * from Websites where name='Google' or name='菜鸟教程';
```



## BETWEEN 操作符

BETWEEN 操作符选取介于两个值之间的数据范围内的值。这些值可以是数值、文本或者日期。

```
SELECT column_name(s)
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
```



## 别名

通过使用 SQL，可以为表名称或列名称指定别名。

基本上，创建别名是为了让列名称的可读性更强。

### 列的 SQL 别名语法

```
SELECT column_name AS alias_name
FROM table_name;
```

### 表的 SQL 别名语法

```
SELECT column_name(s)
FROM table_name *AS *alias_name;
```



## JOIN

SQL JOIN 子句用于把来自两个或多个表的行结合起来，基于这些表之间的共同字段。

最常见的 JOIN 类型：**SQL INNER JOIN（简单的 JOIN）**。 SQL INNER JOIN 从多个表中返回满足 JOIN 条件的所有行。

```
SELECT Websites.id, Websites.name, access_log.count, access_log.date
FROM Websites
INNER JOIN access_log
ON Websites.id=access_log.site_id;
```

### 不同的 SQL JOIN

在我们继续讲解实例之前，我们先列出您可以使用的不同的 SQL JOIN 类型：

- **INNER JOIN**：如果表中有至少一个匹配，则返回行
- **LEFT JOIN**：即使右表中没有匹配，也从左表返回所有的行
- **RIGHT JOIN**：即使左表中没有匹配，也从右表返回所有的行
- **FULL JOIN**：只要其中一个表中存在匹配，则返回行

逻辑：

首先，连接的结果可以在逻辑上看作是由SELECT语句指定的列组成的新表。

左连接与右连接的左右指的是以两张表中的哪一张为基准，它们都是外连接。

外连接就好像是为非基准表添加了一行全为空值的万能行，用来与基准表中找不到匹配的行进行匹配。假设两个没有空值的表进行左连接，左表是基准表，左表的所有行都出现在结果中，右表则可能因为无法与基准表匹配而出现是空值的字段。



## INNER JOIN 关键字

INNER JOIN 关键字在表中存在至少一个匹配时返回行。

```
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.column_name=table2.column_name;

或：

SELECT column_name(s)
FROM table1
JOIN table2
ON table1.column_name=table2.column_name;

注释：INNER JOIN 与 JOIN 是相同的。
```

![图片](http://www.runoob.com/wp-content/uploads/2013/09/img_innerjoin.gif)

## 笔记

在使用 jion 时，**on** 和 **where** 条件的区别如下：

- 1、 **on** 条件是在生成临时表时使用的条件，它不管 **on** 中的条件是否为真，都会返回左边表中的记录。
- 2、**where** 条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有 left join 的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。



## LEFT JOIN 关键字

LEFT JOIN 关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL。

```
SELECT column_name(s)
FROM table1
LEFT JOIN table2
ON table1.column_name=table2.column_name;

或：

SELECT column_name(s)
FROM table1
LEFT OUTER JOIN table2
ON table1.column_name=table2.column_name;

注释：在某些数据库中，LEFT JOIN 称为 LEFT OUTER JOIN
```

![图片](http://www.runoob.com/wp-content/uploads/2013/09/img_leftjoin.gif)



## RIGHT JOIN 关键字

RIGHT JOIN 关键字从右表（table2）返回所有的行，即使左表（table1）中没有匹配。如果左表中没有匹配，则结果为 NULL。

```
SELECT column_name(s)
FROM table1
RIGHT JOIN table2
ON table1.column_name=table2.column_name;

或：

SELECT column_name(s)
FROM table1
RIGHT OUTER JOIN table2
ON table1.column_name=table2.column_name;

注释：在某些数据库中，RIGHT JOIN 称为 RIGHT OUTER JOIN。
```

![图片](http://www.runoob.com/wp-content/uploads/2013/09/img_rightjoin.gif)



## FULL OUTER JOIN 关键字

FULL OUTER JOIN 关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.

FULL OUTER JOIN 关键字结合了 LEFT JOIN 和 RIGHT JOIN 的结果。

```
SELECT column_name(s)
FROM table1
FULL OUTER JOIN table2
ON table1.column_name=table2.column_name;
```

![图片](http://www.runoob.com/wp-content/uploads/2013/09/img_fulljoin.gif)



## UNION 操作符

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。

```
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
注释：默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。
```

```
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
注释：UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。
```



## PRIMARY KEY 约束

PRIMARY KEY 约束唯一标识数据库表中的每条记录。

主键必须包含唯一的值。

主键列不能包含 NULL 值。

每个表都应该有一个主键，并且每个表只能有一个主键。

```
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (P_Id)
)
```

当表已被创建时，如需在 "P_Id" 列创建 PRIMARY KEY 约束，请使用下面的 SQL：

```
ALTER TABLE Persons
ADD PRIMARY KEY (P_Id)
```

如需撤销 PRIMARY KEY 约束，请使用下面的 SQL：

```
ALTER TABLE Persons
DROP PRIMARY KEY
```



## FOREIGN KEY 约束

一个表中的 FOREIGN KEY 指向另一个表中的 UNIQUE KEY(唯一约束的键)。

```
CREATE TABLE Orders
(
O_Id int NOT NULL,
OrderNo int NOT NULL,
P_Id int,
PRIMARY KEY (O_Id),
FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
)
```

当 "Orders" 表已被创建时，如需在 "P_Id" 列创建 FOREIGN KEY 约束，请使用下面的 SQL：

```
ALTER TABLE Orders
ADD FOREIGN KEY (P_Id)
REFERENCES Persons(P_Id)
```

如需命名 FOREIGN KEY 约束，并定义多个列的 FOREIGN KEY 约束，请使用下面的 SQL 语法：

```
ALTER TABLE Orders
ADD CONSTRAINT fk_PerOrders
FOREIGN KEY (P_Id)
REFERENCES Persons(P_Id)
```

如需撤销 FOREIGN KEY 约束，请使用下面的 SQL：

```
ALTER TABLE Orders
DROP FOREIGN KEY fk_PerOrders
```



## CHECK 约束

CHECK 约束用于限制列中的值的范围。

如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会基于行中其他列的值在特定的列中对值进行限制。

下面的 SQL 在 "Persons" 表创建时在 "P_Id" 列上创建 CHECK 约束。CHECK 约束规定 "P_Id" 列必须只包含大于 0 的整数。

```
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (P_Id>0)
)
```

如需命名 CHECK 约束，并定义多个列的 CHECK 约束，请使用下面的 SQL 语法：

```
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
)
```

当表已被创建时，如需在 "P_Id" 列创建 CHECK 约束，请使用下面的 SQL：

```
ALTER TABLE Persons
ADD CHECK (P_Id>0)
```

如需命名 CHECK 约束，并定义多个列的 CHECK 约束，请使用下面的 SQL 语法：

```
ALTER TABLE Persons
ADD CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
```

如需撤销 CHECK 约束，请使用下面的 SQL：

```
ALTER TABLE Persons
DROP CHECK chk_Person
```



## DEFAULT 约束

DEFAULT 约束用于向列中插入默认值。

如果没有规定其他的值，那么会将默认值添加到所有的新记录。

下面的 SQL 在 "Persons" 表创建时在 "City" 列上创建 DEFAULT 约束：

```
CREATE TABLE Persons
(
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255) DEFAULT 'Sandnes'
)
```

如需撤销 DEFAULT 约束，请使用下面的 SQL：

```
ALTER TABLE Persons
ALTER City DROP DEFAULT
```



# CREATE  INDEX 语句

CREATE INDEX 语句用于在表中创建索引。

在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。

**注释：**更新一个包含索引的表需要比更新一个没有索引的表花费更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引。

在表上创建一个简单的索引。允许使用重复的值：

```
CREATE INDEX index_name
ON table_name (column_name)
```

在表上创建一个唯一的索引。不允许使用重复的值：唯一的索引意味着两个行不能拥有相同的索引值。Creates a unique index on a table. Duplicate values are not allowed:

```
CREATE UNIQUE INDEX index_name
ON table_name (column_name)
```

**注释：**用于创建索引的语法在不同的数据库中不一样。因此，检查您的数据库中创建索引的语法。



## DROP

### DROP INDEX 语句

DROP INDEX 语句用于删除表中的索引。

```
ALTER TABLE table_name DROP INDEX index_name
```



### DROP TABLE 语句

DROP TABLE 语句用于删除表。

```
DROP TABLE table_name
```



### DROP DATABASE 语句

DROP DATABASE 语句用于删除数据库。

```
DROP DATABASE database_name
```



### TRUNCATE TABLE 语句

如果我们仅仅需要删除表内的数据，但并不删除表本身，那么我们该如何做呢？

请使用 TRUNCATE TABLE 语句：

```
TRUNCATE TABLE table_name
```



## ALTER TABLE 语句

ALTER TABLE 语句用于在已有的表中添加、删除或修改列。

如需在表中添加列，请使用下面的语法:

```
ALTER TABLE table_name
ADD column_name datatype
```

如需删除表中的列，请使用下面的语法（请注意，某些数据库系统不允许这种在数据库表中删除列的方式）：

```
ALTER TABLE table_name
DROP COLUMN column_name
```



## AUTO INCREMENT 字段

我们通常希望在每次插入新记录时，自动地创建主键字段的值。

我们可以在表中创建一个 auto-increment 字段。

下面的 SQL 语句把 "Persons" 表中的 "ID" 列定义为 auto-increment 主键字段：

```
CREATE TABLE Persons
(
ID int NOT NULL AUTO_INCREMENT,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (ID)
)
```

MySQL 使用 AUTO_INCREMENT 关键字来执行 auto-increment 任务。

默认地，AUTO_INCREMENT 的开始值是 1，每条新记录递增 1。

要让 AUTO_INCREMENT 序列以其他的值起始，请使用下面的 SQL 语法：

ALTER TABLE Persons AUTO_INCREMENT=100

要在 "Persons" 表中插入新记录，我们不必为 "ID" 列规定值（会自动添加一个唯一的值）：

```
INSERT INTO Persons (FirstName,LastName)
VALUES ('Lars','Monsen')
```

上面的 SQL 语句会在 "Persons" 表中插入一条新记录。"ID" 列会被赋予一个唯一的值。"FirstName" 列会被设置为 "Lars"，"LastName" 列会被设置为 "Monsen"。



## 通用数据类型

数据库表中的每个列都要求有名称和数据类型。Each column in a database table is required to have a name and a data type.

SQL 开发人员必须在创建 SQL 表时决定表中的每个列将要存储的数据的类型。数据类型是一个标签，是便于 SQL 了解每个列期望存储什么类型的数据的指南，它也标识了 SQL 如何与存储的数据进行交互。

下面的表格列出了 SQL 中通用的数据类型：

| 数据类型                             | 描述                                       |
| -------------------------------- | ---------------------------------------- |
| CHARACTER(n)                     | 字符/字符串。固定长度 n。                           |
| VARCHAR(n) 或CHARACTER VARYING(n) | 字符/字符串。可变长度。最大长度 n。                      |
| BINARY(n)                        | 二进制串。固定长度 n。                             |
| BOOLEAN                          | 存储 TRUE 或 FALSE 值                        |
| VARBINARY(n) 或BINARY VARYING(n)  | 二进制串。可变长度。最大长度 n。                        |
| INTEGER(p)                       | 整数值（没有小数点）。精度 p。                         |
| SMALLINT                         | 整数值（没有小数点）。精度 5。                         |
| INTEGER                          | 整数值（没有小数点）。精度 10。                        |
| BIGINT                           | 整数值（没有小数点）。精度 19。                        |
| DECIMAL(p,s)                     | 精确数值，精度 p，小数点后位数 s。例如：decimal(5,2) 是一个小数点前有 3 位数小数点后有 2 位数的数字。 |
| NUMERIC(p,s)                     | 精确数值，精度 p，小数点后位数 s。（与 DECIMAL 相同）        |
| FLOAT(p)                         | 近似数值，尾数精度 p。一个采用以 10 为基数的指数计数法的浮点数。该类型的 size 参数由一个指定最小精度的单一数字组成。 |
| REAL                             | 近似数值，尾数精度 7。                             |
| FLOAT                            | 近似数值，尾数精度 16。                            |
| DOUBLE PRECISION                 | 近似数值，尾数精度 16。                            |
| DATE                             | 存储年、月、日的值。                               |
| TIME                             | 存储小时、分、秒的值。                              |
| TIMESTAMP                        | 存储年、月、日、小时、分、秒的值。                        |
| INTERVAL                         | 由一些整数字段组成，代表一段时间，取决于区间的类型。               |
| ARRAY                            | 元素的固定长度的有序集合                             |
| MULTISET                         | 元素的可变长度的无序集合                             |
| XML                              | 存储 XML 数据                                |



## SQL 数据类型快速参考手册

然而，不同的数据库对数据类型定义提供不同的选择。

下面的表格显示了各种不同的数据库平台上一些数据类型的通用名称：

| 数据类型                | Access                 | SQLServer                                | Oracle          | MySQL      | PostgreSQL      |
| ------------------- | ---------------------- | ---------------------------------------- | --------------- | ---------- | --------------- |
| *boolean*           | Yes/No                 | Bit                                      | Byte            | N/A        | Boolean         |
| *integer*           | Number (integer)       | Int                                      | Number          | IntInteger | IntInteger      |
| *float*             | Number (single)        | FloatReal                                | Number          | Float      | Numeric         |
| *currency*          | Currency               | Money                                    | N/A             | N/A        | Money           |
| *string (fixed)*    | N/A                    | Char                                     | Char            | Char       | Char            |
| *string (variable)* | Text (<256)Memo (65k+) | Varchar                                  | VarcharVarchar2 | Varchar    | Varchar         |
| *binary object*     | OLE Object Memo        | Binary (fixed up to 8K)Varbinary (<8K)Image (<2GB) | LongRaw         | BlobText   | BinaryVarbinary |



## 用于各种数据库的数据类型

在 MySQL 中，有三种主要的类型：Text（文本）、Number（数字）和 Date/Time（日期/时间）类型。

**Text 类型：**

| 数据类型             | 描述                                       |
| ---------------- | ---------------------------------------- |
| CHAR(size)       | 保存固定长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的长度。最多 255 个字符。 |
| VARCHAR(size)    | 保存可变长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的最大长度。最多 255 个字符。**注释：**如果值的长度大于 255，则被转换为 TEXT 类型。 |
| TINYTEXT         | 存放最大长度为 255 个字符的字符串。                     |
| TEXT             | 存放最大长度为 65,535 个字符的字符串。                  |
| BLOB             | 用于 BLOBs（Binary Large OBjects）。存放最多 65,535 字节的数据。 |
| MEDIUMTEXT       | 存放最大长度为 16,777,215 个字符的字符串。              |
| MEDIUMBLOB       | 用于 BLOBs（Binary Large OBjects）。存放最多 16,777,215 字节的数据。 |
| LONGTEXT         | 存放最大长度为 4,294,967,295 个字符的字符串。           |
| LONGBLOB         | 用于 BLOBs (Binary Large OBjects)。存放最多 4,294,967,295 字节的数据。 |
| ENUM(x,y,z,etc.) | 允许您输入可能值的列表。可以在 ENUM 列表中列出最大 65535 个值。如果列表中不存在插入的值，则插入空值。**注释：**这些值是按照您输入的顺序排序的。可以按照此格式输入可能的值： ENUM('X','Y','Z') |
| SET              | 与 ENUM 类似，不同的是，SET 最多只能包含 64 个列表项且 SET 可存储一个以上的选择。 |

**Number 类型：**

| 数据类型            | 描述                                       |
| --------------- | ---------------------------------------- |
| TINYINT(size)   | 带符号-128到127 ，无符号0到255。                   |
| SMALLINT(size)  | 带符号范围-32768到32767，无符号0到65535, size 默认为 6。 |
| MEDIUMINT(size) | 带符号范围-8388608到8388607，无符号的范围是0到16777215。 size 默认为9 |
| INT(size)       | 带符号范围-2147483648到2147483647，无符号的范围是0到4294967295。 size 默认为 11 |
| BIGINT(size)    | 带符号的范围是-9223372036854775808到9223372036854775807，无符号的范围是0到18446744073709551615。size 默认为 20 |
| FLOAT(size,d)   | 带有浮动小数点的小数字。在 size 参数中规定显示最大位数。在 d 参数中规定小数点右侧的最大位数。 |
| DOUBLE(size,d)  | 带有浮动小数点的大数字。在 size 参数中规显示定最大位数。在 d 参数中规定小数点右侧的最大位数。 |
| DECIMAL(size,d) | 作为字符串存储的 DOUBLE 类型，允许固定的小数点。在 size 参数中规定显示最大位数。在 d 参数中规定小数点右侧的最大位数。 |

**Date 类型：**

| 数据类型        | 描述                                       |
| ----------- | ---------------------------------------- |
| DATE()      | 日期。格式：YYYY-MM-DD**注释：**支持的范围是从 '1000-01-01' 到 '9999-12-31' |
| DATETIME()  | *日期和时间的组合。格式：YYYY-MM-DD HH:MM:SS**注释：**支持的范围是从 '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59' |
| TIMESTAMP() | *时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的秒数来存储。格式：YYYY-MM-DD HH:MM:SS**注释：**支持的范围是从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC |
| TIME()      | 时间。格式：HH:MM:SS**注释：**支持的范围是从 '-838:59:59' 到 '838:59:59' |
| YEAR()      | 2 位或 4 位格式的年。**注释：**4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069。 |

*即便 DATETIME 和 TIMESTAMP 返回相同的格式，它们的工作方式很不同。在 INSERT 或 UPDATE 查询中，TIMESTAMP 自动把自身设置为当前的日期和时间。TIMESTAMP 也接受不同的格式，比如 YYYYMMDDHHMMSS、YYMMDDHHMMSS、YYYYMMDD 或 YYMMDD。



## SQL Aggregate 函数

SQL Aggregate 函数计算从列中取得的值，返回一个单一的值。

有用的 Aggregate 函数：

- AVG() - 返回平均值
- COUNT() - 返回行数
- FIRST() - 返回第一个记录的值
- LAST() - 返回最后一个记录的值
- MAX() - 返回最大值
- MIN() - 返回最小值
- SUM() - 返回总和




# 视图（Views）



## CREATE VIEW 语句

在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表。

视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。

您可以向视图添加 SQL 函数、WHERE 以及 JOIN 语句，也可以呈现数据，就像这些数据来自于某个单一的表一样。

```
CREATE VIEW view_name AS
SELECT column_name(s)
FROM table_name
WHERE condition
```



## 更新视图

您可以使用下面的语法来更新视图：

```
CREATE OR REPLACE VIEW view_name AS
SELECT column_name(s)
FROM table_name
WHERE condition
```



## 撤销视图

您可以通过 DROP VIEW 命令来删除视图。

```
DROP VIEW view_name
```





# MYSQL教程

这是 [在线教程](http://www.runoob.com/mysql/mysql-tutorial.html)

## 创建数据库

```
CREATE DATABASE 数据库名;
```



# 删除数据库

```
drop database <数据库名>;
```



# 选择数据库

```
use <数据库名>;
```



# 数据类型

## 数值类型

MySQL支持所有标准SQL数值数据类型。

这些类型包括严格数值数据类型(INTEGER、SMALLINT、DECIMAL和NUMERIC)，以及近似数值数据类型(FLOAT、REAL和DOUBLE PRECISION)。

下面的表显示了需要的每个整数类型的存储和范围。

| 类型          | 大小                              | 范围（有符号）                                  | 范围（无符号）                                  | 用途      |
| ----------- | ------------------------------- | ---------------------------------------- | ---------------------------------------- | ------- |
| TINYINT     | 1 字节                            | (-128，127)                               | (0，255)                                  | 小整数值    |
| SMALLINT    | 2 字节                            | (-32 768，32 767)                         | (0，65 535)                               | 大整数值    |
| MEDIUMINT   | 3 字节                            | (-8 388 608，8 388 607)                   | (0，16 777 215)                           | 大整数值    |
| INT或INTEGER | 4 字节                            | (-2 147 483 648，2 147 483 647)           | (0，4 294 967 295)                        | 大整数值    |
| BIGINT      | 8 字节                            | (-9 233 372 036 854 775 808，9 223 372 036 854 775 807) | (0，18 446 744 073 709 551 615)           | 极大整数值   |
| FLOAT       | 4 字节                            | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38) | 单精度浮点数值 |
| DOUBLE      | 8 字节                            | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度浮点数值 |
| DECIMAL     | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                 | 依赖于M和D的值                                 | 小数值     |



## 日期和时间类型

表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR

| 类型        | 大小(字节) | 范围                                       | 格式                  | 用途           |
| --------- | ------ | ---------------------------------------- | ------------------- | ------------ |
| DATE      | 3      | 1000-01-01/9999-12-31                    | YYYY-MM-DD          | 日期值          |
| TIME      | 3      | '-838:59:59'/'838:59:59'                 | HH:MM:SS            | 时间值或持续时间     |
| YEAR      | 1      | 1901/2155                                | YYYY                | 年份值          |
| DATETIME  | 8      | 1000-01-01 00:00:00/9999-12-31 23:59:59  | YYYY-MM-DD HH:MM:SS | 混合日期和时间值     |
| TIMESTAMP | 4      | 1970-01-01 00:00:00/2038结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |



## 字符串类型

字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。该节描述了这些类型如何工作以及如何在查询中使用这些类型。

| 类型         | 大小                | 用途                 |
| ---------- | ----------------- | ------------------ |
| CHAR       | 0-255字节           | 定长字符串              |
| VARCHAR    | 0-65535 字节        | 变长字符串              |
| TINYBLOB   | 0-255字节           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255字节           | 短文本字符串             |
| BLOB       | 0-65 535字节        | 二进制形式的长文本数据        |
| TEXT       | 0-65 535字节        | 长文本数据              |
| MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据     |
| MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据           |
| LONGBLOB   | 0-4 294 967 295字节 | 二进制形式的极大文本数据       |
| LONGTEXT   | 0-4 294 967 295字节 | 极大文本数据             |

# 创建数据表

创建MySQL数据表需要以下信息：

- 表名
- 表字段名
- 定义每个表字段

```
CREATE TABLE table_name (column_name column_type);
```

# 删除数据表

MySQL中删除数据表是非常容易操作的， 但是你再进行删除表操作时要非常小心，因为执行删除命令后所有数据都会消失。

```
DROP TABLE table_name ;
```

# 插入数据

MySQL 表中使用** INSERT INTO **SQL语句来插入数据。

```
INSERT INTO table_name ( field1, field2,...fieldN )
                       VALUES
                       ( value1, value2,...valueN );
```

# 查询数据

MySQL 数据库使用SQL SELECT语句来查询数据。

```
SELECT column_name,column_name
FROM table_name
[WHERE Clause]
[LIMIT N][ OFFSET M]
```

- 查询语句中你可以使用一个或者多个表，表之间使用逗号(,)分割，并使用WHERE语句来设定查询条件。
- SELECT 命令可以读取一条或者多条记录。
- 你可以使用星号（*）来代替其他字段，SELECT语句会返回表的所有字段数据
- 你可以使用 WHERE 语句来包含任何条件。
- 你可以使用 LIMIT 属性来设定返回的记录数。
- 你可以通过OFFSET指定SELECT语句开始查询的数据偏移量。默认情况下偏移量为0。

# WHERE 子句

我们知道从 MySQL 表中使用 SQL SELECT 语句来读取数据。

如需有条件地从表中选取数据，可将 WHERE 子句添加到 SELECT 语句中。

```
SELECT field1, field2,...fieldN FROM table_name1, table_name2...
[WHERE condition1 [AND [OR]] condition2.....
```

- 查询语句中你可以使用一个或者多个表，表之间使用逗号, 分割，并使用WHERE语句来设定查询条件。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以使用 AND 或者 OR 指定一个或多个条件。
- WHERE 子句也可以运用于 SQL 的 DELETE 或者 UPDATE 命令。
- WHERE 子句类似于程序语言中的 if 条件，根据 MySQL 表中的字段值来读取指定的数据。

| 操作符    | 描述                                       | 实例                |
| ------ | ---------------------------------------- | ----------------- |
| =      | 等号，检测两个值是否相等，如果相等返回true                  | (A = B) 返回false。  |
| <>, != | 不等于，检测两个值是否相等，如果不相等返回true                | (A != B) 返回 true。 |
| >      | 大于号，检测左边的值是否大于右边的值, 如果左边的值大于右边的值返回true   | (A > B) 返回false。  |
| <      | 小于号，检测左边的值是否小于右边的值, 如果左边的值小于右边的值返回true   | (A < B) 返回 true。  |
| >=     | 大于等于号，检测左边的值是否大于或等于右边的值, 如果左边的值大于或等于右边的值返回true | (A >= B) 返回false。 |
| <=     | 小于等于号，检测左边的值是否小于于或等于右边的值, 如果左边的值小于或等于右边的值返回true | (A <= B) 返回 true。 |

# UPDATE 查询

如果我们需要修改或更新 MySQL 中的数据，我们可以使用 SQL UPDATE 命令来操作。

```
UPDATE table_name SET field1=new-value1, field2=new-value2
[WHERE Clause]
```

- 你可以同时更新一个或多个字段。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以在一个单独表中同时更新数据。

# DELETE 语句

你可以使用 SQL 的 DELETE FROM 命令来删除 MySQL 数据表中的记录。

```
DELETE FROM table_name [WHERE Clause]
```

- 如果没有指定 WHERE 子句，MySQL 表中的所有记录将被删除。
- 你可以在 WHERE 子句中指定任何条件
- 您可以在单个表中一次性删除记录。

# LIKE 子句

 LIKE 子句中使用百分号 %字符来表示任意字符，类似于UNIX或正则表达式中的星号 *。

如果没有使用百分号 %, LIKE 子句与等号 = 的效果是一样的。

```
SELECT field1, field2,...fieldN 
FROM table_name
WHERE field1 LIKE condition1 [AND [OR]] filed2 = 'somevalue'
```

- 你可以在 WHERE 子句中指定任何条件。
- 你可以在 WHERE 子句中使用LIKE子句。
- 你可以使用LIKE子句代替等号 =。
- LIKE 通常与 % 一同使用，类似于一个元字符的搜索。
- 你可以使用 AND 或者 OR 指定一个或多个条件。
- 你可以在 DELETE 或 UPDATE 命令中使用 WHERE...LIKE 子句来指定条件。

# UNION 操作符

UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

```
SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions]
UNION [ALL | DISTINCT]
SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions];
```

### 参数

- **expression1, expression2, ... expression_n**: 要检索的列。
- **tables: **要检索的数据表。
- **WHERE conditions: **可选， 检索条件。
- **DISTINCT: **可选，删除结果集中重复的数据。默认情况下 UNION 操作符已经删除了重复数据，所以 DISTINCT 修饰符对结果没啥影响。
- **ALL: **可选，返回所有结果集，包含重复数据。

# 排序

如果我们需要对读取的数据进行排序，我们就可以使用 MySQL 的 **ORDER BY** 子句来设定你想按哪个字段哪种方式来进行排序，再返回搜索结果。

```
SELECT field1, field2,...fieldN table_name1, table_name2...
ORDER BY field1, [field2...] [ASC [DESC]]
```

- 你可以使用任何字段来作为排序的条件，从而返回排序后的查询结果。
- 你可以设定多个字段来排序。
- 你可以使用 ASC 或 DESC 关键字来设置查询结果是按升序或降序排列。 默认情况下，它是按升序排列。
- 你可以添加 WHERE...LIKE 子句来设置条件。

# GROUP BY 语句

GROUP BY 语句根据一个或多个列对结果集进行分组。

在分组的列上我们可以使用 COUNT, SUM, AVG,等函数。

```
SELECT column_name, function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```

# 连接的使用

JOIN 按照功能大致分为如下三类：

- **INNER JOIN（内连接,或等值连接）**：获取两个表中字段匹配关系的记录。
- **LEFT JOIN（左连接）：**获取左表所有记录，即使右表没有对应匹配的记录。
- **RIGHT JOIN（右连接）：** 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

# NULL 值处理

MySQL提供了三大运算符:

- **IS NULL:** 当列的值是 NULL,此运算符返回 true。
- **IS NOT NULL:** 当列的值不为 NULL, 运算符返回 true。
- **<=>:** 比较操作符（不同于=运算符），当比较的的两个值为 NULL 时返回 true。

# 正则表达式

下表中的正则模式可应用于 REGEXP 操作符中。

| 模式         | 描述                                       |
| ---------- | ---------------------------------------- |
| ^          | 匹配输入字符串的开始位置。如果设置了 RegExp 对象的 Multiline 属性，^ 也匹配 '\n' 或 '\r' 之后的位置。 |
| $          | 匹配输入字符串的结束位置。如果设置了RegExp 对象的 Multiline 属性，$ 也匹配 '\n' 或 '\r' 之前的位置。 |
| .          | 匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，请使用象 '[.\n]' 的模式。 |
| [...]      | 字符集合。匹配所包含的任意一个字符。例如， '[abc]' 可以匹配 "plain" 中的 'a'。 |
| [^...]     | 负值字符集合。匹配未包含的任意字符。例如， '[^abc]' 可以匹配 "plain" 中的'p'。 |
| p1\|p2\|p3 | 匹配 p1 或 p2 或 p3。例如，'z\|food' 能匹配 "z" 或 "food"。'(z\|f)ood' 则匹配 "zood" 或 "food"。 |
| *          | 匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。 |
| +          | 匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。 |
| {n}        | n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。 |
| {n,m}      | m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次  |

# 事务

- 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
- 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
- 事务用来管理 insert,update,delete 语句

一般来说，事务是必须满足4个条件（ACID）：：原子性（**A**tomicity，或称不可分割性）、一致性（**C**onsistency）、隔离性（**I**solation，又称独立性）、持久性（**D**urability）。

- **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
- **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- **持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

### 事务控制语句：

- BEGIN或START TRANSACTION；显式地开启一个事务；
- COMMIT；也可以使用COMMIT WORK，不过二者是等价的。COMMIT会提交事务，并使已对数据库进行的所有修改成为永久性的；
- ROLLBACK；有可以使用ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；
- SAVEPOINT identifier；SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT；
- RELEASE SAVEPOINT identifier；删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；
- ROLLBACK TO identifier；把事务回滚到标记点；
- SET TRANSACTION；用来设置事务的隔离级别。InnoDB存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ和SERIALIZABLE。

# ALTER命令

当我们需要修改数据表名或者修改数据表字段时，就需要使用到MySQL ALTER命令。

## 删除，添加或修改表字段

如下命令使用了 ALTER 命令及 DROP 子句来删除以上创建表的 i 字段

```
ALTER TABLE testalter_tbl  DROP i;
```

## 修改字段类型及名称

如果需要修改字段类型及名称, 你可以在ALTER命令中使用 MODIFY 或 CHANGE 子句 。

```
ALTER TABLE testalter_tbl MODIFY c CHAR(10);
```



## ALTER TABLE 对 Null 值和默认值的影响

当你修改字段时，你可以指定是否包含值或者是否设置默认值。



## 修改表名

如果需要修改数据表的名称，可以在 ALTER TABLE 语句中使用 RENAME 子句来实现。

```
ALTER TABLE testalter_tbl RENAME TO alter_tbl;
```

# 索引

创建索引时，你需要确保该索引是应用在	SQL 查询语句的条件(一般作为 WHERE 子句的条件)。

实际上，索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。

## 普通索引

### 创建索引

这是最基本的索引，它没有任何限制。它有以下几种创建方式：

```
CREATE INDEX indexName ON mytable(username(length)); 
```

如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。

### 修改表结构(添加索引)

```
ALTER table tableName ADD INDEX indexName(columnName)
```

### 创建表的时候直接指定

```
CREATE TABLE mytable(  
 
ID INT NOT NULL,   
 
username VARCHAR(16) NOT NULL,  
 
INDEX [indexName] (username(length))  
 
);  
```

### 删除索引的语法

```
DROP INDEX [indexName] ON mytable; 
```



## 唯一索引

它与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。它有以下几种创建方式：

### 创建索引

```
CREATE UNIQUE INDEX indexName ON mytable(username(length)) 
```

### 修改表结构

```
ALTER table mytable ADD UNIQUE [indexName] (username(length))
```

### 创建表的时候直接指定

```
CREATE TABLE mytable(  
 
ID INT NOT NULL,   
 
username VARCHAR(16) NOT NULL,  
 
UNIQUE [indexName] (username(length))  
 
);  
```



## 使用ALTER 命令添加和删除索引

有四种方式来添加数据表的索引：

- **ALTER TABLE tbl_name ADD PRIMARY KEY (column_list):** 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
- **ALTER TABLE tbl_name ADD UNIQUE index_name (column_list):** 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
- **ALTER TABLE tbl_name ADD INDEX index_name (column_list):** 添加普通索引，索引值可出现多次。
- **ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list):**该语句指定了索引为 FULLTEXT ，用于全文索引。



## 使用 ALTER 命令添加和删除主键

主键只能作用于一个列上，添加主键索引时，你需要确保该主键默认不为空（NOT NULL）。

```
ALTER TABLE testalter_tbl MODIFY i INT NOT NULL;
ALTER TABLE testalter_tbl ADD PRIMARY KEY (i);
```

你也可以使用 ALTER 命令删除主键：

```
ALTER TABLE testalter_tbl DROP PRIMARY KEY;
```

删除主键时只需指定PRIMARY KEY，但在删除索引时，你必须知道索引名。



## 显示索引信息

你可以使用 SHOW INDEX 命令来列出表中的相关的索引信息。可以通过添加 \G 来格式化输出信息。

```
SHOW INDEX FROM table_name; \G
```

# 临时表

MySQL 临时表在我们需要保存一些临时数据时是非常有用的。临时表只在当前连接可见，当关闭连接时，Mysql会自动删除表并释放所有空间。

```
CREATE TEMPORARY TABLE SalesSummary (
    -> product_name VARCHAR(50) NOT NULL
    -> , total_sales DECIMAL(12,2) NOT NULL DEFAULT 0.00
    -> , avg_unit_price DECIMAL(7,2) NOT NULL DEFAULT 0.00
    -> , total_units_sold INT UNSIGNED NOT NULL DEFAULT 0
);
```



# 序列使用

MySQL 序列是一组整数：1, 2, 3, ...，由于一张数据表只能有一个字段自增主键， 如果你想实现其他字段也实现自动增加，就可以使用MySQL序列来实现。

## 使用 AUTO_INCREMENT

MySQL 中最简单使用序列的方法就是使用 MySQL AUTO_INCREMENT 来定义列。

```
CREATE TABLE insect
    -> (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY (id),
    -> name VARCHAR(30) NOT NULL, # type of insect
    -> date DATE NOT NULL, # date collected
    -> origin VARCHAR(30) NOT NULL # where collected
);
```



## 获取AUTO_INCREMENT值

在MySQL的客户端中你可以使用 SQL中的LAST_INSERT_ID( ) 函数来获取最后的插入表中的自增列的值。



## 设置序列的开始值

一般情况下序列的开始值为1，但如果你需要指定一个开始值100，那我们可以通过以下语句来实现：

```
CREATE TABLE insect
    -> (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY (id),
    -> name VARCHAR(30) NOT NULL, 
    -> date DATE NOT NULL,
    -> origin VARCHAR(30) NOT NULL
)engine=innodb auto_increment=100 charset=utf8;
```



或者你也可以在表创建成功后，通过以下语句来实现：

```
ALTER TABLE t AUTO_INCREMENT = 100;
```

# MySQL 函数

## MySQL 字符串函数

| 函数                                    | 描述                                       | 实例                                       |
| ------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| ASCII(s)                              | 返回字符串 s 的第一个字符的 ASCII 码。                 | 返回 CustomerName 字段第一个字母的 ASCII 码：`SELECT ASCII(CustomerName) AS NumCodeOfFirstCharFROM Customers;` |
| CHAR_LENGTH(s)                        | 返回字符串 s 的字符数                             | 返回字符串 RUNOOB 的字符数`SELECT CHAR_LENGTH("RUNOOB") AS LengthOfString;` |
| CHARACTER_LENGTH(s)                   | 返回字符串 s 的字符数                             | 返回字符串 RUNOOB 的字符数`SELECT CHARACTER_LENGTH("RUNOOB") AS LengthOfString;` |
| CONCAT(s1,s2...sn)                    | 字符串 s1,s2 等多个字符串合并为一个字符串                 | 合并多个字符串`SELECT CONCAT("SQL ", "Runoob ", "Gooogle ", "Facebook") AS ConcatenatedString;` |
| CONCAT_WS(x, s1,s2...sn)              | 同 CONCAT(s1,s2,...) 函数，但是每个字符串直接要加上 x，x 可以是分隔符 | 合并多个字符串，并添加分隔符：`SELECT CONCAT_WS("-", "SQL", "Tutorial", "is", "fun!")AS ConcatenatedString;` |
| FIELD(s,s1,s2...)                     | 返回第一个字符串 s 在字符串列表(s1,s2...)中的位置          | 返回字符串 c 在列表值中的位置：`SELECT FIELD("c", "a", "b", "c", "d", "e");` |
| FIND_IN_SET(s1,s2)                    | 返回在字符串s2中与s1匹配的字符串的位置                    | 返回字符串 c 在指定字符串中的位置：`SELECT FIND_IN_SET("c", "a,b,c,d,e");` |
| FORMAT(x,n)                           | 函数可以将数字 x 进行格式化 "#,###.##", 将 x 保留到小数点后 n 位，最后一位四舍五入。 | 格式化数字 "#,###.##" 形式：`SELECT FORMAT(250500.5634, 2);     -- 输出 250,500.56` |
| INSERT(s1,x,len,s2)                   | 字符串 s2 替换 s1 的 x 位置开始长度为 len 的字符串        | 从字符串第一个位置开始的 6 个字符替换为 runoob：`SELECT INSERT("google.com", 1, 6, "runnob");  -- 输出：runoob.com` |
| LOCATE(s1,s)                          | 从字符串 s 中获取 s1 的开始位置                      | 获取 b 在字符串 abc 中的位置：`SELECT LOCATE('st','myteststring');  -- 5` |
| LCASE(s)                              | 将字符串 s 的所有字母变成小写字母                       | 字符串 RUNOOB 转换为小写：`SELECT LOWER('RUNOOB') -- runoob` |
| LEFT(s,n)                             | 返回字符串 s 的前 n 个字符                         | 返回字符串 runoob 中的前两个字符：`SELECT LEFT('runoob',2) -- ru` |
| LEFT(s,n)                             | 返回字符串 s 的前 n 个字符                         | 返回字符串 abcde 的前两个字符：`SELECT LEFT('abcde',2) -- ab` |
| LOCATE(s1,s)                          | 从字符串 s 中获取 s1 的开始位置                      | 返回字符串 abc 中 b 的位置：`SELECT LOCATE('b', 'abc') -- 2` |
| LOWER(s)                              | 将字符串 s 的所有字母变成小写字母                       | 字符串 RUNOOB 转换为小写：`SELECT LOWER('RUNOOB') -- runoob` |
| LPAD(s1,len,s2)                       | 在字符串 s1 的开始处填充字符串 s2，使字符串长度达到 len        | 将字符串 xx 填充到 abc 字符串的开始处：`SELECT LPAD('abc',5,'xx') -- xxabc` |
| LTRIM(s)                              | 去掉字符串 s 开始处的空格                           | 去掉字符串 RUNOOB开始处的空格：`SELECT LTRIM("    RUNOOB") AS LeftTrimmedString;-- RUNOOB` |
| MID(s,n,len)                          | 从字符串 s 的 start 位置截取长度为 length 的子字符串，同 SUBSTRING(s,n,len) | 从字符串 RUNOOB 中的第 2 个位置截取 3个 字符：`SELECT MID("RUNOOB", 2, 3) AS ExtractString; -- UNO` |
| POSITION(s1 IN s)                     | 从字符串 s 中获取 s1 的开始位置                      | 返回字符串 abc 中 b 的位置：`SELECT POSITION('b' in 'abc') -- 2` |
| REPEAT(s,n)                           | 将字符串 s 重复 n 次                            | 将字符串 runoob 重复三次：`SELECT REPEAT('runoob',3) -- runoobrunoobrunoob` |
| REPLACE(s,s1,s2)                      | 将字符串 s2 替代字符串 s 中的字符串 s1                 | 将字符串 abc 中的字符 a 替换为字符 x：`SELECT REPLACE('abc','a','x') --xbc` |
| REVERSE(s)                            | 将字符串s的顺序反过来                              | 将字符串 abc 的顺序反过来：`SELECT REVERSE('abc') -- cba` |
| RIGHT(s,n)                            | 返回字符串 s 的后 n 个字符                         | 返回字符串 runoob 的后两个字符：`SELECT RIGHT('runoob',2) -- ob` |
| RPAD(s1,len,s2)                       | 在字符串 s1 的结尾处添加字符串 s1，使字符串的长度达到 len       | 将字符串 xx 填充到 abc 字符串的结尾处：`SELECT RPAD('abc',5,'xx') -- abcxx` |
| RTRIM(s)                              | 去掉字符串 s 结尾处的空格                           | 去掉字符串 RUNOOB 的末尾空格：`SELECT RTRIM("RUNOOB     ") AS RightTrimmedString;   -- RUNOOB` |
| SPACE(n)                              | 返回 n 个空格                                 | 返回 10 个空格：`SELECT SPACE(10);`            |
| STRCMP(s1,s2)                         | 比较字符串 s1 和 s2，如果 s1 与 s2 相等返回 0 ，如果 s1>s2 返回 1，如果 s1<s2 返回 -1 | 比较字符串：`SELECT STRCMP("runoob", "runoob");  -- 0` |
| SUBSTR(s, start, length)              | 从字符串 s 的 start 位置截取长度为 length 的子字符串      | 从字符串 RUNOOB 中的第 2 个位置截取 3个 字符：`SELECT SUBSTR("RUNOOB", 2, 3) AS ExtractString; -- UNO` |
| SUBSTRING(s, start, length)           | 从字符串 s 的 start 位置截取长度为 length 的子字符串      | 从字符串 RUNOOB 中的第 2 个位置截取 3个 字符：`SELECT SUBSTRING("RUNOOB", 2, 3) AS ExtractString; -- UNO` |
| SUBSTRING_INDEX(s, delimiter, number) | 返回从字符串 s 的第 number 个出现的分隔符 delimiter 之后的子串。如果 number 是正数，返回第 number 个字符左边的字符串。如果 number 是负数，返回第(number 的绝对值(从右边数))个字符右边的字符串。 | `SELECT SUBSTRING_INDEX('a*b','*',1) -- aSELECT SUBSTRING_INDEX('a*b','*',-1)    -- bSELECT SUBSTRING_INDEX(SUBSTRING_INDEX('a*b*c*d*e','*',3),'*',-1)    -- c` |
| TRIM(s)                               | 去掉字符串 s 开始和结尾处的空格                        | 去掉字符串 RUNOOB 的首尾空格：`SELECT TRIM('    RUNOOB    ') AS TrimmedString;` |
| UCASE(s)                              | 将字符串转换为大写                                | 将字符串 runoob 转换为大写：`SELECT UCASE("runoob"); -- RUNOOB` |
| UPPER(s)                              | 将字符串转换为大写                                | 将字符串 runoob 转换为大写：`SELECT UPPER("runoob"); -- RUNOOB` |

------

## MySQL 数字函数

| 函数名                                      | 描述                                       | 实例                                       |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| ABS(x)                                   | 返回 x 的绝对值                                | 返回 -1 的绝对值：`SELECT ABS(-1) -- 返回1`       |
| ACOS(x)                                  | 求 x 的反余弦值(参数是弧度)                         | `SELECT ACOS(0.25);`                     |
| ASIN(x)                                  | 求反正弦值(参数是弧度)                             | `SELECT ASIN(0.25);`                     |
| ATAN(x)                                  | 求反正切值(参数是弧度)                             | `SELECT ATAN(2.5);`                      |
| ATAN2(n, m)                              | 求反正切值(参数是弧度)                             | `SELECT ATAN2(-0.8, 2);`                 |
| AVG(expression)                          | 返回一个表达式的平均值，expression 是一个字段             | 返回 Products 表中Price 字段的平均值：`SELECT AVG(Price) AS AveragePrice FROM Products;` |
| CEIL(x)                                  | 返回大于或等于 x 的最小整数                          | `SELECT CEIL(1.5) -- 返回2`                |
| CEILING(x)                               | 返回大于或等于 x 的最小整数                          | `SELECT CEIL(1.5) -- 返回2`                |
| COS(x)                                   | 求余弦值(参数是弧度)                              | `SELECT COS(2);`                         |
| COT(x)                                   | 求余切值(参数是弧度)                              | `SELECT COT(6);`                         |
| COUNT(expression)                        | 返回查询的记录总数，expression 参数是一个字段或者 * 号       | 返回 Products 表中 products 字段总共有多少条记录：`SELECT COUNT(ProductID) AS NumberOfProducts FROM Products;` |
| DEGREES(x)                               | 将弧度转换为角度                                 | `SELECT DEGREES(3.1415926535898) -- 180` |
| n DIV m                                  | 整除，n 为被除数，m 为除数                          | 计算 10 除于 5：`SELECT 10 DIV 5;  -- 2`      |
| EXP(x)                                   | 返回 e 的 x 次方                              | 计算 e 的三次方：`SELECT EXP(3) -- 20.085536923188` |
| FLOOR(x)                                 | 返回小于或等于 x 的最大整数                          | 小于或等于 1.5 的整数：`SELECT FLOOR(1.5) -- 返回1` |
| GREATEST(expr1, expr2, expr3, ...)       | 返回列表中的最大值                                | 返回以下数字列表中的最大值：`SELECT GREATEST(3, 12, 34, 8, 25); -- 34`返回以下字符串列表中的最大值：`SELECT GREATEST("Google", "Runoob", "Apple");   -- Runoob` |
| LEAST(expr1, expr2, expr3, ...)          | 返回列表中的最小值                                | 返回以下数字列表中的最小值：`SELECT LEAST(3, 12, 34, 8, 25); -- 3`返回以下字符串列表中的最小值：`SELECT LEAST("Google", "Runoob", "Apple");   -- Apple` |
| [LN](http://www.runoob.com/mysql/func_mysql_ln.asp) | 返回数字的自然对数                                | 返回 2 的自然对数：`SELECT LN(2);  -- 0.6931471805599453` |
| LOG(x)                                   | 返回自然对数(以 e 为底的对数)                        | `SELECT LOG(20.085536923188) -- 3`       |
| LOG10(x)                                 | 返回以 10 为底的对数                             | `SELECT LOG10(100) -- 2`                 |
| LOG2(x)                                  | 返回以 2 为底的对数                              | 返回以 2 为底 6 的对数：`SELECT LOG2(6);  -- 2.584962500721156` |
| MAX(expression)                          | 返回字段 expression 中的最大值                    | 返回数据表 Products 中字段 Price 的最大值：`SELECT MAX(Price) AS LargestPrice FROM Products;` |
| MIN(expression)                          | 返回字段 expression 中的最小值                    | 返回数据表 Products 中字段 Price 的最小值：`SELECT MIN(Price) AS LargestPrice FROM Products;` |
| MOD(x,y)                                 | 返回 x 除以 y 以后的余数                          | 5 除于 2 的余数：`SELECT MOD(5,2) -- 1`        |
| PI()                                     | 返回圆周率(3.141593）                          | `SELECT PI() --3.141593`                 |
| POW(x,y)                                 | 返回 x 的 y 次方                              | 2 的 3 次方：`SELECT POW(2,3) -- 8`          |
| POWER(x,y)                               | 返回 x 的 y 次方                              | 2 的 3 次方：`SELECT POWER(2,3) -- 8`        |
| RADIANS(x)                               | 将角度转换为弧度                                 | 180 度转换为弧度：`SELECT RADIANS(180) -- 3.1415926535898` |
| RAND()                                   | 返回 0 到 1 的随机数                            | `SELECT RAND() --0.93099315644334`       |
| ROUND(x)                                 | 返回离 x 最近的整数                              | `SELECT ROUND(1.23456) --1`              |
| SIGN(x)                                  | 返回 x 的符号，x 是负数、0、正数分别返回 -1、0 和 1         | `SELECT SIGN(-10) -- (-1)`               |
| SIN(x)                                   | 求正弦值(参数是弧度)                              | `SELECT SIN(RADIANS(30)) -- 0.5`         |
| SQRT(x)                                  | 返回x的平方根                                  | 25 的平方根：`SELECT SQRT(25) -- 5`           |
| SUM(expression)                          | 返回指定字段的总和                                | 计算 OrderDetails 表中字段 Quantity 的总和：`SELECT SUM(Quantity) AS TotalItemsOrdered FROM OrderDetails;` |
| TAN(x)                                   | 求正切值(参数是弧度)                              | `SELECT TAN(1.75);  -- -5.52037992250933` |
| TRUNCATE(x,y)                            | 返回数值 x 保留到小数点后 y 位的值（与 ROUND 最大的区别是不会进行四舍五入） | `SELECT TRUNCATE(1.23456,3) -- 1.234`    |

------

## MySQL 日期函数

| 函数名                               | 描述                                       | 实例                                       |
| --------------------------------- | ---------------------------------------- | ---------------------------------------- |
| ADDDATE(d,n)                      | 计算其实日期 d 加上 n 天的日期                       | `SELECT ADDDATE("2017-06-15", INTERVAL 10 DAY);->2017-06-25` |
| ADDTIME(t,n)                      | 时间 t 加上 n 秒的时间                           | `SELECT ADDTIME('2011-11-11 11:11:11', 5)->2011-11-11 11:11:16 (秒)` |
| CURDATE()                         | 返回当前日期                                   | `SELECT CURDATE();-> 2018-09-19`         |
| CURRENT_DATE()                    | 返回当前日期                                   | `SELECT CURRENT_DATE();-> 2018-09-19`    |
| CURRENT_TIME                      | 返回当前时间                                   | `SELECT CURRENT_TIME();-> 19:59:02`      |
| CURRENT_TIMESTAMP()               | 返回当前日期和时间                                | `SELECT CURRENT_TIMESTAMP()-> 2018-09-19 20:57:43` |
| CURTIME()                         | 返回当前时间                                   | `SELECT CURTIME();-> 19:59:02`           |
| DATE()                            | 从日期或日期时间表达式中提取日期值                        | `SELECT DATE("2017-06-15");    -> 2017-06-15` |
| DATEDIFF(d1,d2)                   | 计算日期 d1->d2 之间相隔的天数                      | `SELECT DATEDIFF('2001-01-01','2001-02-02')-> -32` |
| DATE_ADD(d，INTERVAL expr type)    | 计算起始日期 d 加上一个时间段后的日期                     | `SELECT ADDDATE('2011-11-11 11:11:11',1)-> 2011-11-12 11:11:11    (默认是天)SELECT ADDDATE('2011-11-11 11:11:11', INTERVAL 5 MINUTE)-> 2011-11-11 11:16:11 (TYPE的取值与上面那个列出来的函数类似)` |
| DATE_FORMAT(d,f)                  | 按表达式 f的要求显示日期 d                          | `SELECT DATE_FORMAT('2011-11-11 11:11:11','%Y-%m-%d %r')-> 2011-11-11 11:11:11 AM` |
| DATE_SUB(date,INTERVAL expr type) | 函数从日期减去指定的时间间隔。                          | Orders 表中 OrderDate 字段减去 2 天：`SELECT OrderId,DATE_SUB(OrderDate,INTERVAL 2 DAY) AS OrderPayDateFROM Orders` |
| DAY(d)                            | 返回日期值 d 的日期部分                            | `SELECT DAY("2017-06-15");  -> 15`       |
| DAYNAME(d)                        | 返回日期 d 是星期几，如 Monday,Tuesday             | `SELECT DAYNAME('2011-11-11 11:11:11')->Friday` |
| DAYOFMONTH(d)                     | 计算日期 d 是本月的第几天                           | `SELECT DAYOFMONTH('2011-11-11 11:11:11')->11` |
| DAYOFWEEK(d)                      | 日期 d 今天是星期几，1 星期日，2 星期一，以此类推             | `SELECT DAYOFWEEK('2011-11-11 11:11:11')->6` |
| DAYOFYEAR(d)                      | 计算日期 d 是本年的第几天                           | `SELECT DAYOFYEAR('2011-11-11 11:11:11')->315` |
| EXTRACT(type FROM d)              | 从日期 d 中获取指定的值，type 指定返回的值。 type可取值为： MICROSECONDSECONDMINUTEHOURDAYWEEKMONTHQUARTERYEARSECOND_MICROSECONDMINUTE_MICROSECONDMINUTE_SECONDHOUR_MICROSECONDHOUR_SECONDHOUR_MINUTEDAY_MICROSECONDDAY_SECONDDAY_MINUTEDAY_HOURYEAR_MONTH | `SELECT EXTRACT(MINUTE FROM '2011-11-11 11:11:11') -> 11` |
| FROM_DAYS(n)                      | 计算从 0000 年 1 月 1 日开始 n 天后的日期             | `SELECT FROM_DAYS(1111)-> 0003-01-16`    |
| HOUR(t)                           | 返回 t 中的小时值                               | `SELECT HOUR('1:2:3')-> 1`               |
| LAST_DAY(d)                       | 返回给给定日期的那一月份的最后一天                        | `SELECT LAST_DAY("2017-06-20");-> 2017-06-30` |
| LOCALTIME()                       | 返回当前日期和时间                                | `SELECT LOCALTIME()-> 2018-09-19 20:57:43` |
| LOCALTIMESTAMP()                  | 返回当前日期和时间                                | `SELECT LOCALTIMESTAMP()-> 2018-09-19 20:57:43` |
| MAKEDATE(year, day-of-year)       | 基于给定参数年份 year 和所在年中的天数序号 day-of-year 返回一个日期 | `SELECT MAKEDATE(2017, 3);-> 2017-01-03` |
| MAKETIME(hour, minute, second)    | 组合时间，参数分别为小时、分钟、秒                        | `SELECT MAKETIME(11, 35, 4);-> 11:35:04` |
| MICROSECOND(date)                 | 返回日期参数所对应的毫秒数                            | `SELECT MICROSECOND("2017-06-20 09:34:00.000023");-> 23` |
| MINUTE(t)                         | 返回 t 中的分钟值                               | `SELECT MINUTE('1:2:3')-> 2`             |
| MONTHNAME(d)                      | 返回日期当中的月份名称，如 Janyary                    | `SELECT MONTHNAME('2011-11-11 11:11:11')-> November` |
| MONTH(d)                          | 返回日期d中的月份值，1 到 12                        | `SELECT MONTH('2011-11-11 11:11:11')->11` |
| NOW()                             | 返回当前日期和时间                                | `SELECT NOW()-> 2018-09-19 20:57:43`     |
| PERIOD_ADD(period, number)        | 为 年-月 组合日期添加一个时段                         | `SELECT PERIOD_ADD(201703, 5);   -> 201708` |
| PERIOD_DIFF(period1, period2)     | 返回两个时段之间的月份差值                            | `SELECT PERIOD_DIFF(201710, 201703);-> 7` |
| QUARTER(d)                        | 返回日期d是第几季节，返回 1 到 4                      | `SELECT QUARTER('2011-11-11 11:11:11')-> 4` |
| SECOND(t)                         | 返回 t 中的秒钟值                               | `SELECT SECOND('1:2:3')-> 3`             |
| SEC_TO_TIME(s)                    | 将以秒为单位的时间 s 转换为时分秒的格式                    | `SELECT SEC_TO_TIME(4320)-> 01:12:00`    |
| STR_TO_DATE(string, format_mask)  | 将字符串转变为日期                                | `SELECT STR_TO_DATE("August 10 2017", "%M %d %Y");-> 2017-08-10` |
| SUBDATE(d,n)                      | 日期 d 减去 n 天后的日期                          | `SELECT SUBDATE('2011-11-11 11:11:11', 1)->2011-11-10 11:11:11 (默认是天)` |
| SUBTIME(t,n)                      | 时间 t 减去 n 秒的时间                           | `SELECT SUBTIME('2011-11-11 11:11:11', 5)->2011-11-11 11:11:06 (秒)` |
| SYSDATE()                         | 返回当前日期和时间                                | `SELECT SYSDATE()-> 2018-09-19 20:57:43` |
| TIME(expression)                  | 提取传入表达式的时间部分                             | `SELECT TIME("19:30:10");-> 19:30:10`    |
| TIME_FORMAT(t,f)                  | 按表达式 f 的要求显示时间 t                         | `SELECT TIME_FORMAT('11:11:11','%r')11:11:11 AM` |
| TIME_TO_SEC(t)                    | 将时间 t 转换为秒                               | `SELECT TIME_TO_SEC('1:12:00')-> 4320`   |
| TIMEDIFF(time1, time2)            | 计算时间差值                                   | `SELECT TIMEDIFF("13:10:11", "13:10:10");-> 00:00:01` |
| TIMESTAMP(expression, interval)   | 单个参数时，函数返回日期或日期时间表达式；有2个参数时，将参数加和        | `SELECT TIMESTAMP("2017-07-23",  "13:10:11");-> 2017-07-23 13:10:11` |
| TO_DAYS(d)                        | 计算日期 d 距离 0000 年 1 月 1 日的天数              | `SELECT TO_DAYS('0001-01-01 01:01:01')-> 366` |
| WEEK(d)                           | 计算日期 d 是本年的第几个星期，范围是 0 到 53              | `SELECT WEEK('2011-11-11 11:11:11')-> 45` |
| WEEKDAY(d)                        | 日期 d 是星期几，0 表示星期一，1 表示星期二                | `SELECT WEEKDAY("2017-06-15");-> 3`      |
| WEEKOFYEAR(d)                     | 计算日期 d 是本年的第几个星期，范围是 0 到 53              | `SELECT WEEKOFYEAR('2011-11-11 11:11:11')-> 45` |
| YEAR(d)                           | 返回年份                                     | `SELECT YEAR("2017-06-15");-> 2017`      |
| YEARWEEK(date, mode)              | 返回年份及第几周（0到53），mode 中 0 表示周天，1表示周一，以此类推  | `SELECT YEARWEEK("2017-06-15");-> 201724` |

------

## MySQL 高级函数

| 函数名                                      | 描述                                       | 实例                                       |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| BIN(x)                                   | 返回 x 的二进制编码                              | 15 的 2 进制编码:`SELECT BIN(15); -- 1111`    |
| BINARY(s)                                | 将字符串 s 转换为二进制字符串                         | `SELECT BINARY "RUNOOB";-> RUNOOB`       |
| `CASE expression    WHEN condition1 THEN result1    WHEN condition2 THEN result2   ...    WHEN conditionN THEN resultN    ELSE resultEND` | CASE 表示函数开始，END 表示函数结束。如果 condition1 成立，则返回 result1, 如果 condition2 成立，则返回 result2，当全部不成立则返回 result，而当有一个成立之后，后面的就不执行了。 | `SELECT CASE 　　WHEN 1 > 0　　THEN '1 > 0'　　WHEN 2 > 0　　THEN '2 > 0'　　ELSE '3 > 0'　　END->1 > 0` |
| CAST(x AS type)                          | 转换数据类型                                   | 字符串日期转换为日期：`SELECT CAST("2017-08-29" AS DATE);-> 2017-08-29` |
| COALESCE(expr1, expr2, ...., expr_n)     | 返回参数中的第一个非空表达式（从左向右）                     | `SELECT COALESCE(NULL, NULL, NULL, 'runoob.com', NULL, 'google.com');-> runoob.com` |
| CONNECTION_ID()                          | 返回服务器的连接数                                | `SELECT CONNECTION_ID();-> 4292835`      |
| CONV(x,f1,f2)                            | 返回 f1 进制数变成 f2 进制数                       | `SELECT CONV(15, 10, 2);-> 1111`         |
| CONVERT(s USING cs)                      | 函数将字符串 s 的字符集变成 cs                       | `SELECT CHARSET('ABC')->utf-8    SELECT CHARSET(CONVERT('ABC' USING gbk))->gbk` |
| CURRENT_USER()                           | 返回当前用户                                   | `SELECT CURRENT_USER();-> guest@%`       |
| DATABASE()                               | 返回当前数据库名                                 | `SELECT DATABASE();   -> runoob`         |
| IF(expr,v1,v2)                           | 如果表达式 expr 成立，返回结果 v1；否则，返回结果 v2。        | `SELECT IF(1 > 0,'正确','错误')    ->正确`     |
| [IFNULL(v1,v2)](http://www.runoob.com/mysql/mysql-func-ifnull.html) | 如果 v1 的值不为 NULL，则返回 v1，否则返回 v2。          | `SELECT IFNULL(null,'Hello Word')->Hello Word` |
| ISNULL(expression)                       | 判断表达式是否为 NULL                            | `SELECT ISNULL(NULL);->1`                |
| LAST_INSERT_ID()                         | 返回最近生成的 AUTO_INCREMENT 值                 | `SELECT LAST_INSERT_ID();->6`            |
| NULLIF(expr1, expr2)                     | 比较两个字符串，如果字符串 expr1 与 expr2 相等 返回 NULL，否则返回 expr1 | `SELECT NULLIF(25, 25);->`               |
| SESSION_USER()                           | 返回当前用户                                   | `SELECT SESSION_USER();-> guest@%`       |
| SYSTEM_USER()                            | 返回当前用户                                   | `SELECT SYSTEM_USER();-> guest@%`        |
| USER()                                   | 返回当前用户                                   | `SELECT USER();-> guest@%`               |
| VERSION()                                | 返回数据库的版本号                                | `SELECT VERSION()-> 5.6.34`              |











