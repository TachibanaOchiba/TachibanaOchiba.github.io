---
title: SQLZoo游玩记录
tags:
  - sql
categories:
  - 技术宅的日常
date: 2020-05-05 00:15:56
---


<!--在此处插入概述-->

（ry

<!--more-->

<!--以下为正文-->

## 查询

### 在某范围中匹配
- `IN(a,b,c)`
```sql
SELECT persons from employee WHERE lastname IN ('Tachibana','橘');
```
- `BETWEEN a AND b`
```sql
SELECT persons FROM employee WHERE commencement BETWEEN 2011 AND 2020; # 含本数
```
### 计算顺序
- `()` > `AND` > `OR`
    - Example: show the name of all players who scored a goal against Germany.
    ```sql
    SELECT DISTINCT player
    FROM goal LEFT JOIN game ON goal.matchid = game.id
    WHERE (team1 = 'GER' OR team2 = 'GER') AND teamid != 'GER' # NOT:  WHERE team1 = 'GER' OR team2 = 'GER' AND teamid != 'GER'
    ```
- `CASE`
    - Syntax: ` CASE WHEN CondA THEN a WHEN [B] THEN b ELSE c END`
    - Example: List every match with the goals scored by each team
        ```sql
        SELECT mdate,team1,
            SUM(CASE
                WHEN teamid = team1 THEN 1
                ELSE 0
                END) AS score1,
            team2,
            SUM(CASE
                WHEN teamid = team2 THEN 1 ELSE 0
                END) AS score2
        FROM game LEFT JOIN goal ON game.id = goal.matchid
        GROUP BY id
        ORDER BY mdate,matchid,team1,team2
        ```
- `COALESCE(*args)`
    - Show teacher name and mobile number or '07986 444 2266'
        ```sql
        SELECT name,COALESCE(mobile,'07986 444 2266') FROM teacher;
        ```
###  模糊匹配：
- 简单模式可以使用`LIKE`匹配,复杂匹配可以使用`REGEXP`用正则表达式匹配。
#### LIKE
```sql
SELECT* FROM nobel WHERE winner LIKE 'C%' AND　winner LIKE `%n';
```
#### 正则表达式 (REGular EXPressions)
```sql
SELECT * FROM nobel WHERE winner REGEXP '^Cn$';
```
##### 简单易懂的正则表达式使用方法

###### 字符
- `\d`: 数字
- `^[A-Za-z]`:字母
- `[\u4e00-\u9fa5]`：汉字
- `\s`：单个空字符。常见：换行符`\n`, 回车`\t`， Tab`\t`
###### 语法
- `^p`: 以p开头
- `p$`: 以p结尾
- `[abcde]`： abcde中任意一个
- `a|b`： a或者b
- `f*`:匹配`f`零次或多次
- `f+`:匹配`f`一次或多次
- `f{n,m}`: 匹`f` n次至m次。m可以留空，表示匹配n次。
- `()`: 计算顺序。例如`(o|oc)hiba`匹配`ochiba`或者`chiba`.

### 套娃
#### 嵌套子查询 (Nested subqueries)
- 一条语句的一部分是另一条语句（套娃）
    - 例子：Show the years when a Medicine award was given but no Peace or Literature award was
    ```sql
    SELECT yr
    FROM nobel
    WHERE 
        subject = 'Medicine' 
        AND yr NOT IN (Select year FROM nobel WHERE subject = 'Literature')
        AND yr NOT IN (Select year FROM nobel WHERE subject = 'Peace')
    ```
#### 相关子查询(Correlated subqueries)
- 子查询根据母查询的需求运行多次
    - 例子：Find the largest country (by area) in each continent, show the continent, the name and the area:
    ```sql
    SELECT continent, name, area
    FROM world x
    WHERE area >= ALL
    (SELECT area FROM world y WHERE x.continent = y. continent)
    ```
### 输出格式
#### 设定列名：`AS`
``` sql
SELECT name,max(grade) AS best, min(grade) AS worst
FROM students
GROUP by name
HAVING min(grade) > 80
```
#### 数值
- 舍去小数点：`ROUND`
#### 文本
- 连接字符：`CONCAT`
    - 例子：Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.
    ```sql
    SELECT name, CONCAT(ROUND(population * 100/ (SELECT population FROM world WHERE name = 'Germany')),'%') AS percentage
    FROM world 
    WHERE continent = 'Europe'
    ```
#### 排序 `ORDER BY`
- `DESC`: 降序； `ASC`:升序
    ```sql
    SELECT ename, salary FROM employee ORDER BY ename ASC
    ```