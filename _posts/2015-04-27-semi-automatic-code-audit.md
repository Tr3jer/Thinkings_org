---
layout: post
title: 半自动化PHP代码审计思路与环境搭建
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

#### 代码审计常见流程:

<img src="http://pfr2vvlbk.bkt.clouddn.com/2132b1y3k21.png">

	(1)通过可控变量(输入点)回溯危险函数
	(2)查找危险函数确定可控变量
	(3)传递的过程中触发漏洞
	
#### 思考:
<img src="http://pfr2vvlbk.bkt.clouddn.com/32ee43tgreg.png">

#### 最佳出发点:
**枚举所有`可控变量(输入点)` & `危险函数`,包括但不限于:**

<img src="http://pfr2vvlbk.bkt.clouddn.com/ahs98isuaf.png">

> * 大型Web程序逻辑越复杂越易出现漏洞,这就需要代码审计人员拥有较强的`代码理解度`和`逻辑思维`,每一段代码都有可能辅助触发漏洞,当代码涉及很多文件时,比如在`MVC`中,`Action`层代码可能会涉及几个甚至几十个文件,这些文件包括框架的`核心配置初始化文件`,`类文件`,`路由控制文件`等,如果用手工挖掘漏洞会比较累,另外,某些应用可能比较复杂,我们并不清楚流程是怎样的,该去哪里追踪变量.
> * So,合理的利用半自动化对于代码审计来说即是事半功倍.

#### 环境:

> `MAMP:`**使用它的主要原因是很Easy的切换环境版本(Win下的PhpStudy同理),想要深入挖掘漏洞就要同事根据环境版本的不同审计代码中特有的漏洞.[建议自己先尝试手动配置多版本与各模块,Enjoy折腾时的乐趣,正所谓可以不自己造轮子,但要了解轮子的原理:)]**

> `PhpStorm:`**这个IDE就不多说了,做PHP开发的都说好.**

> `Xdebug:` **开源的PHP程序调试工具,用来跟踪,调试和分析PHP程序的运行状况,毋庸置疑,不论应用到开发调试还是代码审计都称得上是神器**

#### Article:
<a target="_blank" href="http://www.thinkings.org/public/osx_mamp_phpstorm_setting_xdebug.pdf">OSX PHPSTORM+MAMP 配置 Xdebug</a>

#### Other:

* <a target="_blank" href="http://raveren.github.io/kint/">Kint</a>
* <a target="_blank" href="https://www.bluestatic.org/software/macgdbp/">MacGDBp</a>
* <a target="_blank" href="http://krumo.kaloyan.info/">Krumo</a>
* <a target="_blank" href="http://sourceforge.net/projects/php-dyn/">PHP_Dyn</a>
* <a target="_blank" href="http://www.debugbar.com/">DebugBar</a>