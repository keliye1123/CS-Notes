# SQL语法
- [SQL语法](#SQL语法)
  - [一、SQL分类](#一SQL分类)
  - [二、数据类型](#二数据类型)
  - [三、DDL](#三DDL)
    - [数据库操作](#数据库操作)
    - [表操作](#表操作)
  - [四、DML](#四DML)
    - [添加数据](#添加数据)
    - [修改数据](#修改数据)
    - [删除数据](#删除数据)
  - [五、DQL](#五DQL)
    - [基本查询](#基本查询)
    - [条件查询](#条件查询)
    - [分组查询](#分组查询)
    - [排序查询](#排序查询)
    - [分页查询](#分页查询)
  - [六、函数](#六函数)
  
## 一、SQL分类
SQL语法一共分类四大类
- DDL:数据定义语言，用来定义数据库对象（数据库，表，字段）
- DML:数据操作语言，用来对数据库表中的数据进行增删改
- DQL：数据查询语言，用于查询数据库中表的记录
- DCL：数据控制语言，用来创建数据库用户、控制数据库的访问权限

## 二、数据类型
|   数据类型    | 大小    | 数据类型       | 大小          |
|:---------:|-------|------------|-------------|
|  TINYINT  | 1byte | CHAR       | 0~255byte   |
| SMALLINT  | 2byte | VARCHAR    | 0~65535byte |
| MEDIUMINT | 3byte | TINYBLOB_  | 二进制数据       |
|    INT    | 4byte | TINYTEXT   | 短文本字符串      |
|  BIGINT   | 8byte | BLOB       | 二进制长文本字符串   |
|   FLOAT   | 4byte | TEXT       | 长文本数据       |
|  DOUBLE   | 8byte | MEDIUMBLOB | 二进制中等长度文本数据 |

- 注意：使用浮点数时后面要跟着两个参数第一个是整个数字的长度，第二个是允许出现几个小数。
- 例子：double(4,1)表示数字长度为4保留一位小数
- 注意：使用char和varchar时后面要跟一个参数为最大字符数量。


|    类型     |         格式          |                   取值范围                    |     说明     |
|:---------:|:-------------------:|:-----------------------------------------:|:----------:|
|   DATE    |     YYYY‑MM‑DD      |          1000‑01‑01 ~ 9999‑12‑31          |    只存日期    |
|   TIME    |      HH:MM:SS       |          -838:59:59 ~ 838:59:59           | 时间，可支持负数时长 |
| DATETIME  | YYYY‑MM‑DD HH:MM:SS | 1000‑01‑01 00:00:00 ~ 9999‑12‑31 23:59:59 |  日期 + 时间   |
| TIMESTAMP | YYYY‑MM‑DD HH:MM:SS |          1970‑01‑01 ~ 2038‑01‑19          | 时间戳，自动时区转换 |
|   YEAR    |        YYYY         |                 1901‑2155                 |    仅年份     |


## 三、DDL

### 数据库操作

#### 1.数据库查询
~~~sql
##查询所有数据库
SHOW DATABASES;
##查询当前数据库
SELECT DATABASE();
~~~

#### 2.创建数据库
~~~sql
CREATE DATABASE [if not exists] 数据库名 [default charset 字符集] [collate 排序规则];
~~~

#### 3.删除数据库
~~~sql
DROP DaATBASE [if exists] 数据库名;
~~~

#### 4.使用数据库
~~~sql
USE 数据库名;
~~~

### 表操作

#### 1.表查询
~~~sql
##查询当前数据库所有表
SHOW TABLES;
##查询表结构
DESC 表名;
##查询指定表的建表语句
SHOW CREATE TABLE 表名;
~~~

#### 2.表创建
~~~sql
CREATE TABLE 表名(
    字段1 字段1类型[COMMENT 字段1注释],
    字段2 字段2类型[COMMENT 字段2注释],
    字段3 字段3类型[COMMENT 字段3注释],
    ...
    字段n 字段n类型[COMMENT 字段3注释]
)[COMMENT 表注释];
~~~

#### 3.表修改
~~~sql
##添加字段
ALTER TABLE 表名 ADD 字段名 类型(长度) [comment 注释][约束];

~~~

#### 4.修改字段
~~~sql
##修改数据类型
ALTER TABLE 表名 MODIFY 字段名 新数据类型(长度);
##修改字段名和字段类型
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型(长度)[COMMENT 注释][约束]; 
##删除字段
ALTER TABLE 表名 DROP 字段名;
##修改表名
ALTER TABLE 表名 RENAME TO 新表名;
##删除表
DROP TABLE [if exits]表名;
##删除指定表并重新创建该表
TRUNCATE TABLE 表名;
~~~

## 四、DML

### 添加数据
~~~
##给指定字段添加数据
INSERTER INTO 表名（字段名1，字段名2，...）VALUES(值1，值2，...);
##给全部字段添加数据
INSERTER INTO 表名 VALUES (值1，值2，...);
##批量添加数据
INSERTER INTO (字段名1，字段名2,…) VALUES(值1,值2, ……),(值1,值2,…),(值1,值2, .…);
INSERT INTO 表名 VALUES (值1, 值2, ……), (值1, 值2, …), (值1, 值2, …);
~~~
注意：
- 插入数据时，指定的字段顺序需要与值的顺序是一一对应的。
- 字符串和日期型数据应该包含在引号中。
- 插入的数据大小，应该在字段的规定范围内。

### 修改数据
~~~sql
UPDATE 表名 SET 字段名1=值1,字段名2=值2,...[WHERE 条件];
~~~

### 删除数据
~~~sql
DELETE FROM 表名 [WHERE 条件];
~~~

## 五、DQL

### 基本查询

#### 1.查询多个字段
~~~sql
SELECT 字段1,子段2,子段3... FROM 表名;
SELECT FROM 表名;
~~~

#### 2.设置别名
~~~sql
SELECT 字段1[AS 别名1],字段2[AS 别名2]...FROM 表名;
~~~

#### 3、去除重复记录
~~~sql
SELECT DISTINCT 字段列表 FROM 表名;
~~~

### 条件查询
~~~sql
SELECT 字段列表 FROM 表名 WHERE 条件列表;
~~~
![img_1.png](Image/databases/SQL语法/SQL语法_image1.png)

### 分组查询
~~~sql
SELECT 字段列表 FROM 表名 [WHERE 条件] GROUP BY 分组字段名 [HAVING 分组后过滤条件];
~~~
- where与having的区别：执行时机不同：where是分组之前进行过滤，不满足where条件，不参与分组；而having是分组之后对结果进行过滤，判断条件不同:where不能对聚合函数进行判断，而having可以。
    - 注意
      • 执行顺序: where > 聚合函数> having。
      • 分组之后，查询的字段一般为聚合函数和分组字段，查询其他字段无任何意义。


### 排序查询
~~~sql
SELECT 字段列表 FROM 表名 ORDERBY 字段1排序方式1,字段2 排序方式2;
~~~
- 排序方法：ASC:升序(默认值),DESC:降序;
- 注意：如果多个字段排序，当第一个字段相同时，才根据第二个字段排序。

### 分页查询
~~~
SELECT 字段列表 FROM 表名 LIMIT 起始索引，查询记录数;
~~~

## 六、函数
聚合函数：将一列数据作为一个个体，进行纵向计算。
- 常见聚合函数：
  - count:统计数量
  - max:最大值
  - min:最小值
  - avg:平均值
  - sum:求和
~~~sql
SELECT 聚合函数(字段列表)FROM表名 WHERE 条件;
~~~
- 注意：null是不参与聚合函数的运算。