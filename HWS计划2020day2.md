---
title: HWS计划2020day2
date:   2020-11-02 19:45:08
updated:  2020-11-02 19:45:08
tags:
    - 逆向
    - 混淆
    - 花指令
    - SMC
categories: 
	- HWS计划2020
use:
  - Valine
text: true
lazyload: true
count: true
---

# dump进程

使用scylla脱壳并修复IAT后依旧无法运行时，可能是重定位地址访问错误，需要修改pe文件的字段：

第一种，勾选nt头中第四项（Relocaation...）：
![image.png](https://i.loli.net/2020/11/17/8OJlyPqRUiMZhb4.png)

第二种，可选头dll第一项去掉（dll can move）：

![image.png](https://i.loli.net/2020/11/17/K3RmZepjtTuogdv.png)

# 反调试

![image.png](https://i.loli.net/2020/11/17/6wHrtzoPIgDXnub.png)

## 调用Windows API进行反调试

![image.png](https://i.loli.net/2020/11/17/JKfipDqW5zA6xvE.png)

![image.png](https://i.loli.net/2020/11/17/NRPbAjI9ak5Fe2V.png)

## 使用断点检测进行反调试

### 软件断点



![image.png](https://i.loli.net/2020/11/17/kgzDxcVdTlGZCOA.png)

![image.png](https://i.loli.net/2020/11/17/HQX13wcYUKhVZ8C.png)

### 硬件断点

![image.png](https://i.loli.net/2020/11/17/4Iwdi3q1fa2WbQP.png)

![image.png](https://i.loli.net/2020/11/17/yzCh2Ncpj6amKPf.png)

## 时间间隔检测

![image.png](https://i.loli.net/2020/11/17/lGI7q1JBHEuMnPt.png)

## 基于异常的反调试

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604308989610-29f02852-c2e8-4069-8226-6437b5a33374.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604309216503-1d464057-8f88-4886-a142-ce26edf01fca.png)

## 其它

![image.png](https://i.loli.net/2020/11/17/voCJhnGRXTmd6fa.png)

# SMC

![image.png](https://i.loli.net/2020/11/17/6eEB5KInfstDjdA.png)

例子：
查找字符串定位到关键anti函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604312026136-8a7c8370-4366-4489-9dd5-a44c10d0d0fd.png)

并发现此时运行程序，程序直接退出

猜测有反调试功能，根据字符串定位：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604312123027-0bdd9281-6832-4276-b715-32d92559cce7.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604312150275-4cd6fe8c-3603-4472-9ef0-42910eb1a661.png)

![image.png](https://i.loli.net/2020/11/17/dkhlstTXmrjxC1B.png)

反anti：直接修改为无条件跳转
![image.png](https://i.loli.net/2020/11/18/rPhtsEGc3HNTJ4M.png)

返回到主函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604312470627-3db863e4-3890-40a9-8e0d-9da0afd145ce.png)

发现这个函数并没有被执行

分析上个函数：
![image.png](https://i.loli.net/2020/11/17/wyfJ7p2vktsAgcN.png)

发现ida将字符串识别错误

修改正确：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604312706942-7fc36391-b706-4821-af54-3dfc1328f0f8.png)

![image.png](https://i.loli.net/2020/11/17/cX7xVonYMSkZDaI.png)

![image.png](https://i.loli.net/2020/11/17/sehfHiULE2F7BXl.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604312903248-d118f21d-d2ce-4698-b604-5463125bba30.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604312987744-e4bc01e8-4670-4698-b187-0c3b589cf679.png)

编写脚本

```python
magic='Rising_Hopper!'
magic=[ord(each) for each in magic]
v1=[0]*18
v1[0] = 17
v1[1] = 8
v1[2] = 6
v1[3] = 10
v1[4] = 15
v1[5] = 20
v1[6] = 42
v1[7] = 59
v1[8] = 47
v1[9] = 3
v1[10] = 47
v1[11] = 4
v1[12] = 16
v1[13] = 72
v1[14] = 62
v1[15] = 0
v1[16] = 7
v1[17] = 16
flag=[]
for i in range(18):
    for each in range(0x100):
        v18 = ~(each & magic[i % 14]) & (each | magic[i % 14]);
        if v18 == v1[i]:
            flag.append(each)
            break
print(bytearray(flag).decode())
```

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604313608592-c64359d8-ca7a-4314-b985-9028a3d430ee.png)

输入到源程序后：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604313791098-846118fd-06ed-4b0b-be16-d9dba0afa0cf.png)

后面的字符串并不能找到

观察函数，发现输入放在了bss段：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604313985563-d35798b9-1c4e-44eb-8249-4fd62ed8d618.png)

![image.png](https://i.loli.net/2020/11/17/QLsePwlxAHNF3X7.png)

并有多初引用

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604314074415-f3f275d9-cdbe-4d8b-8151-555c20c9c835.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604314100060-5c002130-9a5d-4c6c-a3f1-5cbb143429b7.png)

发现代码段中的未识别的数据：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604314155044-cdc140d4-e085-43d9-83a2-b61aad2e01e1.png)

分析解密函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604314289961-d7f640c1-eb78-4704-be8d-ce67e4575699.png)

先设置代码段可修改

然后进行异或操作，结束后恢复权限

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604314330298-1eb784cb-4313-4059-b384-9784b80c10b6.png)

查看第二个函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604314380012-05882a4e-694a-4091-ad67-a2cd62969668.png)![image.png](https://i.loli.net/2020/11/17/gLYvD86OBbqZc3d.png)

查看anti-od的函数

方法一：动态调试

下断点：

![image.png](https://i.loli.net/2020/11/17/q13gujKMXOEnzG6.png)

运行发现已解密:

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604314907786-51a502af-f2e9-4dc3-81ce-050f003bc940.png)

可以选择二进制复制到ida中分析

方法二：静态解密

idapython

```python
def smc(addr,length,key):
    for i in range(length):
        bt=Byte(addr+i)
        bt^=key[i%18]
        PatchByte(addr+i,bt)
    print('done!')
Str='Caucasus@s_ability'
Str=[ord(each) for each in Str]
smc(0x10040164D,1045,Str)
```

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604315355914-8759a1a3-c9e2-40b9-8a5c-3f37ddcc4700.png)

解密完成

按快捷键c：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604315431657-3bd79f93-4cb5-4c8d-9557-3ac610e363e9.png)

再按快捷键p变成函数

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604315496082-cd0d7fef-cb6b-494d-95e7-068245358792.png)

继续写脚本：

```python
dst=[0]*51
dst[0] = 2007666
dst[1] = 2125764
dst[2] = 1909251
dst[3] = 2027349
dst[4] = 2421009
dst[5] = 1653372
dst[6] = 2047032
dst[7] = 2184813
dst[8] = 2302911
dst[9] = 2263545
dst[10] = 1909251
dst[11] = 2165130
dst[12] = 1968300
dst[13] = 2243862
dst[14] = 2066715
dst[15] = 2322594
dst[16] = 1987983
dst[17] = 2243862
dst[18] = 1869885
dst[19] = 2066715
dst[20] = 2263545
dst[21] = 1869885
dst[22] = 964467
dst[23] = 944784
dst[24] = 944784
dst[25] = 944784
dst[26] = 728271
dst[27] = 1869885
dst[28] = 2263545
dst[29] = 2283228
dst[30] = 2243862
dst[31] = 2184813
dst[32] = 2165130
dst[33] = 2027349
dst[34] = 1987983
dst[35] = 2243862
dst[36] = 1869885
dst[37] = 2283228
dst[38] = 2047032
dst[39] = 1909251
dst[40] = 2165130
dst[41] = 1869885
dst[42] = 2401326
dst[43] = 1987983
dst[44] = 2243862
dst[45] = 2184813
dst[46] = 885735
dst[47] = 2184813
dst[48] = 2165130
dst[49] = 1987983
dst[50] = 2460375

flag=[]
for i in range(51):
    for each in range(0x100):
        v18 = 19683*each%0x8000000B
        if v18 == dst[i]:
            flag.append(each)
            break
print(bytearray(flag).decode())
```

成功得到flag

![image.png](https://i.loli.net/2020/11/17/dClpOZuhGIHk6oY.png)

# 花指令（混淆）

![image.png](https://i.loli.net/2020/11/17/8tZyrvoqOchFSGi.png)

![image.png](https://i.loli.net/2020/11/17/LwXmS9R6geWntYi.png)

![image.png](https://i.loli.net/2020/11/17/TsNApCPrMc12Yq8.png)

![image.png](https://i.loli.net/2020/11/17/49ElUpbtuLN2O3K.png)

![image.png](https://i.loli.net/2020/11/17/9FvKRiSlDk4wcMj.png)