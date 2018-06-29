---
layout: post
title:  "mysql修改表"
date:  2016-03-30 9:39:00 +0800
categories: mysql
tags: mysql
keywords: mysql,web1992
---

mysql 备忘
---
<!--more-->

###mysql 表中使用存储过程添加新字段 


```sql
	-- 1, 注意SQL 语句开始处不要空格
	-- 2, 在使用 [--] 进行注释时，后面请加空格
	
	SET CHARACTER_SET_CLIENT=utf8;
	SET CHARACTER_SET_CONNECTION=utf8;
	drop procedure if exists schema_change;
	delimiter ';;';
	create procedure schema_change() begin
	if exists (select * from information_schema.columns where table_name = 't_my_table' and column_name = 'my_name') then
			alter table t_my_table drop column my_name;
	end if;
	alter table t_my_table add column my_name varchar(255) DEFAULT '';
	end;;
	delimiter ';';
	call schema_change();
	drop procedure if exists schema_change;

```sql

###mysql 删除表

```sql
	DROP TABLE IF EXISTS `t_my_table`;
```


```sql

   #查询表的信息
   select * from information_schema.columns where table_name ='t_payment_ysepay_wap_req';
   
   select c.column_name as '列名',c.data_type as '类型',substring_index( substring_index(c.COLUMN_TYPE,'\)',1) ,'\(',-1)  as '长度',c.column_comment as '备注'   from information_schema.columns as c   where table_name ='t_payment_ysepay_wap_req';
```