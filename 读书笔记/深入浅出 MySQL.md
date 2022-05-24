

### 深入浅出 MySQL

#### 一、SQL基础

##### SQL 简介

SQL 是 Structure Query Language（结构化查询语言）的缩写，它是使用关系模型的数据库应用语言。大多数关系型数据库系统都支持 SQL 语言。

##### ( My ) SQL 使用入门

###### SQL 分类

SQL 语句可以划分为以下 3 个类别。

- DDL ( Data Definition Language ) 语句：数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象。（create、drop、alter 等）
- DML ( Data Manipulation Language ) 语句：数据操纵语言，用于添加、删除、更新和查询数据库记录，并检查数据完整性。（insert、delete、update、select 等）
- DCL ( Data Control Language ) 语句：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。（grant、revoke 等）

###### DDL 语句

DDL 是对数据库内部的对象进行创建、删除、修改等操作的语言。DDL 和 DML 语句的最大区别是 DML 只是对表内部数据操作，而不涉及表的定义、结构的修改，更不会涉及其他对象。

1. 创建数据库：

   ``` mysql
   create database test1;
   ```

   查看存在的数据库：

   ```mysql
   show databases;
   ```

   选择数据库进行操作：

   ```mysql
   use test1;
   ```

   查看数据库中创建的所有数据表：

   ```mysql
   show tables;
   ```

2. 删除数据库：

   ```mysql
   drop database test1;
   ```

3. 创建表：

   ```mysql
   create table emp(ename varchar(10), hiredate date, sal decimal(10, 2), deptno int(2));
   ```

   查看表定义：

   ```mysql
   desc emp;
   ```

   完整查看表定义：

   ```mysql
   show create table emp;
   ```

4. 删除表

   ```mysql
   drop table emp;
   ```

5. 修改表

   ```mysql
   alter table emp modify ename varchar(20);
   ```

   增加表字段：

   ```mysql
   alter table emp add column age int(3);
   ```

   删除表字段：

   ```mysql
   alter table emp drop column age;
   ```

   字段改名：

   ```mysql
   alter table emp change age age1 int(4);
   ```

   修改字段排列顺序：

   ```mysql
   alter table emp add birth date after ename;
   alter table emp modify age int(3) first;
   ```

   更改表名：

   ```mysql
   alter table emp rename emp1;
   ```

###### DML 语句

DML 语句是指对数据库中表记录的操作，包括插入（insert）、更新（update）、删除（delete）、查询（select）。

1. 插入

   ```mysql
   insert into emp (ename, hiredate, sal, deptno) values ('zzx1', '2000-01-01', '2000', 1);
   ```

   可以不指定字段名，但是 values 后面的顺序应该和字段的排列顺序一致：

   ```mysql
   insert into emp values ('lisa', '2003-02-01', '3000', 2);
   ```

   含可空字段、非空但是含有默认值的字段、自增字段，可以不用在 insert 后的字段列表里出现，没写的字段可以自动设置成 NULL、默认值、自增的下一个数字。

   ```mysql
   insert into emp (ename, sal) values ('dony', 1000);
   ```

   可以一次性插入多条记录：

   ```mysql
   insert into dept values (5, 'dept5'), (6, 'dept6');
   ```

2. 更新

   ```mysql
   update emp set sal=4000 where ename='lisa';
   ```

   update 命令可以同时更新多个表中数据：

   ```mysql
   update emp a, dept b set a.sal=a.sal*b.deptno, b.deptname=a.ename where a.deptno=b.deptno;
   ```

3. 删除

   ```mysql
   delete from emp where ename='dony';
   ```

   可以一次删除多个表的数据：

   ```mysql
   delete a, b from emp a, dept b where a.deptno=b.deptno and a.deptno=3;
   ```

4. 查询

   ```mysql
   select * from emp;
   ```

   “*” 表示查询全部字段，也可以选择需要的字段查询：

   ```mysql
   select ename, hiredate, sal, deptno from emp;
   ```

   **查询不重复的记录：**

   | DISTINCT 的作用是消除结果集中的重复行

   ```mysql
   SELECT DISTINCT
   	columns
   FROM
   	table_name
   WHERE
   	where_conditions;
   ```

   **条件查询：**

   ```mysql
   select * from emp where deptno=1;
   ```

   除了 = 外，还可以使用 >、 <、 >=、 <=、 != 等比较运算符；多个条件之间还可以使用 or、and 等逻辑运算符进行多条件联合查询。

   ```mysql
   select * from emp where deptno=1 and sal<3000;
   ```

   **排序和限制：**

   | DESC 和 ASC 是排序顺序关键字，DESC 表示按照字段进行降序排列，ASC 表示升序排列，如果不写此关键字默认是升序排列。ORDER BY 后面可以跟多个不同的排序字段。

   ```mysql
   select * from emp order by sal;
   ```

   如果排序字段的值一样，则值相同的字人段按照第二个排序字段进行排序，以此类推。如果只有一个排序字段，则这些字段相同的记录将会无序排列。

   ```mysql
   select * from emp order by deptno, sal desc;
   ```

   如果希望只现实一部分，可以使用 LIMIT 关键字来实现：

   ```mysql
   select * from emp order by sal limit 3;
   ```

   默认是 limit 0,x，如果希望显示 emp 排序后从第二条记录开始的 3 条记录：

   ```mysql
   select * from emp order by sal limit 1,3;
   ```

   **聚合：**

   | 需要进行一些汇总操作，比如统计整个公司的人数或者统计每个部门的人数，就要用到聚合操作：

   ```mysql
   SELECT [filed1,field2,...,fieldn] fun_name
   FROM tablename
   [WHERE where_condition]
   [GROUP BY field1,field2,...,fieldn
   [WITH ROLLUP]]
   [HAVING where_condition]
   ```

   - fun_name 表示聚合函数，常用的有 sum、count、max、min。
   - GROUP BY 关键字表示要进行分类聚合的字段，比如要按照部门分类统计员工数量，部门就应该写在 GROUP BY 后面。
   - WITH ROLLUP 是可选语法，表明是否对分类聚合后的结果进行再汇总。
   - HAVING 关键字表示对分类后的结果再进行条件的过滤。

   注意：having 是对聚合后的结果进行条件的过滤，where 是在聚合前就对记录进行过滤，尽可能用 where 先过滤记录。

   统计公司总人数：

   ```mysql
   select count(1) from emp;
   ```

   统计各个部门人数：

   ```mysql
   select deptno, count(1) from emp group by deptno;
   ```

   既要统计各部门人数，又要统计总人数：

   ```mysql
   select deptno, count(1) from emp group by deptno with rollup;
   ```

   统计人数大于 1 人的部门：

   ```mysql
   select deptno, count(1) from emp group by deptno having count(1)>1;
   ```

   统计公司所有员工的新手总额、最高和最低薪水：

   ```mysql
   select sum(sal), max(sal), min(sal) from emp;
   ```

   **表连接：**

   | 当需要同时显示多个表中的字段时，就可以用表连接来实现这样的功能。表连接分为内连接和外连接，内连接仅选出两张表中互相匹配的记录，外连接会选出其他不匹配的记录。

   内连接：查询出所有雇员名字和所在部门名称，需要两张表：

   ```mysql
   select ename, deptname from emp, dept where emp.deptno=dept.deptno;
   ```

   外连接。又分为左连接和右连接。

   - 左连接：包含所有的左边表中的记录甚至是右边表中没有和它匹配的记录。
   - 右连接：包含所有的右边表中的记录甚至是左边表中没有和它匹配的记录。

   查询 emp 中所有用户名和所在部门名称：

   左连接。可以看到结果中包含所有左边表中的记录，即使 ‘dony’并没有和之符合的部门。

   ```mysql
   select ename, deptname from emp left join dept on emp.deptno=dept.deptno;
   +--------+----------+
   | ename  | deptname |
   +--------+----------+
   | zzx1   | tech     |
   | bjguan | tech     |
   | lisa   | sale     |
   | bzshen | hr       |
   | dony   | NULL     |
   +--------+----------+
   ```

   右连接。结果中包含所有右边表中的记录。

   ```mysql
   select ename, deptname from emp right join dept on emp.deptno=dept.deptno;
   +--------+----------+
   | ename  | deptname |
   +--------+----------+
   | zzx1   | tech     |
   | lisa   | sale     |
   | bjguan | tech     |
   | bzshen | hr       |
   +--------+----------+
   ```

    **子查询：**

   | 进行查询所需要的条件是另一个 select 语句的结果，这个时候就要用到子查询。子查询的关键字包括 in、not in、=、!=、exists、not exists 等。

   ```mysql
   select * from emp where deptno in (select deptno from dept);
   ```

   某些情况下，子查询可以转化为表连接

   ```mysql
   select emp.* from emp, dept where emp.deptno=dept.deptno;
   ```

   记录联合：

   | 将两个表的数据按照一定条件查询出来后，将结果合并到一起显示出来。

   ```mysql
   select deptno from emp union all select deptno from dept;
   +--------+
   | deptno |
   +--------+
   |      1 |
   |      2 |
   |      1 |
   |      3 |
   |      4 |
   |      1 |
   |      2 |
   |      3 |
   +--------+
   ```

   将结果去重可以使用 union：

   ```mysql
   select deptno from emp union select deptno from dept;
   +--------+
   | deptno |
   +--------+
   |      1 |
   |      2 |
   |      3 |
   |      4 |
   +--------+
   ```

###### DCL 语句

| DCL 语句是 DBA 用来管理系统中的对象权限时使用的。

创建数据库用户，并添加对 sakila 数据库中的所有表的 SELECT/INSERT 权限：

```mysql
grant select, insert on sakila.* to 'zl'@'localhost' identified by '123';
```

这个时候登录 zl，可以对 sakila 中的所有表进行 INSERT/SELECT 操作。

收回 zl 用户的 INSERT 权限：

```mysql
revoke insert on sakila.* from 'zl'@'localhost';
```

#### 二、MySQL 支持的数据类型

| MySQL提供了多种数据类型，主要包括数值型、字符串类型、日期和时间类型。

##### 数值类型

标准 SQL 的数值类型包括 

- 严格数值类型：INTEGER、SMALLINT、DECIMAL、NUMERIC
- 近似数值类型：FLOAT、REAL、DOUBLE、PRECISION

MySQL 还扩展增加了 TINYINT、DEMIUMINT 和 BIGINT 三种长度不同的整型，并增加了 BIT 类型，用于存放位数据。

MySQL 中的数值类型：

| 整数类型     | 字节 | 最小值                                | 最大值                                                  |
| ------------ | ---- | ------------------------------------- | ------------------------------------------------------- |
| TINYINT      | 1    | 有符号 -128，无符号 0                 | 有符号 127，无符号 255                                  |
| SMALLINT     | 2    | 有符号 -32768，无符号 0               | 有符号 32767，无符号 65535                              |
| MEDIUMINT    | 3    | 有符号 -8388608，无符号 0             | 有符号 8388607，无符号 1677215                          |
| INT、INTEGER | 4    | 有符号 -2147483648，无符号 0          | 有符号 2147483647，无符号 4294967295                    |
| BIGINT       | 8    | 有符号 -9223372036854775808，无符号 0 | 有符号 9223372036854775807，无符号 18446744073709551615 |

整数类型如果超出类型范围，会发生“Out of range” 错误提示，在选择数据类型时，要根据应用的实际情况确定取值范围，最后根据确定的结果慎重选择数据类型。

**ZEROFILL**

MySQL 支持在整型数据类型后面的小括号内指定显示宽度，注意不是取值范围。

```mysql
create table t1 (id1 int, id2 int(5));
```

这里的 id1 和 id2 取值范围是相同的，而 id2 多了一个显示范围 5，id1 默认显示范围 11。显示范围可以配合 zerofill 使用，也就是在数字位数不够的空间用字符 “0” 填满。

```mysql
alter table t1 modify id1 int zerofill;
alter table t1 modify id2 int(5) zerofill;
select * from t1;
+-------------+--------+
|     id1     |   id2  |
+-------------+--------+
| 00000000001 |  00001 |
+-------------+--------+
```

**UNSIGNED**

简单理解就是不保存负值，所以让出了更多空间保存正数，上限值更大。例如 tinyint 有符号范围是 -128~127，UNSIGNED范围是 0~255。如果一个列指定为 zerofill，则 MySQL 自动为该列添加 UNSIGNED 属性。

**AUTO_INCREMENT**

只用于整数类型，一般从 1 开始，每行增加1，插入 NULL 时，MySQL 自动插入一个比该列中最大值大 1 的值。一个表中最多只能有一个 AUTO_INCREMENT 的列，必须定义为 NOT NULL，并定义为 PRIMARY KEY 或 UNIQUE 键。

可以使用以下任何一种方式定义 AUTO_INCREMENT：

```mysql
create table ai (id int auto_increment not null primary key);
create table ai (id int auto_increment not null, primary key(id));
create table ai (id int auto_increment not null, unique(id));
```

| 浮点数类型 | 字节 | 最小值                   | 最大值                   |
| ---------- | ---- | ------------------------ | ------------------------ |
| FLOAT      | 4    | ±1.175494351E-38         | ±3.402328466E+38         |
| DOUBLE     | 8    | ±2.2250738585072014E-308 | ±1.7976931348623157E+308 |

| 定点数类型 | 字节 | 描述                                                         |
| ---------- | ---- | ------------------------------------------------------------ |
| DEC(M,D)   | M+2  | 最大取值范围与 DOUBLE 相同，给定 DECIMAL 的有效取值范围由 M  和 D 决定 |

定点数在 MySQL 内部以字符串形式存放，比浮点数更精确，适合用来表示货币等精度高的数据。

浮点数和定点数都可以用类型名称后面加 “(M,D)” 的方式来进行表示，M 表示该值一共显示 M 位数字（整数位+小数位），其中包含 D 位小数，M 称为 **精度**，D 称为**标度**。如，定义一个 float(7,4) 可以显示 -999.9999。保存值时进行四舍五入，如果在 float(7,4) 中插入 999.00009，近似结果是 999.0001。

浮点数如果不写精度和标度，则会按照时机精度值显示，如果有精度和标度，则会自动将四舍五入后的结果插入，系统不会报错；

定点数如果不写精度和标度，则按照默认值 decimal(10,0) 来进行操作，并且如果数据超越了精度和标度值，系统则会报错。

| 位类型 | 字节  | 最小值 | 最大值  |
| ------ | ----- | ------ | ------- |
| BIT(M) | 1 ~ 8 | BIT(1) | BIT(64) |

BIT （位）类型，用于存放字段值，BIT(M) 可以用来存放多位二进制数，M 范围从 1 ~ 64，如果不写，默认为 1 位。

可以用 bin() （二进制）或者 hex() （十六进制）函数进行读取，直接 select 不显示结果。

```mysql
select bin(id), hex(id) from t2;
+---------+---------+
| bin(id) | hex(id) |
+---------+---------+
| 1       | 1       |
+---------+---------+
```

##### 日期时间类型

多种表示日期和时间的类型区别：

- 表示年月日，用 DATE
- 表示年月日时分秒，用 DATETIME
- 表示时分秒，用 TIME
- 需要经常插入或者更新日期位当前系统时间，用 TIMESTAMP
- 只是表示年份，用 YEAR

DATETIME 是 DATE 和 TIME 的组合。

```mysql
create table t (d date, t time, dt datetime);
insert into t values (now(), now(), now());
select * from t;
+------------+----------+---------------------+
| d          | t        | dt                  |
+------------+----------+---------------------+
| 2022-05-24 | 16:52:13 | 2022-05-24 16:52:13 |
+------------+----------+---------------------+
```

**TIMESTAMP 和 DATETIME 的区别：**

- TIMESTAMP 支持的时间范围较小
- TIMESTAMP 会默认为 CURRENT_TIMESTAMP（系统日期），并且一个表中只能有一个 TIMESTAMP 默认值为 CURRENT_TIMESTAMP。插入 NULL 值时会自动设置为当前的日期和时间。不明确给 TIMESTAMP 赋值时也会自动设置为当前的日期和时间。
- TIMESTAMP 类型还与时区相关，插入和从数据库取出时，都会将日期转换为本地时区后显示。DATETIME 只能反映出插入当地的时区。

采用不同格式将日期 “2007-9-3 12:10:10” 插入到 DATETIME 列中：

```mysql
create table t6 (dt datetime);
insert into t6 values ('2007-9-3 12:10:10');
insert into t6 values ('2007/9/3 12+10+10');
insert into t6 values ('20070903121010');
insert into t6 values (20070903121010)
select * from t6;
+---------------------+
| dt                  |
+---------------------+
| 2007-09-03 12:10:10 |
| 2007-09-03 12:10:10 |
| 2007-09-03 12:10:10 |
| 2007-09-03 12:10:10 |
+---------------------+
```

##### 字符串类型

MySQL 包括了 CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM 和 SET 等多种字符串类型。

**CHAR 和 VARCHAR 类型**

区别主要在于存储方式不同：CHAR 列的长度固定为创建表时声明的长度，可以为 0~255 的任何值；而 VARCHAR 的值为可变长字符串。检索的时候，CHAR 列删除了尾部的空格，而 VARCHAR 保留这些空格。

**BINARY 和 VARBINARY 类型**

类似于 CHAR 和 VARCHAR ，不同的是它们包含二进制字符串。

**ENUM 类型**

ENUM 的值范围需要在创建表时通过枚举方式显式指定。

 ```mysql
create table t (gender enum('M', 'F'));
insert into t values ('M'), ('1'), ('f'), (NULL);
select * from t;
+--------+
| gender |
+--------+
| M      |
| M      |
| F      |
| NULL   |
+--------+
 ```

ENUM 类型是忽略大小写的，对于插入不在 ENUM 指定范围的值时，会插入 enum('M', 'F') 的第一个值 'M'。

**SET 类型**

SET 和 ENUM 非常类型，区别在于 SET 一次可以选取多个成员，而 ENUM 则只能选一个。

```mysql
create table t (col set('a', 'b', 'c','d'));
insert into t values ('a,b'), ('a,d,a'), ('a,b'),('a,c'),('a');
select * from t;
+------+
| col  |
+------+
| a,b  |
| a,d  |
| a,b  |
| a,c  |
| a    |
+------+
```

SET 可以从允许值集合中任意选择 1 个或多个元素进行组合，对于超出允许范围的值例如 ('a,d,f') 将不允许注入到上面的例子中。对于 ('a,d,a') 这样包含重复成员的集合将只取一次。