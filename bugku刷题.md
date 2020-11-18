---
title: bugku刷题
date: 2019-08-01 18:37:57
updated:   2019-08-02 13:00:58
tags:
    - 逆向
    - 刷题
    - bugku
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

# bugku-baby

# 1.运行程序

程序一闪而过

# 2.用od打开

1.查找字符串

2.找到“Hi~ this is a babyre”，跟踪进入

3.发现后面函数很像是flag的产生

4.先不管，单步运行，查看栈的变化

5.发现无flag类似字段的出现

# 3.用ida打开

1.找到主函数，f5反汇编

2.只有一个输出函数

3.看反汇编代码

4.注意到输出函数的下方很像是flag的产生（上面第三步）

5.尝试按r

6.成功得到flag

# bugkyu-easy_vb

# 1.运行程序

发现九个可选数字+确认按钮，猜测找到数字后确认给flag

# 2.用od打开

1.搜索字符串，直接得到flag（好简单）

# 3.尝试用ida打开

发现只有一个thunrtmain函数（加了壳？）

# bugku-Easy-re

# 1.运行程序

显示需要输入flag，随便输入，回车，得到提示flag不正确。猜测程序内有一个比较函数，比较真正的flag以及输入的flag

# 2.用od打开

1.查找字符串，找到主函数

2.稍微过一遍反汇编代码，设置断点

3.由于是比较，大部分是先把正确的flag压入栈中或保存在寄存器中，所以f8单步调试，注意这俩个地方的值

4.果然，得到了flag

# bugku-逆向入门

# 1.下载文件

运行，运行失败

# 2.在exeinfope中查看，不是pe文件？

# 3.用010editor打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563849004807-2efa1842-d8db-4cf9-9f7c-6da60aa4569f.png)

# 4.百度，得知正确解法

用记事本打开，把内容放在<img src=""/>里，改为html文件

# 5.打开html文件

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563849319371-ac51bfb6-3f28-4dfe-8605-47bae2ea444f.png)

扫描，得到flag（这是逆向？？？？）

# bugku-love

# 1.下载文件

查壳，无壳

# 2.ida打开

1.shift+f12，发现有base64input

2.找到主函数，f5

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563862185781-fdb88e1c-48b8-4044-be8b-4fd15760593f.png)

分析得知，输入的字符串经过加密后与其中的字符串进行相比：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563862237935-ded3adad-71d9-43dd-8c93-fd9d8e73e150.png)

则将原有字符串按加密方式解密，就能得到flag

查看加密函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563862337278-6ce3a665-18ee-4051-b653-69fc4b888082.png)

发现有/3 *4之类的运算，猜测为base64加密（百度得知base64加密算法）

并且发现：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563863308422-9484fe11-75c5-4fb9-8ff7-2a7934f7ee8b.png)
输入的字符串保存在数组dest中后每个还加了其对应数组位置的值

如此，大概明白了加密方式

# 3.编写相应脚本



```python
import base64
s='e3nifIH9b_C@n@dH'
flag=''
for i in range(len(s)):
    a=ord(s[i])-i
    flag+=chr(a)
print(str(base64.b64decode(flag),'utf-8'))
```

运行，得到flag

# bugku-不好用的ce

# 1.下载文件，查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564634381479-d41ce758-cbfd-4a08-a47f-bfef4f7e73dd.png)

无壳，且为vb编写

# 2.运行程序

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564634610065-fbe9835d-2045-4089-8569-de2dfa14ff87.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564634637176-1d5fd9ee-c17b-4a5f-b573-b9c7e1fade5e.png)

# 3.od载入

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564634670248-7714a2e2-6b00-42d7-825a-6b64ca0d9f3c.png)

查找字符串：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564634749909-6d0ecbd3-37bc-460f-a85e-bb9c782a3621.png)

# 4.下载vb反编译程序

反编译：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564634862650-fcf715e7-02a9-40fc-85f4-0cb8f0005c44.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564634873233-c8d1033f-856c-41f2-9be3-2d53b2f6db06.png)

找到第二个点击事件地址在401c80

# 5.od中找到，下断点运行

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564635030327-ce1fc02d-f646-4fe4-8927-29c0e0fdac83.png)

# 6.分析

发现下面有好几串字符串，其中一串下面调用了messagebox：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564635175445-594a16ae-e445-48f7-8d7e-904692f57084.png)

# 7.调试

找到疑似比较函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564635245165-5032bc56-9d73-4584-b143-0e2fcbefaa0e.png)

断点调试

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564635287638-1298f042-1f34-4ec0-aefa-32eb15667a99.png)

尝试修改st的值，并运行

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564635358189-237ae612-a082-41f0-b4c7-be53bf749bb4.png)

得到了字符串（果然在messagebox里）

# 8.猜测为base加密

尝试，得到flag：
![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564635467978-459d2c88-19f1-4d4d-8f43-79662b290597.png)

# BugkuCTF-Mountain climbing

# 1.下载，查壳

发现为upx的壳

# 2.脱壳（esp定律）

# 3.ida载入

f5分析：

首先长度要是19，且只能输入R或L

有一个数组，随机生成，因为是伪随机，种子一样跑出来一样，自己跑一遍

继续分析得知要让和最大，则从第一个数往下走，选左边或右边更大的数，记录下来

得到：RRRRRLLRRRLRLRRRLRL

结果错误

# 4.od调试

跟踪输入字符串，发现字符串被改变

# 5.两者结合分析

在ida中发现有一个函数对输入有改变，记住地址，在od中调试，发现算法，偶数位不变，奇数位异或4

# 6.得到flag

# bugku-sign_in

# 1.下载，在模拟器上运行

# ![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564653855524-5bf77c56-876f-4baf-98e3-a8b17d1b9e6f.png)

# 2.androidkiller加载

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564653996213-55ef8d14-c849-49d4-8779-27840030df00.png)

反编译源码

# ![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564654030667-564fb801-6c51-42e1-8a26-21f003edf436.png)3.分析

在网上搜索发现两个功能：

reverse()

​    功能：反转数组中的元素的顺序

​    语法：arrayobject.reverse.(）

tostring()

  功能：将各种进制的数字转化为字符串

  语法：number.toString(radix)（radix代表进制数）

# 4.观察

得到一个id：2131427360

在r文件中找

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564654507351-1fce1c15-cc08-4833-b780-f092be0742ff.png)

果然是tostring，接下来在string.xml里找（可以直接搜索toString）

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564654631625-4cccc53a-ffc3-49d2-b53a-e4068e3f4a00.png)

# 5.得到字符串

反向后解码，成功得到flag

# bugku-Timer(阿里CTF)

# 1.模拟器上运行

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564654905982-e1c6d873-64b6-40c9-a55e-48328a6a1706.png)

不断在倒计时

# 2.androidkiller加载

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564655022793-a90be12f-57e3-4065-975d-f012c533139b.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564655031159-afe76686-37f8-4ef4-bfcf-b2e72b8ed71b.png)

# 3.分析

输出flag里面，是一个k

而k随着时间不断在变

前方有一个判断，应该是时间的判断

思路：修改时间的判断，并且获得k的最终值（flag）

# 4.写脚本



```java
public class test{
    
    public static boolean is2(int arg4) {
        boolean v1 = true;
        if(arg4 > 3) {
            if(arg4 % 2 != 0 && arg4 % 3 != 0) {
                int v0 = 5;
                while(true) {
                    if(v0 * v0 <= arg4) {
                        if(arg4 % v0 != 0 && arg4 % (v0 + 2) != 0) {
                            v0 += 6;
                            continue;
                        }

                        return false;
                    }
                    else {
                        return v1;
                    }
                }   
            }

            v1 = false;
        }
        else if(arg4 <= 1) {
            v1 = false;
        }

        return v1;
    }
    
    public static void main(String args[])
    {
        int time = 200000;
        int k = 0;
        while(time > 0){
            if(is2(time)){
                k += 100;
            }
            else
                k--;
            
            time--;
        }
        
        System.out.println(k);
    }
}
--------------------- 
作者：Sea_Sand 
来源：CSDN 
原文：https://blog.csdn.net/zz_Caleb/article/details/92407254 
```

# 5.得到最终k值，在源码中修改两处值

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564655665008-a4462b6c-9b80-49ef-9e69-9d61b8dca70e.png)

改为

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564655684469-d096d4a5-3558-4e55-88e5-055bda73c294.png)

并加上const v3，1616384

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564655834778-027b1dfc-3fe1-4051-b7ed-c8f1901ba856.png)

# 6.重新打包并签名

重新安装，运行得到flag