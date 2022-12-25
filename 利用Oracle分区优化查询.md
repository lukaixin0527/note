---
title: 利用Oracle分区优化查询
date: 2022-03-23 15:13:47
categories:
- 数据库
tags:
- 数据库
- Oracle
- 分区
---

表分区功能能够改善应用程序性能，提高数据库可管理性和可用性，是数据库管理非常关键的技术。

数据库通过使用分区提高查询性能，简化日常管理维护工作。　　

## 1 分区优点　　

1) 减少维护工作量，独立管理每个表分区比管理整个大表要轻松的多　　

2) 增加数据库的可用性，由于将数据分散到各个分区中，减少了数据损坏的可能性　　

3) 均衡I/O，减少竞争，通过把表的不同分区分配到不同的磁盘来平衡I/O改善性能　　

4) 分区对用户保持透明，用户感受不到它的存在　　

5) 提高查询速度，对于大表的DML操作可以分解到表的不同分区来执行，可以加快执行速度　　

## 2 分区缺点　　

已经存在的表，不能直接转化为分区表　　

## 3 什么时候使用分区表

1) 表的大小超过2GB

2) 表中包含历史数据，新的数据被增加到新的分区中 

## 4 分区类型

1) Range 分区

2) HASH分区(散列分区)

3) 列表分区

4) 组合分区(复合分区)


显示分区表信息

显示数据库所有分区表的信息：DBA_PART_TABLES

显示当前用户可访问的所有分区表信息：ALL_PART_TABLES

显示当前用户所有分区表的信息：USER_PART_TABLES

显示表分区信息 显示数据库所有分区表的详细分区信息：DBA_TAB_PARTITIONS

显示当前用户可访问的所有分区表的详细分区信息：ALL_TAB_PARTITIONS

显示当前用户所有分区表的详细分区信息：USER_TAB_PARTITIONS

显示子分区信息 显示数据库所有组合分区表的子分区信息：DBA_TAB_SUBPARTITIONS

显示当前用户可访问的所有组合分区表的子分区信息：ALL_TAB_SUBPARTITIONS

显示当前用户所有组合分区表的子分区信息：USER_TAB_SUBPARTITIONS

显示分区列 显示数据库所有分区表的分区列信息：DBA_PART_KEY_COLUMNS

显示当前用户可访问的所有分区表的分区列信息：ALL_PART_KEY_COLUMNS

显示当前用户所有分区表的分区列信息：USER_PART_KEY_COLUMNS

显示子分区列 显示数据库所有分区表的子分区列信息：DBA_SUBPART_KEY_COLUMNS

显示当前用户可访问的所有分区表的子分区列信息：ALL_SUBPART_KEY_COLUMNS

显示当前用户所有分区表的子分区列信息：USER_SUBPART_KEY_COLUMNS

## 5 实战

Oracle查询昨天日期

```sql
select to_char(sysdate - 1,'yyyy-MM-dd') from dual;
```

Oracle查询分区

```sql
SELECT * FROM all_PART_KEY_COLUMNS
```

![image-20221101093630834](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221101093630834.png)

根据分区字段 查询，速度提升

```sql
select count(1) from BD_NETCAR_OPERATEPAY partition where BILLTIME >to_timestamp(to_char(sysdate - 1,'yyyy-MM-dd'), 'yyyy-mm-dd hh24:mi:ss.ff')
```

