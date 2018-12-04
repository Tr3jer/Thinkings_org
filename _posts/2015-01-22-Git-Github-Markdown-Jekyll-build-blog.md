---
layout: post
title: Git+Github+Markdown+Jekyll Build a Blog
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>
<center>
<img src="https://ww2.sinaimg.cn/mw690/9c5c5d93tw1erp3be7l47j20ya0jhdgg.jpg">
</center>

#### 0x01<科普>:
>`Github & Git`:是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目,既然能逛到我的Blog就说明你不可能不知道:)
>`Markdown`:Markdown是一种可以使用普通文本编辑器编写的标记语言，通过类似HTML的标记语法，它可以使普通文本内容具有一定的格式。
>`Jekyll`:jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。

	以上是Wikipedia的术语.

首先引用下阮一峰所总结的三个阶段:

* 刚接触Blog,觉得很新鲜,试着选择一个免费空间来写.
* 现免费空间限制太多,就自己购买域名和空间,搭建独立博客.
* 得独立博客的管理太麻烦,最好在保留控制权的前提下,让别人来管,自己只负责写文章.

#### 0x02<开始搭建>:
在本机创建一个目录.

	$ mkdir demo
	
对该目录进行git初始化.

	$ cd demo
	$ git init
	
然后，创建一个分支gh-pages,因为github规定，只有该分支中的页面，才会生成网页文件.

	$ git checkout --orphan gh-pages
	

以下所有动作，都在该分支下完成.

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/32refds.png">

在项目根目录下，建立一个名为`_config.yml`的文本文件。它是jekyll的设置文件，我们在里面填入如下内容，其他设置都可以用默认选项，具体解释参见[官方网页](https://github.com/jekyll/jekyll/wiki/Configuration)。

	baseurl: /demo
	
接着就需要自定义模版了,根目录创建`_layouts`文件夹,cd进去后创建一个默认模版文件`default.html`:

	<!DOCTYPE html>
	<html>
	<head>
	
	<meta http-equiv="content-type" content="text/html; charset=utf-8" />
	<title>{% raw %}{{ page.title }}{% endraw %}</title>

	</head>
	<body>
	   {% raw %}{{ content }}{% endraw %}
	</body>
	</html>
	
然后就是在项目根目录创建一个文章存放的目录了,命名规范为`_post`

	$ mkdir _post
	$ cd _post
	
>进入该目录后,创建第一篇文章。文章就是普通的文本文件，文件名假定为2015-01-22-hello-world.html。(注意，文件名必须为"年-月-日-文章标题.后缀名"的格式。如果网页代码采用html格式，后缀名为html；如果采用markdown格式，后缀名为.md

在该文件中，填入以下内容：（注意，行首不能有空格）

	---
	layout: default
	title: 你好，世界
	---
	<h2>
	{% raw %}{{ page.title }}{% endraw %}
	</h2>
	
	<p>我的第一篇文章</p>
	
	<p>{% raw %}{{ page.date | date_to_string }}{% endraw %}</p>
	
>每篇文章开头必须有[yaml文件头](https://github.com/jekyll/jekyll/wiki/YAML-Front-Matter)表示，用来设置一些元数据。它用三根短划线"---"，标记开始和结束，里面每一行设置一种元数据。layout表示该文章使用default模板，标题为"你好，世界"。如果不设置这个值，默认使用嵌入文件名的标题，即"hello world"。在yaml文件头后面，就是文章的正式内容，里面可以使用模板变量。{% raw %}`{{ page.title }}`{% endraw %}就是文件头中设置的"你好，世界"，{% raw %}`{{ page.date }}`{% endraw %}则是嵌入文件名的日期（也可以在文件头重新定义date变量），`date_to_string`表示将page.date变量转化成人类可读的格式.

回到项目根目录,创建主页文件`index.html`

	---
	layout: default
	title: Index
	---
	<h2>{% raw %}{{ page.title }}{% endraw %}</h2>
	<p>New Posts</p>
	<ul>{% raw %}
	  {% for post in site.posts %}
	  <li>{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
	{% endfor %}{% endraw %}
	</ul>
	
>首页使用了{% raw %}`{% for post in site.posts %}`{% endraw %}，表示对所有帖子进行一个遍历。这里要注意的是，Liquid模板语言规定，输出内容使用两层大括号，单纯的命令使用一层大括号。至于{% raw %}`{{site.baseurl}}`{% endraw %}就是_config.yml中设置的baseurl变量。

此时的目录结构:

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/6rythfg.png">

>其实也可以直接用Jekyll New一个项目,然后将其覆盖/修改,同时也可以开启Jekyll服务进行本地调试.

本地安装Jekyll:

	gem install jekyll

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/5rthfgbv.png">

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/43etrdgf.png">

#### 0x03<提交Pages>:

	$ git add .
	$ git commit -m "frist post"
	$ git remote add origin https://github.com/username/demo.git
	$ git push origin gh-pages
	
push成功后大约十分钟生效,同时生效了的话Github也会发`Page build warning`的Mail提醒.

#### 0x04<绑定域名>:

>在项目上创建名为CNAME文件,内容就是你的域名,然后在域名控制面板将DNS创建两个A纪录,并指向192.30.252.153,绑定成功后,生效时间要看域名解析时间.

#### 0x05<扩展阅读>:

>[Github pages](https://pages.github.com/)

>[Jekyll](http://jekyllcn.com/)

>[搭建一个免费的，无限流量的Blog—-github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)

>[通过GitHub Pages建立个人站点(详细步骤)](http://www.cnblogs.com/purediy/archive/2013/03/07/2948892.html)

>[Jekyll官方文档](http://jekyllcn.com/docs/home/)

>[Jekyll Pagination](http://jekyllrb.com/docs/pagination/)

>[Jekyll 中的语法高亮：Pygments](http://havee.me/internet/2013-08/support-pygments-in-jekyll.html)

>[使用 GitHub, Jekyll 打造自己的免费独立博客](http://blog.csdn.net/on_1y/article/details/19259435)

>[评论功能](https://disqus.com/)