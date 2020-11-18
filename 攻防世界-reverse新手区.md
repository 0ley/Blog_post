---
title: 攻防世界-reverse新手区
date: 2019-08-01 12:59:14
updated: 2019-08-02 12:55:05
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

# 攻防世界-maze

# 1.下载文件

**发现是64位elf文件，题目提示为走迷宫**

# 2.在ida中打开

**找到主函数，发现比较复杂，f5：**

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563844787299-e89b3205-3913-4e76-85bd-9e8df1bd8dc6.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563844797689-ae4d2107-6125-42d0-a11d-a8dac4fd7eb1.png)

**一点点分析**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563844849362-171c89de-ab4a-4b2e-80e4-d06f45665dec.png)**

**v10为0，s1是输入的字符串，与flag进行比较，可知flag长度为24，前面为ntcf{，最后为}**

**（别忘了用R！！！！）**

**在下面发现四个函数：**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845047410-5df9006d-7a81-4ed4-b4b5-9dbf0c98fe0e.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845065204-322f157a-4973-4696-9a6a-dd16563586ae.png)**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845086067-f908208b-89c5-4d09-9693-3898547a9fad.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845108116-7322d901-8d31-46b6-8b6b-04ed13bd914c.png)**

**分别对应字符：‘O’‘o’‘.’‘0’**

**最后有一比较函数，且走迷宫要走到‘#’结束**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845237591-9a636802-7ac3-42ca-a8d6-77c957da53e7.png)**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845261370-7edbf81d-27db-4540-8268-d12b27814e6f.png)**

**查看迷宫，发现是8列为一行的迷宫**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845900858-179908ec-f638-4141-85ed-ba2023667bc0.png)**

**
**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845910981-88dda2ca-90f0-4bef-8d31-0cd8fa7e1b8f.png)**

**画出迷宫：**

**![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563845978743-e25d789d-5320-4c07-8398-33755cd43c7a.png)**

**分析上面四个函数，知道四个字符分别代表上下左右**

**最后，把走到迷宫的步骤转化为字符，得到flag**

# 攻防世界-no-strings-attached

# 1.下载文件，在ida内打开

找到主函数，f5：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563777220318-9e46f4e4-7907-4b03-bb02-dee14ebf17ec.png)

发现四个函数，逐个进入分析最后发现authenticate（鉴定）最重要：

# 2.函数authenticate

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563777333721-9be5a179-654f-401e-b5c3-b5b48919e131.png)

其中又存在一个decrypt解密函数，分析得知s2为flag

# 3.动态调试（ida远程）

百度ida如何动态调试，得到远程调试方法：

## 1.将所选复制到linux下

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563777483870-3cbd630a-6317-45ae-833e-bf424f2677fd.png)

## 2.在Linux中运行

## 3.ida打开，调试器选择Linux的那个，并按要求配置：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563776722128-a03f76c8-640c-4d4e-9bd2-8158d3aad3bb.png)

## 4.开始动态调试，找到主函数，设断点，运行

## 5.分析汇编代码，发现生成的flag保存在了eax中，则在dump窗口跟随eax，并逐步运行：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563776867781-01fda01b-68ae-40e5-8fdf-8e0cf0ab061a.png)

成功得到flag

## 注意：仔细看eax的值，是从9开始，而不是1！！！

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563777052394-900f4fbb-51ef-4a2e-ab1d-f143bac355d8.png)

# 4.gdb调试

百度后发现第二种动态调试方法，使用linux下的gdb调试：

## 1.打开gdb,调试文件：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563780334903-bf71b5b7-0fca-4dd5-9f8a-3197ad559d9d.png)

## 2.设置断点到authenticate

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563780402008-477ed261-78dc-4cad-904d-cb77b7aa502e.png)

## 3.r

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563780607013-cf9d010c-dfa9-4884-948c-2ecea906a3b5.png)

## 4.n运行至08048725

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563780710715-b1e7fc34-c2e2-40bc-a507-5704a2b09f67.png)

## 5.执行命令 x/50x $eax(or x/200c $eax)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563780803778-e444a6b5-9bdb-4798-8152-a30d409799c4.png)



![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563780883796-16181c7d-ea2d-4a24-87d8-03c336b44667.png)

## 6.编写相应脚本

```python
a=[0x39,0x34,0x34,0x37,0x7b,0x79,0x6f,0x75,0x5f,0x61,0x72,0x65,0x5f,0x61,0x6e,0x5f,0x69,0x6e,0x74,0x65,0x72,0x6e,0x61,0x74,0x69,0x6f,0x6e,0x61,0x6c,0x5f,0x6d,0x79,0x73,0x74,0x65,0x72,0x79]
c=''
for x in a:
    c+=chr(x)
print(c)
```

得到flag

## 7.指令：

### n是next，b是breakpoint s是step， n是单步不进入，s是单步进入

### 用gdb查看内存：

#### 格式: x /nfu  

说明：

x 是 examine 的缩写

n表示要显示的内存单元的个数

f表示显示方式, 可取如下值

x 按十六进制格式显示变量。

d 按十进制格式显示变量。

u 按十进制格式显示无符号整型。

o 按八进制格式显示变量。

t 按二进制格式显示变量。

a 按十六进制格式显示变量。

i 指令地址格式

c 按字符格式显示变量。

f 按浮点数格式显示变量。

u表示一个地址单元的长度

b表示单字节，

h表示双字节，

w表示四字节，

g表示八字节

# 攻防世界-csaw2013reversing2

# 1.下载文件，并运行

跳出弹窗，且为乱码

# 2.用od打开

字符串搜索，找到主函数，观察发现：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563789158049-9bc759e2-ac9f-450b-8093-1a04d7b8db76.png)

啥也不想，先把int3 nop掉，另外把je改成jne（最常见的暴力解法）

在此设断点，运行

发现在00b31000处有两个很像加解密的循环，继续调试

密切注意调试这两段循环时寄存器，dump的变化（可以一次调试就换一个寄存器在dump中跟随）

发现在第二次循环快结束时，得到了flag：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563790443900-727faeaf-1e83-4e16-8548-db964befd618.png)

# 3.用ida打开：

## 1.找到主函数，f5：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563790581460-8a667677-b596-419b-890d-c39bb8171b19.png)

一个个函数点进去观查后发现sub_401000函数时加解密函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563790645540-637d212c-fe65-4276-adb1-b406ef7042f1.png)

## 2.看主视图

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563790673208-97b27dfc-45d3-40b9-ae2f-394320ea4091.png)

发现加解密函数无法得到运行，先将int 3这个中断指令nop掉

## 3.分析视图

猜测loc_4010b9为输出，则将加解密后的跳转和最开始的跳转都修改一下

修改后：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563790881770-a73a940b-3188-4906-add9-026621736c96.png)

保存修改，运行程序：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563791217818-b6f1b045-7d59-429a-9932-2a5221349624.png)

得到flag

# 攻防世界-getit

# 1.下载文件

发现是elf文件

# 2.ida打开

找到主函数，f5：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563771907858-38c9270e-fcdf-4e29-8eb7-a5b5a856494a.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563771925208-4ee17330-5152-497b-a587-63413c2e1f58.png)



# 3.分析，并编写相应脚本：



```python
v5=0
s='c61b68366edeb7bdce3c6820314b7498'
v3=-1
for i in s:
    print(chr(ord(i)+v3),end='')
    v3=-v3
```

得到flag

ps.不要忘记：

前缀’S'

# 攻防世界-logmein

# 1.下载文件

依旧是elf文件，这次没有加壳

# 2.ida打开

1.找到主函数，shift+f12，无明显flag

2.f5，得到伪代码如下：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563263103235-dd3da317-efe0-4d8d-919c-15400c485ecb.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563263126930-c47e5d46-a3eb-462a-b59c-9663dabfb40d.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563263142259-459d736e-deef-4c19-be41-089cd1aaed32.png)

3.分析伪代码，得到flag的算法

4.根据伪代码写代码：



```c
#include<stdio.h>
int main(void)
{
    char a[] = {":\"AL_RT^L*.?+6/46" };
    long long int b = 28537194573619560;
    char * p = (char*)&b;
    for (int i = 0;a[i] != 0;i++) 
    { a[i] = a[i] ^ p[i % 7]; }
    printf("%s\n", a);
    return 0;
}
```

5.得到flag

# 攻防世界-insanity

# 1.下载文件

发现又是elf文件

# 2.用ida打开

找到主函数

# 3.shift+f12

得到flag（记得用啊）

# 攻防世界-simple-unpack

# 1.下载

发现是个txt文件，用010editor打开，如下：

# ![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563260490643-4bf5ceee-b04d-452c-bd39-3961040e786b.png)

发现是elf，linux下的文件，且用upx加壳

# 2.百度如何去upx的壳

找到upx自带去壳功能

# 3.在官网下载upx,并运行

发现要在控制台运行，根据网上教程，输入如下指令：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563260656674-2ff2876e-e341-4c79-bff3-2c3ff06fc633.png)

去壳成功

# 4.用ida打开去壳文件

找到主函数，发现flag

# 5.总结

1.知道了怎么看upx的壳（软件检查+010editor)

2.学会了怎么去upx的壳（软件+指令）

# 攻防世界-open-source

# 1.下载文件，发现是源码

# 2.打开源码，如下：



```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    if (argc != 4) {
        printf("what?\n");
        exit(1);
    }

    unsigned int first = atoi(argv[1]);
    if (first != 0xcafe) {
        printf("you are wrong, sorry.\n");
        exit(2);
    }

    unsigned int second = atoi(argv[2]);
    if (second % 5 == 3 || second % 17 != 8) {
        printf("ha, you won't get it!\n");
        exit(3);
    }

    if (strcmp("h4cky0u", argv[3])) {
        printf("so close, dude!\n");
        exit(4);
    }

    printf("Brr wrrr grr\n");

    unsigned int hash = first * 31337 + (second % 17) * 11 + strlen(argv[3]) - 1615810207;

    printf("Get your key: ");
    printf("%x\n", hash);
    return 0;
}
```

# 3.分析源码

1.得到三个参数：“51966”，“25”，"h4cky0u"

2.将代码改为：



```c
#include<stdio.h>
#include<string.h>
int main(void)
{
    unsigned int hash = 51966 * 31337 + (25 % 17) * 11 + strlen("h4cky0u") - 1615810207;
    printf("Get your key: ");
    printf("%x\n", hash);
}
```

3.成功得到flag

# 攻防世界-hello,CTF

# 1.运行程序

看到please input your serial:，随便输入，回车，得到“wrong”

# 2.用od打开

1.搜索字符串，进入主函数

2.在主函数中发现注释（437261636b4d654a757374466f7246756e）

3.联想到题目描述：菜鸡发现Flag似乎并不一定是明文比较的

4.得出这是flag的结论

5.编写程序：

```c
#include <stdio.h>
int main()
{
char a[] = { 0x43, 0x72, 0x61, 0x63, 0x6B, 0x4D, 0x65, 0x4A, 0x75, 0x73, 0x74, 0x46, 0x6F, 0x72, 0x46, 0x75, 0x6E };
for (int i = 0; i < sizeof(a); i++)
{
    printf("%c", a[i]);
}
return 0;
}
```

6.得到flag

# 攻防世界-re1

## 1.打开程序：

显示需要输入flag

## 2.用od打开：

1.查看字符串，跟踪“%s”,设置断点，f8句句调试，注意栈中内容，并未发现flag相关的字符串。

2.在dump窗口跟踪相关地址，发现flag。

# 攻防世界-game（bugku-游戏过关）

# 1.运行程序：

发现是个开关灯的小游戏，依次从1-8输入，通关游戏，得到flag

# 2.用工具

1.假装没得到flag，用Od打开

2.字符串查找

3.看到“done!!! the flag is ”

4.下面是一堆操作

5.分析无果，用ida打开

6.跟进主函数，f5，跳转再跳转

7.分析代码，创建了两个字符串并进行运算，得到最后flag

8.编写脚本（c)



```c
#include<stdio.h>
int main(void)
{
    char s1[] = { 123, 32, 18, 98, 119, 108, 65, 41, 124, 80, 125, 38, 124, 111, 74, 49, 83, 108, 94, 108, 84, 6, 96, 83, 44, 121, 104, 110, 32, 95, 117, 101, 99, 123, 127, 119, 96, 48, 107, 71, 92, 29, 81, 107, 90, 85, 64, 12, 43, 76, 86, 12, 114, 1, 117, 126, 0 };
    char s2[]= { 18, 64, 98, 5, 2, 4, 6, 3, 6, 48, 49, 65, 32, 12, 48, 65, 31, 78, 62, 32, 49, 32, 1, 57, 96, 3, 21, 9, 4, 62, 3, 5, 4, 1, 2, 3, 44, 65, 78, 32, 16, 97, 54, 16, 44, 52, 32, 64, 89, 45, 32, 65, 15, 34, 18, 16, 0 };
    int i;
    char a[60];
    for (i = 0;i < 56;i++)
    {
        a[i]= s1[i] ^ s2[i] ^ 0x13;
        printf("%c", a[i]);
    }
    return 0;
}
```



10.产生新想法：能否直接改变跳转地址，不经过判断，直接输出flag.

11.在od中更改跳转地址，调试，最终只显示“done!!! the flag is ”，并未输出flag.

12.再次尝试，将跳转指令提前几位，调试，果然不管输入多少，最终都能得到flag

# 3.总结

1.od+ida的结合；

2.ida的使用

3.python脚本的编写

4.新想法的实践

# 攻防世界-python-trade

# 1.下载文件

发现是Py.pyc，百度，找到一个在线python反编译工具

# 2.反编译

结果如下：



```python
import base64

def encode(message):
    s = ''
    for i in message:
        x = ord(i) ^ 32
        x = x + 16
        s += chr(x)
    
    return base64.b64encode(s)

correct = 'XlNkVmtUI1MgXWBZXCFeKY+AaXNt'
flag = ''
print 'Input flag:'
flag = raw_input()
if encode(flag) == correct:
    print 'correct'
else:
    print 'wrong'
```

# 3.分析代码

百度得知base64解码编码的概念，例：

```python
s = '我是字符串'
>>> a = base64.b64encode(s)
>>> print a
ztLKx9fWt/u0rg==
>>> print base64.b64decode(a)
'我是字符串'
```

# 4.编写相应脚本



```python
import base64
 s=base64.b64decode('XlNkVmtUI1MgXWBZXCFeKY+AaXNt')
 flag =''
 for i in s:
     i-=16
     i^=32
     flag+=chr(i)
 print(flag)
```

得到flag