---
layout: post
title: MetaDockers
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

> Vulhub Team做为以收集/制作docker漏洞靶场为基础，并发展Docker相关的开发，MetaDockers用于管理vulhub以及自实现的Docker可视化。

> <a target="_blank" href="https://github.com/Tr3jer/dnsAutoRebinding">vulhub</a> - Docker-Compose file for vulnerability environment.

> <a target="_blank" href="https://github.com/Tr3jer/dnsAutoRebinding">MetaDockers</a> - Responsible for visualization the vulhub or docker.

## setting：
> 配置vulhub path和docker api开关，docker若启动且加入环境变量那么默认即可docker.from_env()：
MetaDockers/controller/lib/config.conf

sudo pip install -r requirements.txt

cd MetaDockers/

python manage.py runserver 8000

## Index:
![](http://pfr2vvlbk.bkt.clouddn.com/gerggf.png)
## Vulhubs:
> 这个启动项没写，后期会直接拆分docker-compose包融合进去，还有各个靶场的README未分配。

![](http://pfr2vvlbk.bkt.clouddn.com/dsfwqe221.png)
## Images:
![](http://pfr2vvlbk.bkt.clouddn.com/regregeh.png)
> 还有这个images的启动参数有50多个。。。挨个测试心态炸了，好心人可以按[api文档](http://docker-py.readthedocs.io/en/stable/containers.html)加下哈哈（涉及controller/lib/dockerOperation.py的image_operation函数和templates/images.html的$(".btn-run").click），没人的话有空我也会加上：

![](http://pfr2vvlbk.bkt.clouddn.com/3rvyjar.png)
## Networks:
![](http://pfr2vvlbk.bkt.clouddn.com/43tyo8dsf.png)

> 可增加删除Networks：

![](http://pfr2vvlbk.bkt.clouddn.com/thth.png)

![](http://pfr2vvlbk.bkt.clouddn.com/ewtewr.png)
## Volumes:
![](http://pfr2vvlbk.bkt.clouddn.com/32r32r3r2.png)

> 可增加删除volumes：

![](http://pfr2vvlbk.bkt.clouddn.com/rwiubsdf.png)
## Containers:

> 这个Containers才是重点，解决了在使用Container时的几个常见问题，比如因child导致kill不掉等。

![](http://pfr2vvlbk.bkt.clouddn.com/dsfwwefrew4.png)

> 正经的功能都有，还可直接跳转到映射的端口地址，其他功能可以摸索下：

![](http://pfr2vvlbk.bkt.clouddn.com/sadbkyuasd.png)

> 该Container的Log：

![](http://pfr2vvlbk.bkt.clouddn.com/wg7vyksdf.png)

> 该Container的Info：

![](http://pfr2vvlbk.bkt.clouddn.com/3qrg7iqw.png)
## Docker Info:

> 层叠太多了，直接用的json view，浏览器ctrl+f吧2333.

![](http://pfr2vvlbk.bkt.clouddn.com/3rb78sdfk.png)

