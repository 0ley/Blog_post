---
title: 攻防世界-mobile新手区
date: 2019-08-01 18:53:24
updated: 2019-08-01 18:53:24
tags:
    - 安卓逆向
    - 刷题
    - 攻防世界
    - ctf
categories: 
	- 逆向工程
	- 刷题
use:
  - Valine
text: true
lazyload: true
count: true
---

# 攻防世界-app1

# 1.模拟器运行

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564656270418-565a62c5-bbc5-42bb-bb9b-0dcc0869960a.png)

# 2.反编译

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564656480909-41478bd8-80ab-4bf9-87b2-5e0f59e3b04e.png)

# 3.分析

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564656525104-96f3a2cc-a380-4c14-8439-954c868d1993.png)

一个字符串异或了一个整数

# 4.寻找

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564656742865-ad0c493d-d5a6-429e-ad4f-e3806dd2c2b3.png)

# 5.编写脚本



```python
k = 'X<cP[?PHNB<P?aj' 
t = 15 
a ='' 
for i in range(len(k)): 
    a = a + chr(ord(k[i]) ^ t)
print (a)
```

# 6.得到flag