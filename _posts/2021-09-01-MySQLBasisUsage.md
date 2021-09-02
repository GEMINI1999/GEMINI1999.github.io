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

MySQL支持四种类型的连接：
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
![例子]({{site.url}}/assets/images/MySQLBasisUsage/join_example_table.png)

#### `inner join`

`inner join`的语句需要一个连接谓词的条件

```sql
select t1.user_name, t2.grade from t1 inner join t2 on t1.user_name = t2.user_name;
```

#### `left join`

```sql
select col1, col2 from table_name1 left join table_name2 on table_name1.id = table_name2.id
```

#### `right join`

```sql
select col1, col2 from table_name1 right join table_name2 on table_name1.id = table_name2.id
```

### 组合

## 视图

## 存储过程

## 事务

## 常用`show`命令

|命令|功能|
|-|-|
|||

