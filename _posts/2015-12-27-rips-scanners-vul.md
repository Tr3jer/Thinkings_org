---
layout: post
title: Rips Scanners 本地文件读取漏洞分析
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**RIPS是一个源代码分析工具，它使用了静态分析技术，能够自动化地挖掘PHP源代码潜在的安全漏洞。渗透测试人员可以直接容易的审阅分析结果，而不用审阅整个程序代码。由于静态源代码分析的限制，漏洞是否真正存在，仍然需要代码审阅者确认。RIPS能够检测XSS, SQL注入, 文件泄露, LFI/RFI, RCE漏洞等。**

>/rips-0.55/windows/code.php 第102行

	$file = $_GET['file'];
	$marklines = explode(',', $_GET['lines']);
	$ext = '.'.pathinfo($file, PATHINFO_EXTENSION);

	
	if(!empty($file) && is_file($file) && in_array($ext, $FILETYPES))
	{
		$lines = file($file); 
		
		// place line numbers in extra table for more elegant copy/paste without line numbers
		echo '<tr><td><table>';
		for($i=1, $max=count($lines); $i<=$max;$i++) 
			echo "<tr><td class=\"linenrcolumn\"><span class=\"linenr\">$i</span><A id='".($i+2).'\'></A></td></tr>';
		echo '</table></td><td id="codeonly"><table id="codetable" width="100%">';
		
		$in_comment = false;
		for($i=0; $i<$max; $i++)
		{				
			$in_comment = highlightline($lines[$i], $i+1, $marklines, $in_comment);
		}
	} else
	{
		echo '<tr><td>Invalid file specified.</td></tr>';
	}

>$file为GET过来的，验证是否为文件，如果是就将其传入到数组，然后打印出来，算是本地文件读取吧。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**翻到RIPS前版本发现漏洞是同样存在的。**

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/54rtgdfcvx.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**相比来看利用的条件比前版本鸡肋一点，新版本验证了扩展名，而前版本可以读取文件的范围更广。**

	$ext = '.'.pathinfo($file, PATHINFO_EXTENSION);
	if(!empty($file) && is_file($file) && in_array($ext, $FILETYPES))

>跟踪到/rips-0.55/config/general.php 第49行

	$FILETYPES = array(						// filetypes to scan
		'.php', 
		'.inc', 
		'.phps', 
		'.php4', 
		'.php5', 
		//'.html', 
		//'.htm', 
		//'.txt',
		'.phtml', 
		'.tpl',  
		'.cgi'
	); 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**以上为RIPS新版本允许读取文件的扩展名，最后来个利用:**

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/5555rrfdd.png">