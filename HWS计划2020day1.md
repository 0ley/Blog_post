---
title: HWS计划2020day1
date:   2020-10-28 15:10:51
updated:  2020-10-28 15:10:51
tags:
    - 逆向
    - crypto
    - ctf
    - 智能合约
categories: 
	- HWS计划2020
use:
  - Valine
text: true
lazyload: true
count: true
---

# de4dot去混淆

使用dnspy打开crackme，发现存在混淆，比较难分析：

![image.png](https://i.loli.net/2020/11/17/wTYxR5mofC9EVDA.png)



命令行使用de4dot去混淆：
![image.png](https://i.loli.net/2020/11/17/I6KBp7aOMSJd1x8.png)

再拖入dnspy

![image.png](https://i.loli.net/2020/11/17/oYO7MxkDhIlv1GN.png)

# 提取pyc文件

Py2exe---使用unpy2exe

pyInstaller---使用pyinstxtrator

ida打开login.exe，看到有很多pyxxxx字符串，说明是由python打包的程序

![image.png](https://i.loli.net/2020/11/17/7T9x3NKHwZ6fWJ1.png)

提取pyc文件：

首先使用unpy2.exe：
![image.png](https://i.loli.net/2020/11/17/I9SwKpX1PqN7OCW.png)

不行，尝试另一个：
![image.png](https://i.loli.net/2020/11/17/sJwvKx8b47YynZh.png)

得到：
![image.png](https://i.loli.net/2020/11/17/wYacWFhDGIbSxTs.png)

分析login二进制文件：

![image.png](https://i.loli.net/2020/11/17/TjzgROSv7carQo3.png)

缺少魔数，那就补上缺少的：

![image.png](https://i.loli.net/2020/11/17/FlXqYTBO8zHCvUa.png)

反编译：

![image.png](https://i.loli.net/2020/11/17/9CdVipXzf3D2l6h.png)

# 部分古典密码

##  XXencode编码

![image.png](https://i.loli.net/2020/11/17/gw5e93hZMQ72GLq.png)

## UUencode编码

![image.png](https://i.loli.net/2020/11/17/LoHTamijrcS2QOD.png)

## shellcode编码

![image.png](https://i.loli.net/2020/11/17/l2ESX6LambHQJfA.png)

## Escape/Unescape编码

![image.png](https://i.loli.net/2020/11/17/ocHOt8a3UJRlsY5.png)

## HTML实体编码

![image.png](https://i.loli.net/2020/11/17/tv7XzUaKBo9Qp8J.png)

## 敲击码

![image.png](https://i.loli.net/2020/11/17/xvjl6wcTsG3DSP4.png)



## 曲路密码

- 明文填入一张表中，并按照一定的曲路遍历
- 攻击方法：逆向遍历

![image.png](https://i.loli.net/2020/11/17/G6eCVYHi8gAjnZD.png)

## 云影密码

![image.png](https://i.loli.net/2020/11/17/DeMhlvoKgC1dk47.png)

## 栅栏密码

![image.png](https://i.loli.net/2020/11/17/QFGLOaBCARsmUDi.png)

## 埃特巴什码

![image.png](https://i.loli.net/2020/11/17/RhxGmfNYK1AMkei.png)

## 培根密码

![image.png](https://i.loli.net/2020/11/17/UWRpJvZOBMfy651.png)

## 猪圈密码

![image.png](https://i.loli.net/2020/11/17/gZJvU6OsqiarjzB.png)

## 跳舞的小人

![image.png](https://i.loli.net/2020/11/17/HunSj5AUozcxfe9.png)

## 维吉尼亚密码

![image.png](https://i.loli.net/2020/11/17/NEIM8QKjroRhU7w.png)

# 智能合约初步

MetaMask插件+三个网址

https://etherscan.io/

http://remix.ethereum.org/#optimize=false&evmVersion=null&version=soljson-v0.6.6+commit.6c089d02.js

https://ropsten.etherscan.io/

## MetaMask插件初见

### 选择ropesten测试网络

![image.png](https://i.loli.net/2020/11/17/BAy6vsD5cxkuI1C.png)

### 选择buy--测试水管

![image.png](https://i.loli.net/2020/11/17/12dunBHSaqvXJAt.png)

### 获取测试币

![image.png](https://i.loli.net/2020/11/17/I1ueUHjgpoBx7MG.png)

![image.png](https://i.loli.net/2020/11/17/PGutb1czg5jQhmR.png)

### 创建账户2

![image.png](https://i.loli.net/2020/11/17/xAFclRt7qivf9oJ.png)

### 测试转账

![image.png](https://i.loli.net/2020/11/17/51fCAJ4Ws92oIgS.png)![image.png](https://i.loli.net/2020/11/17/6LiTVeczjbsUuY1.png)

## remix网站使用

### 部署代码

![image.png](https://i.loli.net/2020/11/17/voQPMS9zecmgE3f.png)![image.png](https://i.loli.net/2020/11/17/ejidnmLvHCBz1AM.png)

### 选择合适版本编译

![image.png](https://i.loli.net/2020/11/17/f6wh3xWSMmoaLiy.png)

### 获取账号并部署

![image.png](https://i.loli.net/2020/11/17/WNLqbw1Fpoit4l2.png)![image.png](https://i.loli.net/2020/11/17/vCK6qNtXZSO9Wyz.png)

### 查看owner

![image.png](https://i.loli.net/2020/11/17/l3n1y7jsDVTErox.png)

### 切换账号，并改变owner：

![image.png](https://i.loli.net/2020/11/17/rsKPeYbmcTH9Ixv.png)

### DASP top10

![image.png](https://i.loli.net/2020/11/17/EbSGXIRY4Og6ynK.png)

#### 整形溢出漏洞

- uint8 : 0-2^8-1
- uint256 : 0-2^256-1 

![image.png](https://i.loli.net/2020/11/17/KnEMJChBbwgF8oN.png)

![image.png](https://i.loli.net/2020/11/17/HoQ6lWXFT5ukD9j.png)

![image.png](https://i.loli.net/2020/11/17/Vl6MRWxXJs5uDdC.png)

![image.png](https://i.loli.net/2020/11/17/Enh3ZtFcADX9fGg.png)

#### 重入漏洞

![image.png](https://i.loli.net/2020/11/17/MDFtN29miCVeIJq.png)

![image.png](https://i.loli.net/2020/11/17/HwdRJ9TO1Lvolzj.png)