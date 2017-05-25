---
layout: post
title: Auto Hooks Spider
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

> 好像好久没发东西了，抽空研究自动化，一部分是如何有效获取自动化的food，想了很多也实现出了很多，比如一些常规手法、PassiveDNS等等，尤其后者虽然有效果但发现有个弊端，就是说从这种底层记录进行筛选贼TMD烧硬件。不然我就想去拆小区交换机了= =。倒不如实现个自动spider，省了来回点啊点。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实现很简单，因为我本地实现的比较复杂，所以是把它拆分下改成现成的发出来。还有就是代理的实现我没放进去，想爬墙外的话可以改下requests模块，拆分url判断是否存在于gfw列表就行了。

![](http://7xiw31.com1.z0.glb.clouddn.com/4rfedsxz.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;个人收集的有价值主域名1000+，hooks.txt先放200个你们玩。

![](http://7xiw31.com1.z0.glb.clouddn.com/4trefds.png)

* <a target="_blank" href="https://github.com/Tr3jer/AutoHookSpider">https://github.com/Tr3jer/AutoHookSpider</a>

```
AutoHookSpider
├── LICENSE
├── README.md
├── hooks.txt   #hooks字典，随机放了200个，可以自己收集。
├── lib
│   ├── __init__.py
│   ├── common.py   #琐碎功能
│   └── record.sql  #先在Mysql创建这个表
├── main.py #主程序
└── requirements.txt

sudo pip install -r requirements.txt
lib/record.sql into mysql
usage: python main.py {Options}[ google.com,twitter.com,facebook.com | -t 20 ]
或者直接python main.py会直接在hooks.txt抽取(thread_cnt)个入口域名。
```

> 比如你想专门抓一个公司的子域名，那么将hooks放入该inc的主域名直接运行就ok。


