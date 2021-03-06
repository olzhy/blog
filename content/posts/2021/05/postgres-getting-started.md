---
title: PostgreSQL初探
author: olzhy
type: post
date: 2021-05-21T15:14:46+08:00
url: /posts/postgres-getting-started.html
categories:
  - 计算机
tags:
  - PostgreSQL
keywords:
  - PostgreSQL
  - 初探
description: PostgreSQL初探 (PostgreSQL Getting Started)
---
[上一篇](/posts/install-postgres-on-centos-from-source.html)已经安装好了PostgreSQL环境，本篇会在其上使用SQL做一些简单的操作。

### 1 基础SQL操作

**a) 建表**

建两张表：一张是天气表（`weather`），记录各个城市每天的温度与降水量；一张是城市表（`cities`），记录城市的坐标。PostgreSQL推荐关键字采用大写格式，字段名及类型采用小写格式。

如下为建表语句：

```sql
CREATE TABLE weather (
    city        varchar(80), -- city name (城市名)
    temp_low    int,  -- low temperature (最低温度)
    temp_high   int,  -- high temperature (最高温度)
    prcp        real, -- precipitation (降水量)
    date        date  -- date (日期)
);

CREATE TABLE cities (
    name        varchar(80), -- city name (城市名)
    location    point -- point为PostgreSQL特有类型，该字段表示地理坐标(经度, 纬度)
);
```

**b) 插值**

采用如下语句分别为`weather`表及`cities`表插入数据。

```sql
INSERT INTO weather (city, temp_low, temp_high, prcp, date)
    VALUES ('Beijing', 18, 32, 0.25, '2021-05-19'), 
           ('Beijing', 20, 30, 0.0, '2021-05-20'),
           ('Dalian', 16, 24, 0.0, '2021-05-21');

INSERT INTO cities (name, location)
    VALUES ('Beijing', '(116.3, 39.9)'),
           ('Shanghai', '(121.3, 31.1)');
```

**c) 简单查询**

在被选列上使用表达式`(temp_low + temp_high) / 2`，返回城市每天的平均温度。

```sql
SELECT city, (temp_low + temp_high) / 2 AS temp_avg, date
FROM weather;
```

```text
  city   | temp_avg |    date    
---------+----------+------------
 Beijing |       25 | 2021-05-19
 Beijing |       25 | 2021-05-20
 Dalian  |       20 | 2021-05-21
(3 rows)
```

使用`WHERE`条件，筛选城市为Beijing且降水量大于0的记录。

```sql
SELECT *
FROM weather
WHERE city = 'Beijing'
  AND prcp > 0.0;
```

```text
  city   | temp_low | temp_high | prcp |    date    
---------+----------+-----------+------+------------
 Beijing |       18 |        32 | 0.25 | 2021-05-19
(1 row)
```

在被选列上使用`DISTINCT`关键字，筛选出去重后的城市名，并使用`ORDER BY`关键字按城市名字段正序返回结果。

```sql
SELECT DISTINCT city 
FROM weather 
  ORDER BY city;
```

```text
  city   
---------
 Beijing
 Dalian
(2 rows)
```

**d) 连表查询**

内连接：将`weather`表及`cities`表进行内连接（取两表中城市名相同的所有行），返回城市在对应日期的的温度，降水量及地理位置。

```sql
SELECT w.city, w.temp_low, w.temp_high, w.prcp, c.location, w.date
FROM weather w, cities c 
WHERE w.city = c.name;

-- 两种写法等价
SELECT w.city, w.temp_low, w.temp_high, w.prcp, c.location, w.date
FROM weather w INNER JOIN cities c 
  ON (w.city = c.name);
```

```text
  city   | temp_low | temp_high | prcp |   location   |    date    
---------+----------+-----------+------+--------------+------------
 Beijing |       18 |        32 | 0.25 | (116.3,39.9) | 2021-05-19
 Beijing |       20 |        30 |    0 | (116.3,39.9) | 2021-05-20
(2 rows)
```

左外连接：将`weather`表及`cities`表进行左外连接（返回左表所有行，若左表的某行在右表没有匹配行，则补空值），返回城市在对应日期的的温度，降水量及地理位置。

```sql
SELECT w.city, w.temp_low, w.temp_high, w.prcp, c.location, w.date
FROM weather w LEFT OUTER JOIN cities c
  ON (w.city = c.name);
```

```text
  city   | temp_low | temp_high | prcp |   location   |    date    
---------+----------+-----------+------+--------------+------------
 Beijing |       18 |        32 | 0.25 | (116.3,39.9) | 2021-05-19
 Beijing |       20 |        30 |    0 | (116.3,39.9) | 2021-05-20
 Dalian  |       16 |        24 |    0 |              | 2021-05-21
(3 rows)
```

右外连接：将`weather`表及`cities`表进行右外连接（返回右表所有行，若右表的某行在左表没有匹配行，则补空值），返回城市在对应日期的的温度，降水量及地理位置。

```sql
SELECT c.name, w.temp_low, w.temp_high, w.prcp, c.location, w.date
FROM weather w RIGHT OUTER JOIN cities c
  ON (w.city = c.name);
```

```text
   name   | temp_low | temp_high | prcp |   location   |    date    
----------+----------+-----------+------+--------------+------------
 Beijing  |       20 |        30 |    0 | (116.3,39.9) | 2021-05-20
 Beijing  |       18 |        32 | 0.25 | (116.3,39.9) | 2021-05-19
 Shanghai |          |           |      | (121.3,31.1) | 
(3 rows)
```

全外连接：将`weather`表及`cities`表进行全外连接（返回两表的所有行，当一表的某行在另一表没有匹配行，则补空值），返回城市在对应日期的的温度，降水量及地理位置。

```sql
SELECT (CASE WHEN w.city IS NOT NULL THEN w.city ELSE c.name END), w.temp_low, w.temp_high, w.prcp, c.location, w.date
FROM weather w FULL OUTER JOIN cities c
  ON (w.city = c.name);
```

```text
   name   | temp_low | temp_high | prcp |   location   |    date    
----------+----------+-----------+------+--------------+------------
 Beijing  |       18 |        32 | 0.25 | (116.3,39.9) | 2021-05-19
 Beijing  |       20 |        30 |    0 | (116.3,39.9) | 2021-05-20
 Dalian   |       16 |        24 |    0 |              | 2021-05-21
 Shanghai |          |           |      | (121.3,31.1) | 
(4 rows)
```

自连接：`weather`表与自己连接，找出同一城市，某一天的最低温度比另一天低的记录。

```sql
SELECT w1.city, w1.temp_low, w1.date, w2.temp_low, w2.date
FROM weather w1, weather w2
WHERE w1.city = w2.city
  AND w1.temp_low < w2.temp_low;
```

```text
  city   | temp_low |    date    | temp_low |    date    
---------+----------+------------+----------+------------
 Beijing |       18 | 2021-05-19 |       20 | 2021-05-20
(1 row)
```

**e) 聚集函数使用**

聚集函数针对多行输入计算一个结果。

下面，找出`weather`表中的历史最低温度。

```sql
SELECT min(temp_low) FROM weather;
```

```text
 min 
-----
  16
(1 row)
```

找出拥有这个历史最低温度的是哪个城市哪一天的记录。

```sql
-- 使用子查询
SELECT city, temp_low, date
FROM weather
WHERE temp_low = (SELECT min(temp_low) FROM weather);

-- 错误写法 聚集函数不允许在WHERE条件中使用
SELECT city FROM weather WHERE temp_low = min(temp_low); 
```

```text
  city  | temp_low |    date    
--------+----------+------------
 Dalian |       16 | 2021-05-21
(1 row)
```

聚集函数结合`GROUP BY`找出每个城市的历史最低温度。

```sql
SELECT city, min(temp_low)
FROM weather
GROUP BY city;
```

```text
  city   | min 
---------+-----
 Dalian  |  16
 Beijing |  18
(2 rows)
```

进一步找出每个城市历史最低温度低于17的记录。

```sql
SELECT city, min(temp_low)
FROM weather
GROUP BY city
  HAVING min(temp_low) < 17;
```

```text
  city  | min 
--------+-----
 Dalian |  16
(1 row)
```

从如上示例也看到了`WHERE`与`HAVING`使用场景的不同：`WHERE`用于分组和聚集函数使用前的输入行筛选；而`HAVING`用于分组和聚集函数使用后的分组行筛选。且`WHERE`语句中不可以使用聚集函数，而`HAVING`语句中一般总使用聚集函数（`HAVING`语句中不使用聚集函数的条件，不如直接将其移到`WHERE`语句中）。

如，接着上面，筛选首字母为`D`的城市，并返回这些城市历史最低温度低于17的记录。

```sql
SELECT city, min(temp_low)
FROM weather
WHERE city like 'D%'
GROUP BY city
  HAVING min(temp_low) < 17;

-- 不要用这种写法
SELECT city, min(temp_low)
FROM weather
GROUP BY city
  HAVING min(temp_low) < 17 AND city like 'D%';
```

```text
  city  | min 
--------+-----
 Dalian |  16
(1 row)
```

**f) 更新及删除**

假定`2021-05-20`及之后的数据录入时温度均比实际值低了1度，可以使用如下`UPDATE`语句进行校正。

```sql
UPDATE weather 
  SET temp_low = temp_low + 1, temp_high = temp_high + 1 
  WHERE date >= '2021-05-20';
```

重新查询数据。

```sql
SELECT * FROM weather;
```

```text
  city   | temp_low | temp_high | prcp |    date    
---------+----------+-----------+------+------------
 Beijing |       18 |        32 | 0.25 | 2021-05-19
 Beijing |       21 |        31 |    0 | 2021-05-20
 Dalian  |       17 |        25 |    0 | 2021-05-21
(3 rows)
```

若不再需要城市名为`Beijing`的数据，可以使用`DELETE`语句进行删除。

```sql
DELETE FROM weather WHERE city = 'Beijing';
```

```sql
SELECT * FROM weather;
```

```text
  city  | temp_low | temp_high | prcp |    date    
--------+----------+-----------+------+------------
 Dalian |       17 |        25 |    0 | 2021-05-21
(1 row)
```

若整个表的数据都不需要了，确认无误后，可以进行删除。

```sql
DELETE FROM weather;
```

### 2 高级特性

**a) 视图**

针对上面的场景，若天气与城市坐标总是一起展示，则可以为其创建视图，其使用跟普通的表一样。视图有许多好处，如隐藏表的细节，可以随着应用演进而不必更改接口定义。当然还可以在视图上创建视图。

```sql
CREATE VIEW myview AS
    SELECT w.city, w.temp_low, w.temp_high, w.prcp, c.location, w.date
    FROM weather w, cities c 
    WHERE w.city = c.name;

SELECT * FROM myview;
```

**b) 外健**

针对上述天气表`weather`与城市表`cities`，若我们要确保没有人可以在`weather`表插入`cities`表中不存在的城市的天气记录。这种约束即是保障数据的参照完整性，可以使用外健来实现。

新的表定义如下：

```sql
CREATE TABLE cities (
    name varchar(80) primary key,
    location point
);

CREATE TABLE weather (
    city varchar(80) references cities(name),
    temp_low int,
    temp_high int,
    prcp real,
    date date
);
```

现在尝试对`weather`表插入一个新的城市的天气记录，会报错。

```sql
INSERT INTO weather VALUES ('Tianjin', 22, 30, 0.0, '2021-05-22');
```

```text
ERROR:  insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL:  Key (city)=(Tianjin) is not present in table "cities".
```

在`cities`表补全该城市后，即可对`weather`进行插入。

```sql
INSERT INTO cities VALUES ('Tianjin', '(117.2, 39.1)');
INSERT INTO weather VALUES ('Tianjin', 22, 30, 0.0, '2021-05-22');
```

同理，`cities`表被参照，所以涉及被参考字段数据的更新及删除等都会受影响。

```sql
DELETE FROM cities WHERE name = 'Tianjin';
```

```text
ERROR:  update or delete on table "cities" violates foreign key constraint "weather_city_fkey" on table "weather"
DETAIL:  Key (name)=(Tianjin) is still referenced from table "weather".
```

**c) 事务**

**d) 窗口函数**

**e) 表继承**



> 参考资料
>
> [1] [PostgreSQL Tutorial](https://www.postgresql.org/docs/13/tutorial.html)