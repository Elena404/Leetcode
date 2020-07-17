# SQL - 数据分析

## 基本结构

### 数据库和数据表

### 字段

#### 字段特点

- 一个字段只能是一种数据类型，包括同一类型得到数据
- 一个表中所有字段的行数相同，可以没有值，可以为null
- 一个表中只能有一个主键

#### 数据类型 - 常用

```mysql
int() # 宽度最多11个数字
float() # 默认为(10, 2)
double(m, d) # 默认为(16, 4)
data # YYYY-MM-DD
datatime # YYYY-MM-DD HH:MM:SS
char(m)
varchar(m)
```

#### 约束条件

约束指在表上强制执行的数据检验规则，用来保证创建的表得到数据完整和正确。

- 主键约束 `primary key`
  - 唯一、不重复、非空
  - 分类：单字段主键、多字段主键、联合主键
  - 结合外键定义不同数据表之间的关系。注意必须先把副表先删除再删除主表
- 非空约束 `not null`
- 唯一约束 `unique`：唯一、一列或几列不出现重复值、只能有一个空值
- 自增字段`auto_increment`
  - 一个表只能有一个自增字段（主键）：非空、不重复
  - 数据类型为整数，默认从1开始
  - MySQL特有
- 默认值 `default`：默认值的数据类型与前面设置数据类型相同
- 外键约束`foreign key`：外键依附于主键，字段名可以不同，当副表插入数据时会受到主表的主键的约束，以避免数据错误

```mysql
CREATE TABLE dep(
	d_id char(3) primary key, # 设置主键
    d_name char(10),
    num int
);

CREATE TABLE emp(
	e_id int primary key,
    e_name char(10) not null,
    e_age int,
    e_id char(3)
    # 设置外键
    foreign key (d_id) references dep (d_id)
);
```

## 创建数据库/表：`create` `show` `use` `drop`

```mysql
# 创建数据库
CREATE DATABASE d_name;
--
CREATE TABLE t_name(
	字段名1 字段类型1 约束条件1，
    字段名2 字段类型2 约束条件2，
    字段名3 字段类型3 约束条件3
);

# 查看属性
SHOW CREATE DATABASE d_name;
SHOW DATABASES;
--
SHOW TABLES;

# 定位数据库
USE d_name;

# 删除数据库
DROP DATABASE d_name;
--
DROP TABLE t_name;
```

## 插入数据 `INSERT INTO...VALUES...`

```mysql
# 逐条插入数据
INSERT INTO t_name (f_name1, f_name2)
	VALUES(data1 data1),(data2 data2);

# 导入整个表
# MySQL不支持，直接点鼠标导入数据
LOAD DATA LOCAL INFILE 'path.txt' # 绝对路径，纯英文，斜杠反方向
	INTO TABLE t_name
	FIELDS TERMINATED BY '\t'
	IGNORE 1 LINES;
```

## 检查表数据 `SELECT...FROM...`

```mysql
SELECT * FROM t_name;

SELECT COUNT(*) FROM t_name;

DESC t_name;
```

## 修改数据表结构 - 针对列的修改 `ALTER TABLE`

- `modify`：重新定义字段，修改字段数据类型、排序
- `change`：重新命名，修改字段名、数据类型
- `add`
- `drop`：删除字段或主键
- `rename`

```MYSQL
# 修改表名
ALTER TABLE emp RENAME empdep;

# 修改字段名
ALTER TABLE emp CHANGE depname new_depname varchar(20);

# 修改字段数据类型
ALTER TABLE emp MODIFY depname varchar(30);

# 增加字段
ALTER TABLE emp ADD maname varchar(10) not null;

# 设置主键
ALTER TABLE emp ADD CONSTRAINT PRIMARY KEY (d_id);

# 修改自增字段初始增长
ALTER TABLE emp AUTO_INCREMENT=600;

# 修改字段的排列排序，注意要写上数据类型
ALTER TABLE emp MODIFY maname varchar(10) AFTER depid;

# 删除字段
ALTER TABLE emp DROP maname;
```

## 修改数据表结构 - 针对行的修改：更新`UPDATE...SET`和删除`DELETE`

默认设定下无法执行，为了安全，执行先执行`set sql_safe_updates = 0`

### 更新数据

`update t_name set f_name = value`

```MYSQL
UPDATE fruits
SET f_name = CONCAT('fruit_', f_name); # 合并字符串

UPDATE emp
SET mrg=0 
WHERE empno=250;
```

### 删除记录

`delete from t_name [where clause]`

```mysql
DELETE FROM fruits
WHERE f_id = 'bs';
```

- delete与drop的区别
  - delete可以指定具体删除的数据内容，只删除记录
  - drop删除记录和表结构
- delete与truncate的区别
  - truncate清空数据表，再重建表结构，不能接条件，效率更高，速度更快 `truncate t_name`
  - delete结合where一条一条删除

## 数据查询

### 基本操作

```mysql
SELECT 目标列组
FROM 数据源
WHERE 元组选择条件
GROUP BY 分列组 HAVING 组选组条件
ORDER BY 排序列1 排序要求1

```

- where和having

  - where针对原始数据筛选，是对主键、原始数据的每一行值进行过滤筛选
  - having针对分组后的数据进行条件筛选，对数据透视表进行筛选，汇总值，汇总维度
  - 执行顺序：where→group by→having，所以where不能作用于聚合函数，having再分组和聚合后筛选

- group by：知道主键是什么，才能知道查询分组的单位是什么

- order by：默认升序，`desc`降序，默认最小值为null

- 聚合函数：`avg sum max min count`

- select可以指定列查询，查询结果为一个虚拟表（存在内存里）

- `distinct`操作符：消除重复记录，搭配`select`使用

- 算术操作符

- 比较操作符

- 查询操作符：用再where或having后做条件限定

  ```mysql
  # 在[不在]其中 [not] in
  # [不]存在 [not] exists
  # 在[不在]范围 [not] between and
  # 是[不是]空值 is [not] null
  
  # 模式比较 [not] like
  -- 模糊查询，与通配符"%"（多个字符）或"_"（单个字符）搭配使用
  -- 在excel中即"*"或"?"
  
  # 与运算 and
  # 或运算 or
  # 非运算 not
  
  # 任何一个 any
  # 全部（每个）all
  
  ```

- 空值查询

  判断是否为空值

  `SELECT * FROM emp WHERE mgr IS NULL`

  如果某列记录为null，则返回0，否则返回本身。注意如果想要改变数据可以使用`UPDATE...SET...`

  `SELECT IFNULL(c_name, 0) FROM emp`

  

### 表连接

先找一个表的主键，再找另一个表对应的非主键

1. 方向性：左连接、右连接
2. 主副关系：主表出维度，副表出度量
3. 对应关系
   - 一对一：最不可能出现，主表是表的单位，主键相同即为同一个表
   - 一对多：最符合业务逻辑，一表出维度，多表出度量
   - 多对多：非主键连接，不代表任何一个表的记录单位，不符合业务要求；两个表的值会在重复项上翻倍，维度和度量上都不符合要求

#### 横向连接

- 内连接

  ```mysql
  SELECT c_name
  FROM t1
  INNER JOIN t2
  ON t1.key = t2.key
  ```

  

- 左连接

  ```mysql
  SELECT c_name
  FROM t1
  LEFT JOIN t2
  ON t1.key = t2.key
  ```

  

- 右连接

  ```mysql
  SELECT c_name
  FROM t1
  LEFT JOIN t2
  ON t1.key = t2.key
  ```

  

#### 纵向合并

合并两个或多个select语句的结果集

```mysql
# 消除表中重复行
SELECT * FROM t1
UNION
SELECT * FROM t2
# 保留表中重复行
SELECT * FROM t1
UNION ALL
SELECT * FROM t2
```



### 子查询

先执行（）里的内部查询，再执行外部查询

- `select`表子查询
  - 标量子查询：查询结果为一个单量值
  - 行子查询：返回结果为一行
  - 列子查询：返回结果为一列
  - `exists`子查询：如果为真就查询
  - `from`子查询：类似虚拟表，**别名**后的表可以和其他表连接

```mysql
# 标量子查询
SELECT AVG(sal) FROM emp;
SELECT * FROM emp
WHERE sal > (
	SELECT AVG(sal) FRON emp
);

# 行子查询
SELECT empno, ename, job,deptno
FROM emp
WHERE (deptno, job) = (
	SELECT deptno, job FROM emp WHERE ename='smith'
)
	AND ename != 'smith'
```

- `where`条件子查询

  - 多数用查询操作符解决子查询问题
  - 子查询的返回结果可以替换外部查询任意位置
  - `in`和`=all`的查询结果基本相同
  - `exists`是否存在，如果存在则作为外部查询。不建议使用，理解逻辑即可，判断真伪

  

### as与limit

#### as：重命名

- 临时该表名和字段名，可以为查询结果或子查询命名
- 只在这个查询中有效
- as可以省略，但不建议
- 可以不加引号（注意，Oracle默认输出结果为全大写，别名使用双引号可以规避这个问题。另外使用双引号可以与保留字区别）

#### limit：限制查询结果行数

`LIMIT 6 [OFFSET] 5  `偏移5行，显示6行

默认从0开始

### 常用函数

- 数学函数

- 字符串函数

- 日期或时间函数

- 其他函数

- 逻辑函数

  ```mysql
  # if函数
  SELECT *,
  	IF (comm>0, '是',
          IF(comm=0, '否', '未知')
      ) AS 是否完成绩效
  FROM emp；
  
  # 逻辑表达式
  SELECT *,
  	CASE WHEN comm>0 THEN '是',
  	CASE WHEN comm=0 THEN '否',
  	ELSE '未知' AS 是否完成绩效
  FROM emp;
  ```



