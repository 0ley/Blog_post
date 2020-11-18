---
title: 攻防世界-reverse高手区
date:  2019-08-02 12:54:20
updated: 2019-08-13 12:41:57
tags:
    - 逆向
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

# 攻防世界-dmd-50

# 1.下载文件

发现是无壳的elf文件

# 2.ida64打开

找到主函数，发现是c++写的

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563865825215-35d4486c-73d4-4c26-9aa8-3184d77224aa.png)

![image.png](https://cdn.nlark.com/yuque/0/2019/png/403807/1563865854420-27cef422-43d1-4af4-8ea1-0eca1e87566c.png)

先将出现的数字都r掉，得到一串数字和一个判断语句，得知key为flag

分析代码，点进md5函数查看

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563865942737-220d4983-65be-4998-8be3-9c889d58dc1e.png)

猜测为md5加密

找到MD5在线解密网站，把字符串解密，得到：**grape**

**提交，****错误**

百度得知md5(md5($pass)) 的具体含义找出正确答案

md5(md5($pass)) ：第一次加密后，结果转换成小写，对结果再加密一次

则解密为：解密一次后，再解密一次

所以将grade加密（MD5）

得到：**b781cbb29054db12f88f08c6e161c199**

**提交，****正确**





ps.对flag进行再次加密，果然得到了最开始的字符串

# 攻防世界-Shuffle

# 1.下载文件

打开发现为无壳32位elf文件

# 2.ida打开

找到主函数，记得提示“找到字符串在随机化之前.”

f5，并且将数字r掉

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563866615081-b973f9c9-d184-4201-9be0-a0aefd925c09.png)

得到flag（这也太简单了？？？？）

提交，正确

# 攻防世界-re2-cpp-is-awesome

# 1.下载文件

发现是64位elf文件

# 2.ida64打开

找到主函数，f5

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563870194900-936bb5ac-ca75-4d45-a51d-a3eeaa8ef9cb.png)

分析，找到主要比较语句

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563870236625-b98ef842-b1ce-4633-9459-74feda58ea13.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563870250500-b46aaf5a-2ac4-4285-beaf-c2d1900900bf.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563870272893-3781fcf9-c6b4-47a9-9313-7bddb5b819c4.png)

# 3.编写相应脚本



```python
flag=''
s='L3t_ME_T3ll_Y0u_S0m3th1ng_1mp0rtant_A_{FL4G}_W0nt_b3_3X4ctly_th4t_345y_t0_c4ptur3_H0wev3r_1T_w1ll_b3_C00l_1F_Y0u_g0t_1t'
ss=[0x24, 0x0, 0x5, 0x36, 0x65, 0x7, 0x27, 0x26, 0x2d, 0x1, 0x3, 0x0, 0xd, 0x56, 0x1,0x3, 0x65, 0x3, 0x2d, 0x16, 0x2, 0x15, 0x3, 0x65, 0x0, 0x29, 0x44, 0x44, 0x1, 0x44, 0x2b]
for i in ss:
    flag+=s[i]
print(flag)
```

运行，得到flag

# 攻防世界-crackme

# 1.下载文件

运行，需要输入flag

# 2.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563929081845-d38b372a-b9e3-4d9c-b7b1-802f51172e95.png)

发现有壳

# 3.用od打开

esp定律找oep，dump，修复：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564475731749-78404a32-f973-4fcf-99c7-6963b7f0af17.png)

# 4.重新载入od

调试，发现在此地址函数运行时：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564478749611-e2e93eac-1a43-4b82-a372-50345f4ab501.png)

重新调试，这次跟进函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564478887241-cab02ae8-7e33-4ad8-9f23-f8468cd26a04.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564478922436-2af13692-7e60-41ec-845c-a8d918be9733.png)

# 5.单步调试

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564479910841-08f6a6e9-087e-4bab-8873-8a08d54a393b.png)

此处比较后，跳转不实现，直接error（猜测为比较长度），修改为跳转实现

继续调试：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564480011693-014d30bf-4bac-42cb-bd77-473a5fa4f7a0.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564480225938-ae2cd27d-a93c-42d1-8f8b-e70cabfb25f3.png)

该处比较后跳到error，猜测比较字符串藏在这附近

# 6.继续调试

无果

# 7.ida打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564480768554-2e41172b-98c3-46fe-a750-7c59c3f3a33f.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564481017993-bb45d3a3-bb90-476d-87e1-b86d5e001b5d.png)

# 8.编写脚本



```python
flag=''
a='this_is_not_flag'
b=[0x12,0x04,0x08,0x14,0x24,0x5C,0x4A,0x3D,0x56,0x0A,0x10,0x67,0x0,0x41,0x0,0x01,0x46,0x5A,0x44,0x42,0x6E,0x0C,0x44,0x72,0x0C,0x0D,0x40,0x3E,0x4B,0x5F,0x02,0x01,0x4C,0x5E,0x5B,0x17,0x6E,0x0C,0x16,0x68,0x5B,0x12]
c=0
while c<42:
    flag+=str(chr(ord(a[c%16])^b[c]))
    c=c+1
print(flag)
```

得到flag

# 攻防世界-re-for-50-plz-50

# 1.下载文件，发现是elf

# 2.ida打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564721361241-9de22acc-8fa3-4230-88bc-ad2f34eda82c.png)

发现和以往所见的汇编语言不一样，且无法反编译

# 3.百度

发现是mips指令集下的汇编，若想反编译，需要下载插件

# 4.学习后看ida代码

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564721462445-350e13ff-5331-409f-af3d-8312bd330f15.png)

发现一串字符串异或了0x37

# 5.猜测为flag，编写脚本



```python
flag=''
a="cbtcqLUBChERV[[Nh@_X^D]X_YPV[CJ"
for i in a:
    flag+=chr(ord(i)^0x37)
print(flag)
```

成功得到flag

# 攻防世界-key

# 1.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564722994024-7c91721d-eb48-4bd7-a2a7-6d288aa47716.png)

无壳，且无法运行

# 2.od打开

查找字符串：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723087989-355d9f2f-7159-4362-b773-15c65f2feda6.png)

# 3.单步调试

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723338330-34ec4ae7-1696-4e30-aa1f-f4f3a74b0000.png)

发现一串很奇怪的东西

继续调试

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723553851-b5b191c5-d225-4274-91d5-cc7eef90a9eb.png)

又是一串字符串

继续调试，修改了两个跳转指令（不修改就会结束时就修改）后，发现输出了：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723701484-8c2ed5fb-13ee-46b0-9edb-6d6cd74371c6.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723752112-83196c47-3195-4489-b36d-bdacb6552cd1.png)

# 4.ida打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723923159-5b78ced5-8339-4a45-bf3c-c799f1157a09.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723938859-9e33d90c-e295-4925-a464-64d8b263bccd.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723962108-f21b5752-fc60-4a45-855e-f4fc8a07b6c8.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723984458-cb1b979c-e9ad-4ec0-b7b9-134d93b60cfd.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564723996846-ae5da33c-350d-4cb4-b62b-5f7a2bb6f5cd.png)

# 5.分析

刚开始是一堆初始化，发现两个字符串

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564724098990-475f73fe-e943-429c-b9b1-31c1704bc32b.png)

猜测这里是加密

# 6.写个脚本跑一下



```python
str1 = "themidathemidathemida"
str2 = ">----++++....<<<<."
 
t =""
flag=""
for i in range(18):
    t += chr((ord(str1[i]) ^ ord(str2[i]))+22)
for i in t:
    flag+=chr(ord(i)+9)
print(flag)
```

居然得到了flag

# 7.od再打开

这次先看ida里的地址（记得打开）

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564728618977-1195cedb-9368-47d4-9f24-f69ae5b54357.png)

在此处下断，运行

发现下面两个循环：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564728660554-111b4058-a7dc-4951-90b8-38d8ebeb84e5.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564728678543-fcc9bbbe-a7ea-4ed9-b361-68e1f4f517b1.png)

# 8.在od中调试

跟踪完两个循环后得到两个字符串，其中一个为flag

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564728803181-f7282805-0604-4773-b750-0f40dcba11d6.png)



![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564728708007-a986788a-0157-431b-929f-d0772c9f62f2.png)

# 攻防世界-simple-check-100

# 1.下载

发现是一个压缩包，解压得到一个exe，两个elf

# 2.查壳

无壳

# 3.ida打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564732242039-b0ee27ff-059a-47be-8330-8e16c2df6415.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564732271715-4cec5a19-f3f5-4e19-a1ca-b15df5de0919.png)

发现有一个check函数，记住地址

# 4.od打开

转到地址，nop掉函数

# ![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564732306846-406d5b36-2070-472a-88aa-65408af7b1a8.png)5.单步调试

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564732321862-be14d99d-455f-44cb-98a1-6b9731ab27de.png)

乱码？？？？？？？

# 6.elf文件拖入ida

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564732476056-86c4eac5-6295-447b-a2fb-fc7b7ab82793.png)

发现主函数一样

# 7.ida动态调试

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564732678179-f288652e-467f-4e3d-bfd5-54a4985881f2.png)

相同地点下断

nop掉后运行

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564732913794-7622a1dc-f040-4c90-a0a4-bae08b14d236.png)

得到flag

# 攻防世界-re1-100

# 1.下载，解压

发现为64位elf文件

# 2.在linux下运行

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564790351545-bd258195-ca55-49f2-9c8b-06c1625493cb.png)

# 3.ida64打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564790377736-850631e1-d42d-4122-a8e0-bae64fd8a4e7.png)

找到关键比较代码

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564790407216-8ac058c8-b5ac-4cf4-a1f9-9a8be85145af.png)

对输入进行重组

猜测{daf29f59034938ae4efd53fc275d81053ed5be8c}

重组后反ascii码为flag

# 4.写脚本

发现跑不出来，直接提交成功（不是ascii码！！）

# 攻防世界-IgniteMe

# 1.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564793125962-928f7717-9ea0-460a-a9fb-3a073558c4c5.png)

无壳

# 2.ida打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564793145938-c2cbf5b0-f9c0-4ea2-9ec3-02101c58e82e.png)

# 3.分析

flag格式为EIS{}

且

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564793192037-6e3583f8-9635-46d2-9e0d-6027ae5c66c8.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564793209623-fcb989ef-3298-46e1-ac30-2706181ffce4.png)

先大小写转化，在运算跟一个字符串相比

# 4.写脚本



```python
flag=''
t1=''
t2=''
a=0
s1="GONDPHyGjPEKruv{{pj]X@rF"
s2=[0x0D,0x13,0x17,0x11,0x02,0x01,0x20,0x1D,0x0C,0x02,0x19,0x2F,0x17,0x2B,0x24,0x1F,0x1E,0x16,0x09,0x0F,0x15,0x27,0x13,0x26,0x0A,0x2F,0x1E,0x1A,0x2D,0x0C,0x22,0x04]
for i in s1:
    t1+=chr(ord(i)^s2[a])
    a=a+1
for i in t1:
    t2+=chr((ord(i)-72)^0x55)
print(t2.lower())
```

先不转化大小写输出发现全是大写，那直接全转为小写即可，加上格式得到flag

