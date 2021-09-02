---
layout: article
title: "MySQL基础用法"
key: 11
tags: 
        - MySQL
        - 基础用法
toc: true
---
## DDL（Data Definition Language，数据库定义语言）操作

### 创建数据库

### 删除数据库

### 创建表

### 删除表

### 修改表

## DML（Data Manipulation Language，数据库操纵语言）操作

### 插入

```sql
insert into table_name(col1, col2, ...) values(val1, val2, ...)
```

### 更新

```sql
update table_name set col = val where ...
```

### 删除

```sql
delete from table_name where ...
```

### 查询

```sql
select col1, col2 from table_name where ...
```

### 排序

### 数据过滤

### 通配符

### 聚合函数

### 联表

MySQL支持三种类型的连接：
- 内连接（`inner join`）
- 左连接（`left join`）
- 右连接（`right join`）

> 注意：在MySQL中`join`,`cross join`以及`inner join`是等效的，可以互相替换

以下是连接的具体用法、含义以及实验结果：

#### 创建实验表并插入数据

创建表

```sql
CREATE TABLE t1 (
    id INT PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL
);

CREATE TABLE t2 (
    id INT PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL,
    grade INT NOT NULL
);
```

插入数据

```sql
insert into t1(id, user_name)
values(1, "小明"),
(2, "小红"),
(3, "小黄");

insert into t2(id, user_name, grade)
values(1, "小红", 90),
(2, "小黄", 100),
(3, "小绿", 95);
```

效果如下：
![连接测试表]({{site.url}}/assets/images/MySQLBasisUsage/join_example_table.png)

#### `inner join`

```sql
select t1.user_name, t2.grade from t1 inner join t2 on t1.user_name = t2.user_name;
```

`sql`执行结果如下:
![innerjoin执行结果]({{site.url}}/assets/images/MySQLBasisUsage/innor_join_execute_result.png)

很明显可以看出，`inner join`只对左右表完全等值的记录才会作连接（可以简单理解为两个表的交集）。


#### `left join`

```sql
select t1.user_name, t2.grade from t1 left join t2 on t1.user_name = t2.user_name;
```

`sql`执行结果如下:
![leftjoin执行结果]({{site.url}}/assets/images/MySQLBasisUsage/left_join_execute_result.png)

可以看到，`left join`将左侧表作为主表，根据连接谓词条件，与右表中的记录进行比较，满足连接谓词条件的记录与左表进行合并，不满足的用NULL来表示。

#### `right join`

```sql
select col1, col2 from table_name1 right join table_name2 on table_name1.id = table_name2.id
```

`sql`执行结果如下:
![leftjoin执行结果]({{site.url}}/assets/images/MySQLBasisUsage/right_join_execute_result.png)

`right join`的结果与`left join`刚好相反，`right join`将右侧表作为主表，根据连接谓词条件，与左表中的记录进行比较，满足连接谓词条件的记录与右表进行合并，不满足的用NULL来表示。

### 组合

## 视图

## 存储过程

## 事务

## 常用`show`命令

|命令|功能|
|-|-|
|||

