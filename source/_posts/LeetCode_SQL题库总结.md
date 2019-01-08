---
title: LeetCode
date: 2019-01-08 11:05:33
tags: [LeetCode,SQL]
---



针对LeetCode的数据库题目做出的一些总结：



SQL函数和关键字总结

- **mod(n1,n2)** 求余函数,函数返回值是 n1/n2 的余数（参照620）
- **UNION** 操作符用于合并两个或多个 SELECT 语句的结果集（参照595）
  - UNION 内部的 SELECT 语句必须拥有相同数量的列
  - 列也必须拥有相似的数据类型
  - 每条 SELECT 语句中的列的顺序必须相同
- **IF(Condition,A,B)**   Condition是一个表达式（参照 627）
  - 当Condition为TRUE时，返回A；当Condition为FALSE时，返回B
- **DISTINCT** ： 用于返回唯一不同的值（参考197）
  - distinct name,id 这样的mysql 会认为要过滤掉name和id两个字段都重复的记录
- 查询昨天： WHERE TO_DAYS(NOW( )) - TO_DAYS(时间字段名) <= 1
- **IFNULL(expression_1,expression_2)**
  - 如果expression_1不为NULL，则IFNULL函数返回expression_1；否则返回expression_2的结果





重点例题展示：

1. 题目序号：626 换座位

> 一张 `seat` 座位表，用来储存学生名字和与他们相对应的座位 id；id是连续递增的
>
> 要求：改变相邻俩学生的座位；如果学生人数是奇数，则不需要改变最后一个同学的座位

````sql
#UNION 会将三个查询的数据结合,最后按照id排序
select a.id, a.student from (
    # id%2=0 这个条件查询出偶数行的数据
    # id-1 是将偶数行的id变成奇数行
    select id-1 AS id, student from seat where (id%2) = 0
    UNION
    # id%2=1 这个条件查询出奇数行的数据
    # id+1 是将奇数行的id变成偶数行;并且这个id需要小于总行数
    select id+1 AS id, student from seat where id%2 = 1 
    and id < (select COUNT(id) from seat)
    UNION
    # 这里是将如果最后一行是奇数行,那么不用改变
    # id%2=1是奇数行;并且id需要等于总行数
    select id AS id, student from seat where id%2 = 1 
    and id = (select COUNT(id) from seat) 
)a order by a.id asc
````

