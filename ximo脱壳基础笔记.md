---
title: ximo脱壳基础笔记
date: 2019-07-24 13:48:08
updated:  2019-07-29 18:00:07
tags:
    - 逆向
    - 脱壳
categories: 
	- 逆向工程
use:
  - Valine
text: true
lazyload: true
count: true
---

# 不同语言编译的程序以及常见壳入口点特征

# 一.无壳程序（注意常见api）

## 1.vc6

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563935743475-e459df89-5330-45bf-bedc-7db48305c37d.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563935770184-8b882e73-8145-4039-9832-1963d7afe952.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563935791889-50a9736c-cf0d-430d-b782-cb65d06727c8.png)

**栈帧+类似入口特征+相同api**

## 2.vs2008/2013

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563935857614-64be4458-2b7b-4671-875a-ad5a6e1d2ca4.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563935873360-3fb98efb-ce44-43f7-8bfd-25ae497386c1.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563935894041-c26a849b-e503-45c2-a385-4f9d37c1fe99.png)

**call+jmp**

## 3.易语言非独立编译

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563936082685-8b4e20c3-3129-4bb4-b5ce-c1347d7ea837.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563936095409-13a941a8-659e-482e-b12a-a3563b7e6891.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563936105253-f7abb8d2-335f-46a3-a7cd-d7a45208634a.png)

先加载运营库dll

## 4.易语言独立编译

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945099332-a0edefdb-a3e4-4088-9b7a-3eb3f65a6dbd.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945121563-3118bbfd-b86a-4eab-bbd5-6c4537856739.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945151914-86c6b419-81d2-4362-a49b-ed5cca99e971.png)

注意和vc的区别：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945243754-79eb68cd-a207-4b2c-a2cd-e99815da3517.png)

## 5.delphi

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945359901-c07368bc-c256-4e3b-a176-ec6731b20b54.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945375535-0e67ce67-df97-4bf5-a9c6-ab4152247e3c.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945408857-c3b04b79-0349-44ae-93fc-5e74608542b8.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945424115-9664ddd1-196d-4d00-819f-ad1e6709849e.png)

## 6.bc6++/bc2010++

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945460763-663cb3c8-8d13-4e40-907b-716a6bded2ec.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945477957-b1b29d0b-e5e9-4183-aa6e-2ce3da721e3b.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945500954-5c5dd9a0-330a-4245-9e6b-2950f44cc4e3.png)

## 7.asm

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945562961-7925ed2d-d44f-4686-9a54-8fee028fb7bf.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945576119-5c40b5b0-7beb-40bb-8903-dc241fefabf4.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945589059-77f9f8a5-1ec6-4e4d-9b48-5d32fcadde37.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945614919-c3db1bf2-02f4-4118-9012-166389aa0fca.png)

## 8.net

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945681788-5be2ceb9-093d-4fe1-9cf1-c5780e950a7c.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945692792-67303932-2b33-47b4-8109-39283ea95af1.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563945720400-f23b5fdb-c86c-4a54-8ebf-cacc3bcfa6b6.png)

一打开就自动运行

# 二.加壳程序

## 1.aspack

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563946880209-f8ad2cea-785a-489b-a500-de20ef5dd006.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563946890735-7ff6502c-4c72-47c6-a8e7-f97ae66d7ce9.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563946927932-0dd00f10-f6f4-40d1-a7c5-58027c2d4a15.png)

## 2.upx

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563946953145-63058aad-6d75-4857-9d36-1290078c4dfe.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563946964106-73430039-b4c5-4b29-b347-311fa8f502d4.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563946988669-06037d17-868e-4f38-8582-4b214254049e.png)

## 3.themida

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947013347-bd95ca6f-ac7f-4c11-ab5c-7f257dd1c2d4.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947024250-ee7ca6a0-acb6-4a4b-a3ed-98a15f92ac24.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947042431-7b1aa135-e67b-4c4e-9ec6-9a1aff0b414a.png)

## 4.vmprotect

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947096550-64a3713b-0da8-4419-ab93-4bc5541af207.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947130549-a7c9819e-b179-49a7-bb6a-a095aa3cc20e.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947169487-dc180169-752d-4e3a-aec7-e2f9324a5992.png)

## 5.shielden

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947205683-0e78e88f-46b0-41a0-ba62-437eedaead1f.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947235912-fe4e0954-e53d-4b30-a2cf-9e8337eac643.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947250053-d988756f-7e1f-4fa5-9ad5-255a6c0e49a2.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947269812-ea05edf0-74f2-4778-a0b5-754928a102df.png)![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563947283279-5597fc53-32eb-4bf1-8c04-cef80264241f.png)

# 手脱UPX壳

# 一.四种方法

## 方法1：单步跟踪

只向下跳转

## 方法2：ESP定律法

## 方法3：2次内存镜像法

查找内存（m）+在.rsrc下断点，运行；再查找内存+在upx0下断点，运行

## 方法4：一步直达法

ctrl+f（右键+查找+命令），不勾选整个断块

# 二.脱壳

## 1.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563952642881-b3d07052-aaa7-444c-96a0-72d2c99e13c6.png)

## 2.od打开，找到oep

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563952730411-e5bebfc6-b7b7-4f32-9829-58b5490c7304.png)

## 3.od脱壳方式一

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563952794842-69172ab9-aef4-456c-a165-3d845bd1241b.png)

运行：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563956340180-7a1a3e39-8804-4983-8d30-83f522d9321b.png)

使用ImportREC，修复后

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563956380609-b6cac8a9-6d82-4a7b-bba1-77fe9a594ecc.png)

成功

## 4.od脱壳方式二

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563952839361-38496b50-debe-4f61-bcdf-6682276df5a4.png)

如上

# 手脱ASPACK壳

# 一.6种方法

方法1：单步跟踪

方法2：ESP定律

方法3：一步直达

方法4：2次内存镜像

方法5：模拟跟踪

tc eip<xxxxxx（包含sfx，imports的地址行）

方法6：SFX（alt+o）

# 二.脱壳

## 1.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563961651158-d7705c76-14ce-4ebe-b68f-172db93cd894.png)

## 2.脱壳

用方法6找到oep

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563961830575-df372288-32a5-47d2-ba21-bfc0e3b335c2.png)

脱壳，运行成功，查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1563961908201-4807a105-8146-4d69-9c4f-0c9c09eb97d9.png)

# 手脱Nspack（北斗）壳

# 一.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017110405-3568045a-a323-4f3f-a9d4-0684ce3460f0.png)

# 二.脱壳

## 1.od打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017265170-35e9b60e-773f-4707-8161-5928db71411f.png)

## 2.找到入口点

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017341784-398fbf17-83b1-4fdf-847d-620c9a3a503d.png)

## 3.脱壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017395555-4db503e0-bd2b-49b2-8acb-18f318c80eff.png)

## 4.查看2.4版本的壳（3.7也一样）

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017430169-08a56f7c-9788-4297-ab67-c31d54ae4f68.png)

## 5.od打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017457403-13fd9d7c-2475-4ebb-856d-10fcc5f4910f.png)

## 6.跟踪jmp

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017483908-14871b53-d883-4424-827f-dd02a910c978.png)

## 7.同上，脱壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564017555185-d7297e22-87d5-4ed5-90bd-b44758894e4a.png)

# 手脱FSG壳

# 1.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564019042886-568fe2e6-69a2-4f3d-9a5e-e4036c8ae7c6.png)

# 2.脱壳

正常脱壳，修复，发现无法运行；

需要手动查找iat

# 3.修复

找到一个call，命令行输入：

d call的地址

在dump中向上找，找到全为0为之，记下地址：00425000  77DA6BF0  ADVAPI32.RegCloseKey

在dump中向上找，找到全为0为之，记下地址：00425280  7C838DE8  kernel32.LCMapStringA

在修复工具里：

oep不变，rva为25000，大小为80（或1000，方便）

# 手脱PECompact2.X的壳

# 一.查壳

# ![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564192332689-211a5bb7-624b-44bf-9125-6c29e24e5e64.png)

# 二.脱壳

## 1.单步

## 2.ESP定律

## 3.BP VirtualFree

SHIFT+F9，取消断点

ALT+F9(执行到用户代码）

查找 push 8000(特征码)

运行到这

单步跟

## 4.BP VirtualFree

两次SHIFT+F9

中断后取消断点，Alt+F9返回

单步走。

## 5.

0040A86D >  B8 74DE4500   mov eax,qqspirit.0045DE74(原因）

bp 0045de74（设断点）

045DE74   B8 F9CB45F0   mov eax,F045CBF9

0045DE79   8D88 9E120010  lea ecx,dword ptr ds:[eax+1000129E]

0045DE7F   8941 01     mov dword ptr ds:[ecx+1],eax

0045DE82   8B5424 04    mov edx,dword ptr ss:[esp+4]

0045DE86   8B52 0C     mov edx,dword ptr ds:[edx+C]

0045DE89   C602 E9     mov byte ptr ds:[edx],0E9

0045DE8C   83C2 05     add edx,5

0045DE8F   2BCA       sub ecx,edx

0045DE91   894A FC     mov dword ptr ds:[edx-4],ecx

0045DE94   33C0       xor eax,eax

0045DE96   C3        retn

0045DE97   B8 78563412   mov eax,12345678//下断

## 6.bp VirtualAlloc  

SHIFT+F9运行

取消断点

ALT+F9

向下拉，看到JMP。运行到这

## 7.最后一次异常法

取消所有异常。

2次跑飞。

找SE句柄

转到SE xxxx处

045DE74   B8 F9CB45F0   mov eax,F045CBF9

0045DE79   8D88 9E120010  lea ecx,dword ptr ds:[eax+1000129E]

0045DE7F   8941 01     mov dword ptr ds:[ecx+1],eax

0045DE82   8B5424 04    mov edx,dword ptr ss:[esp+4]

0045DE86   8B52 0C     mov edx,dword ptr ds:[edx+C]

0045DE89   C602 E9     mov byte ptr ds:[edx],0E9

0045DE8C   83C2 05     add edx,5

0045DE8F   2BCA       sub ecx,edx

0045DE91   894A FC     mov dword ptr ds:[edx-4],ecx

0045DE94   33C0       xor eax,eax

0045DE96   C3        retn

0045DE97   B8 78563412   mov eax,12345678//下断

## 8.两次内存

## 9.at GetVersion(c++）

# 手脱EZIP壳

# 一.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564206394497-edff0cb5-8b1c-47c9-9b47-d68dad44771a.png)

# 二.脱壳

## 1.od载入

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564206413846-8a38bb4e-4387-442d-b105-5b74fac6addb.png)

## 2.esp定律

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564206465860-5426df0f-3d5b-4627-a6eb-0e1631b8649c.png)

## 3.脱壳+修复

成功

## ****可以考虑重建pe功能****

# 手脱TELock0.98b1壳

# 一.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564207264173-3cb59c14-0400-4619-b446-f05bfcf06ef3.png)

# 二.脱壳

## 1.最后一次异常法（插件，异常计数器）

载入od：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564207329716-58e09cac-a758-4f2f-92dd-d63124e7fe7a.png)

取消异常：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564207362452-ffac5f8e-cedf-4b9c-8cb6-54bd1082a02e.png)

shift+f9一直到程序运行，记录次数，重来，在运行前停止，在栈里找到se，跳转，设断点，运行，找到oep

## 2.模拟跟踪

## 3.两次内存镜像

# 三.修复

等级3跟踪

# 手脱EXE32PACK壳

# 一.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564209426483-d7e994f2-0f2b-48f4-ac34-790c2c0f4f5c.png)

# 二.脱壳

## 1.esp定律

## 2.巧妙

下断：BP IsDebuggerPresent  

   运行，取消断点

   ALT+F9

   计算ss+edi

   转到OEP！

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564209617071-ad250615-f0fe-41d9-8924-6dc73b4ddd6f.png)

# 三.修复

修复后在lordpe里重建pe

# 脱WinUpack加的壳

正常脱壳，时间较长

尝试加条件断点的方法

在跳转未实现处右键，断点，条件，写上满足条件的指令

运行，成功

# 脱壳基本的思路及小结

# 脱壳的基本方法：



## 1.单步

##  

## 2.ESP定律

##  

## 3.内存镜像

##  

## 4.模拟跟踪（2类）

### 1）SFX跟踪

###  

### 2）tc eip<XXXX



## 5.最后一次异常

##  

## 6.特殊



# 常见语言的入口点：



## VB：



```
004012D4 >  68 54474000     push QQ个性网.00404754
004012D9    E8 F0FFFFFF     call <jmp.&MSVBVM60.#100>
004012DE    0000            add byte ptr ds:[eax],al
004012E0    0000            add byte ptr ds:[eax],al
004012E2    0000            add byte ptr ds:[eax],al
004012E4    3000            xor byte ptr ds:[eax],al
004012E6    0000            add byte ptr ds:[eax],al
004012E8    48              dec eax
```



## delphi:



```
004A5C54 >  55              push ebp
004A5C55    8BEC            mov ebp,esp
004A5C57    83C4 F0         add esp,-10
004A5C5A    B8 EC594A00     mov eax,openpro.004A59EC
```



## BC++:



```
00401678 > /EB 10           jmp short btengine.0040168A
0040167A   |66:623A         bound di,dword ptr ds:[edx]
0040167D   |43              inc ebx
0040167E   |2B2B            sub ebp,dword ptr ds:[ebx]
00401680   |48              dec eax
00401681   |4F              dec edi
00401682   |4F              dec edi
00401683   |4B              dec ebx
00401684   |90              nop
00401685  -|E9 98005400     jmp 00941722
0040168A   \A1 8B005400     mov eax,dword ptr ds:[54008B]
0040168F    C1E0 02         shl eax,2
00401692    A3 8F005400     mov dword ptr ds:[54008F],eax
00401697    52              push edx
00401698    6A 00           push 0
0040169A    E8 99D01300     call <jmp.&KERNEL32.GetModuleHandleA>
0040169F    8BD0            mov edx,eax
```



## VC++:



```
0040A41E >  55              push ebp
0040A41F    8BEC            mov ebp,esp
0040A421    6A FF           push -1
0040A423    68 C8CB4000     push 跑跑排行.0040CBC8
0040A428    68 A4A54000     push <jmp.&MSVCRT._except_handler3>
0040A42D    64:A1 00000000  mov eax,dword ptr fs:[0]
0040A433    50              push eax
0040A434    64:8925 0000000>mov dword ptr fs:[0],esp
0040A43B    83EC 68         sub esp,68
0040A43E    53              push ebx
0040A43F    56              push esi
0040A440    57              push edi
```



## MASM(汇编):



```
004035C9 >  6A 00           push 0
004035CB    E8 A20A0000     call <jmp.&kernel32.GetModuleHandleA>
004035D0    A3 5B704000     mov dword ptr ds:[40705B],eax
004035D5    68 80000000     push 80
004035DA    68 2C754000     push 11.0040752C
004035DF    FF35 5B704000   push dword ptr ds:[40705B]
004035E5    E8 820A0000     call <jmp.&kernel32.GetModuleFileNameA>
004035EA    E8 87070000     call 11.00403D76
004035EF    6A 00           push 0
004035F1    68 0B364000     push 11.0040360B
004035F6    6A 00           push 0
004035F8    6A 64           push 64
004035FA    FF35 5B704000   push dword ptr ds:[40705B]
```

# 附加数据的处理方法

# 1.查壳

# ![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564213711560-79a3751f-5a40-40ea-885a-02038f4300da.png)

# 2.脱壳

无法运行

# 3.猜测有附加数据

# 4.用overlay

成功运行

# 5.手动

## 1）手动找后复制

## 2）看pe最后一个节段真实的两个值相加，寻找，复制

# 自校验的去除方法

# 1.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564219911982-a2e318e9-e47f-46d1-9f6d-eb3f84e289c5.png)

# 2.脱壳后

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564219938540-a3eab741-69c6-4111-a756-704f56e0fee1.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564219952766-b0ab7406-0c0f-44a9-a458-fc6759ab2cb1.png)

# 3.去除自校验

1）把去壳的再在od中打开

2）两边下断点 bp CreateFileA

3）比较两边跳转哪边不一样

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564220244563-e616d2d0-f148-4a18-9cef-b093d38d1b89.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564220255538-54324201-8a2a-4acd-ba96-b2ad46f84d7a.png)

4）修改

# 4.运行

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564220308343-12c0cea9-6a65-4886-b447-730fa787e38b.png)

成功去除自校验

# 手脱皇后壳

# 1.查壳

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363400168-1482dd50-6fba-430d-9652-f9b266c57a42.png)

注意到setion名称-皇后

# 2.od打开

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363447401-c251d7ac-4025-44e4-a112-2267c0e88f03.png)

# 3.单步跟

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363475800-cb0a9140-a670-45cb-b112-24ef4066d097.png)

# 4.esp定律

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363517310-12c533d5-c74f-4f85-a985-0f68ecc3afe1.png)

# 5.单步调试

发现在004018635下断点后执行会运行程序

分析，代码在来回jmp

# 6.后一个jmp下断点

f9，发现57次程序运行

则56次停止，单步来到：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363726735-5c6ec90c-13f3-48ce-bac4-91691036e0c5.png)

# 7.单步

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363747177-401ddae9-1394-4608-8bc5-6591fe5794fe.png)

发现一大段nop，继续单步，来到：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363786632-40361c9c-3fef-4cf5-8a68-13955fa022b7.png)

# 8.esp定律

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363827156-18b84704-93dc-4069-a40f-34cbf0ebfe77.png)

来到esp

# 9.脱壳+修复

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1564363878582-2d8a5840-28c6-4b18-a487-bcbd83e6b9ff.png)

# 学会用脚本进行脱壳

***插件-odbgscrpt-运行脚本***

# 脱ACProtect132(无Stolen Code)

# 1.设置异常，隐藏OD

不勾选内存访问异常+strongod里skip some exception（可选）+隐藏od插件

# 2.SE处下内存访问断点

下内存访问断点

# 3.SHIFT+F9，F2，再一次SHIFT+F9，下断，再一次SHIFT+F9

# 4.取消所有断点

记住是所有

# 5.内存，00401000。F2，SHIFT+F9

# 6.直达OEP！！

# 7.修复，脱壳成功

# 脱ACProtect(存在Stolen Code)

# 1.常规方法，直达OEP

发现不像正常oep特征，猜测有偷代码

# 2.在之前的一个retn处

调试---设置条件---push ebp ---跟踪步入 ctrl+f11

# 3.找到被偷取代码



```
004254C9    55              push ebp
004254CA    8BEC            mov ebp,esp
004254CC    83EC 44         sub esp,44
```

二进制复制

# 4.重新到oep

往上数相同字节，粘贴，新建eip

修复被偷取的代码

# 5.脱壳

重建输入表不勾选

# 6.修复

用等级三跟踪（新开程序）

# ACProtect之寻找丢失的Stolen Code

# 1.设置异常（忽略除INT3中断的所有异常），隐藏OD

# 2.来到入口点，F9运行



```
004AC000 >  60              pushad   //记住这个
004AC001    4E              dec esi
004AC002    D3D6            rcl esi,cl
004AC004    4E              dec esi
004AC005    66:D3E8         shr ax,cl
004AC008    4E              dec esi
004AC009    8BC3            mov eax,ebx
004AC00B    48              dec eax
```



# 3.程序中断在这里



```
004BAB23    90              nop   //这其实也是最后一次异常
004BAB24    64:67:8F06 0000 pop dword ptr fs:[0]
004BAB2A    83C4 04         add esp,4
004BAB2D    60              pushad
004BAB2E    E8 00000000     call NetClean.004BAB33
004BAB33    5E              pop esi
004BAB34    83EE 06         sub esi,6
004BAB37    B9 5B000000     mov ecx,5B
004BAB3C    29CE            sub esi,ecx
004BAB3E    BA 09E5B87E     mov edx,7EB8E509
004BAB43    C1E9 02         shr ecx,2
004BAB46    83E9 02         sub ecx,2
004BAB49    83F9 00         cmp ecx,0
004BAB4C    7C 1A           jl short NetClean.004BAB68
004BAB4E    8B048E          mov eax,dword ptr ds:[esi+ecx*4]
```



# 4.看堆栈，找SE句柄，数据窗口跟随，下硬件访问断点，shift+F9

# 5.下断，再次SHIFT+F9，下断，再次SHIFT+F9，取消所有断点，直接F4到RETN处

# 6.下d 12ffc0，下硬件断点。SHIFT+F9。

# 7.记下Stolen Code 



```
004C9B31    55              push ebp
004C9B32    8BEC            mov ebp,esp
004C9B34    6A FF           push -1
```



# 8.找到内存，在00401000处下断，F2，SHIFT+F9



```
004431F9    68 D8B24400     push NetClean.0044B2D8                 //这里其实是假的OEP！！
004431FE    68 B4334400     push NetClean.004433B4                    ; jmp 到
00443203    64:A1 00000000  mov eax,dword ptr fs:[0]
00443209    50              push eax
0044320A    64:8925 0000000>mov dword ptr fs:[0],esp
00443211    83EC 68         sub esp,68
00443214    53              push ebx
```



# 9.写入真实的代码！！

# 10.OD插件脱壳，注意，修正OEP地址！！

# 11.IR修复，用等级3。不能修复的CUT掉。抓取修复。程序还是抱错！

# 12.做下处理。来到壳的入口。也就是第一步记下的地址

# CTRL+G，004AC000

# 13.写入真的OEP代码。

# 14.Lord PE修正入口点

# 15.保存，脱壳成功！

# 脱ACProtect V2.0.X

# 一.常规方法

## 1.忽略除内存访问的所有断点，隐藏OD

## 2.F9，找SE句柄，数据窗口跟随，下内存访问断点

## 3.SHIFT+F9，F2，再SHIFT+F9，F2，再次SHIFT+F9，取消所有断点。

## 4.运行到RETN

## 5.打开内存，0401000处下F2断点。SHIFT+F9

## 6.来到假的OEP。

## 7.修正代码

## 8.脱壳，修复

## 9.OK~！！脱壳完成

# 二.第二种方法

## 1.下断：bp GetCurrentProcessId，F9

## 2.



```
7C809940 >  64:A1 18000000  mov eax,dword ptr fs:[18]  //断在这
7C809946    8B40 20         mov eax,dword ptr ds:[eax+20]
7C809949    C3              retn
```



## 3.用Lord PE查OD的PID。

## 4.把2中的代码改为



```
7C809940 >  B8 3C150000     mov eax,153C （当然，PID号每次都会变）
7C809945    90              nop
7C809946    90              nop
7C809947    90              nop
7C809948    90              nop
7C809949    C3              retn
```



## 5.下断，BP GetModuleHandleA，F9

## 6.



```
7C80B6C1 >  8BFF            mov edi,edi   //断在这
7C80B6C3    55              push ebp
7C80B6C4    8BEC            mov ebp,esp
7C80B6C6    837D 08 00      cmp dword ptr ss:[ebp+8],0
7C80B6CA    74 18           je short kernel32.7C80B6E4
7C80B6CC    FF75 08         push dword ptr ss:[ebp+8]
7C80B6CF    E8 C0290000     call kernel32.7C80E094
7C80B6D4    85C0            test eax,eax
7C80B6D6    74 08           je short kernel32.7C80B6E0
7C80B6D8    FF70 04         push dword ptr ds:[eax+4]
7.来到内存，在CODE断下内存访问断点，F9运行，出现NAG窗口，点确定后，来到OEP！！！
00402150    55              push ebp  //OEP！！
00402151    8BEC            mov ebp,esp
00402153    83C4 F0         add esp,-10
00402156    53              push ebx
00402157    B8 10214000     mov eax,ACP_Feed.00402110
0040215C    E8 4FFCFFFF     call ACP_Feed.00401DB0
00402161    68 C4214000     push ACP_Feed.004021C4                    ; ASCII "ACProtect Feedback Form"
00402166    6A 00           push 0
00402168    6A 00           push 0
```



## 8.脱壳，修复。用等级3！或者手动找MessageBoxA

## 9.抓取，脱壳成功！！

# 另类方法解ACProtect

# 1.常规做法来到oep



```
0040A483    A3 ECFF4000         mov dword ptr ds:[40FFEC],eax  //假OEP
0040A488    E8 16010000         call 跑跑排行.0040A5A3
0040A48D    391D 90FE4000       cmp dword ptr ds:[40FE90],ebx
0040A493    75 0C               jnz short 跑跑排行.0040A4A1
0040A495    68 A0A54000         push 跑跑排行.0040A5A0
0040A49A    FF15 CCC24000       call dword ptr ds:[40C2CC]                      ; msvcrt.__setusermatherr
0040A4A0    59                  pop ecx
0040A4A1    E8 E8000000         call 跑跑排行.0040A58E
0040A4A6    68 1CF04000         push 跑跑排行.0040F01C
0040A4AB    68 18F04000         push 跑跑排行.0040F018
0040A4B0    E8 D3000000         call 跑跑排行.0040A588                              ; jmp 到
0040A4B5    A1 C8FF4000         mov eax,dword ptr ds:[40FFC8]
```



# 2.esp定律，记住f9次数，倒数那次停下，脱壳



```
00441E4D    64:A1 00000000      mov eax,dword ptr fs:[0]        //脱壳最佳时机
00441E53    8905 39CA4200       mov dword ptr ds:[42CA39],eax
00441E59    FF35 39CA4200       push dword ptr ds:[42CA39]
00441E5F    8F05 71CA4200       pop dword ptr ds:[42CA71]
00441E65    FF35 71CA4200       push dword ptr ds:[42CA71]
00441E6B    891C24              mov dword ptr ss:[esp],ebx
00441E6E    890424              mov dword ptr ss:[esp],eax
00441E71    64:8925 00000000    mov dword ptr fs:[0],esp
00441E78    83EC 68             sub esp,68
00441E7B    8905 11C94200       mov dword ptr ds:[42C911],eax
00441E81    FF35 11C94200       push dword ptr ds:[42C911]
00441E87    51                  push ecx
00441E88    B9 91C94200         mov ecx,跑跑排行.0042C991
00441E8D    8BC1                mov eax,ecx
00441E8F    59                  pop ecx
00441E90    8918                mov dword ptr ds:[eax],ebx
```



# 3.修复

先是原oep找表，后是后来的地址转储

# ACProtect之补区段

# 1、忽略所有异常

# 2、打开内存镜像，在.rdata处F2，SHIFT+F9，没有.rdata用.idata

# 3、



```
0043383D    8B46 0C             mov eax,dword ptr ds:[esi+C]     //中断在这里
00433840    0BC0                or eax,eax
00433842    0F84 25020000       je NgaMy.00433A6D
00433848    8366 0C 00          and dword ptr ds:[esi+C],0
0043384C    03C2                add eax,edx
0043384E    8BD8                mov ebx,eax
00433850    56                  push esi
00433851    57                  push edi
00433852    50                  push eax
00433853    8BF3                mov esi,ebx
00433855    8BFB                mov edi,ebx
00433857    AC                  lods byte ptr ds:[esi]
00433858    C0C0 03             rol al,3
0043385B    AA                  stos byte ptr es:[edi]
0043385C    803F 00             cmp byte ptr ds:[edi],0
0043385F  ^ 75 F6               jnz short NgaMy.00433857
00433861    58                  pop eax
00433862    5F                  pop edi
00433863    5E                  pop esi
00433864    50                  push eax
00433865    FF95 90E24100       call dword ptr ss:[ebp+41E290]
0043386B    0BC0                or eax,eax
0043386D    75 43               jnz short NgaMy.004338B2
0043386F    90                  nop
00433870    90                  nop
00433871    90                  nop
00433872    90                  nop
00433873    53                  push ebx
00433874    FF95 94E24100       call dword ptr ss:[ebp+41E294]
0043387A    0BC0                or eax,eax
0043387C    75 34               jnz short NgaMy.004338B2
0043387E    90                  nop
0043387F    90                  nop
00433880    90                  nop
00433881    90                  nop
00433882    8B95 1FFC4000       mov edx,dword ptr ss:[ebp+40FC1F]
00433888    0195 1D1F4000       add dword ptr ss:[ebp+401F1D],edx
0043388E    0195 211F4000       add dword ptr ss:[ebp+401F21],edx
00433894    6A 00               push 0
00433896    FFB5 1D1F4000       push dword ptr ss:[ebp+401F1D]
0043389C    FFB5 211F4000       push dword ptr ss:[ebp+401F21]
004338A2    6A 00               push 0
004338A4    FF95 9CE24100       call dword ptr ss:[ebp+41E29C]
004338AA    6A 00               push 0
004338AC    FF95 98E24100       call dword ptr ss:[ebp+41E298]
004338B2    60                  pushad
004338B3    2BC0                sub eax,eax
004338B5    8803                mov byte ptr ds:[ebx],al
004338B7    43                  inc ebx
004338B8    3803                cmp byte ptr ds:[ebx],al
004338BA  ^ 75 F9               jnz short NgaMy.004338B5
004338BC    61                  popad
004338BD    8985 17FC4000       mov dword ptr ss:[ebp+40FC17],eax
004338C3    C785 1BFC4000 00000>mov dword ptr ss:[ebp+40FC1B],0
004338CD    8B95 1FFC4000       mov edx,dword ptr ss:[ebp+40FC1F]
004338D3    8B06                mov eax,dword ptr ds:[esi]
004338D5    0BC0                or eax,eax
004338D7    75 07               jnz short NgaMy.004338E0
004338D9    90                  nop
004338DA    90                  nop
004338DB    90                  nop
004338DC    90                  nop
004338DD    8B46 10             mov eax,dword ptr ds:[esi+10]
004338E0    03C2                add eax,edx
004338E2    0385 1BFC4000       add eax,dword ptr ss:[ebp+40FC1B]
004338E8    8B18                mov ebx,dword ptr ds:[eax]
004338EA    8B7E 10             mov edi,dword ptr ds:[esi+10]
004338ED    03FA                add edi,edx
004338EF    03BD 1BFC4000       add edi,dword ptr ss:[ebp+40FC1B]
004338F5    85DB                test ebx,ebx
004338F7    0F84 62010000       je NgaMy.00433A5F
004338FD    F7C3 00000080       test ebx,80000000
00433903    75 1D               jnz short NgaMy.00433922
00433905    90                  nop
00433906    90                  nop
00433907    90                  nop
00433908    90                  nop
00433909    03DA                add ebx,edx
0043390B    83C3 02             add ebx,2
0043390E    56                  push esi
0043390F    57                  push edi
00433910    50                  push eax
00433911    8BF3                mov esi,ebx
00433913    8BFB                mov edi,ebx
00433915    AC                  lods byte ptr ds:[esi]
00433916    C0C0 03             rol al,3
00433919    AA                  stos byte ptr es:[edi]
0043391A    803F 00             cmp byte ptr ds:[edi],0
0043391D  ^ 75 F6               jnz short NgaMy.00433915
0043391F    58                  pop eax
00433920    5F                  pop edi
00433921    5E                  pop esi
00433922    3B9D 1FFC4000       cmp ebx,dword ptr ss:[ebp+40FC1F]
00433928    7C 11               jl short NgaMy.0043393B
0043392A    90                  nop
0043392B    90                  nop
0043392C    90                  nop
0043392D    90                  nop
0043392E    83BD 02244000 00    cmp dword ptr ss:[ebp+402402],0
00433935    75 0A               jnz short NgaMy.00433941
00433937    90                  nop
00433938    90                  nop
00433939    90                  nop
0043393A    90                  nop
0043393B    81E3 FFFFFF0F       and ebx,0FFFFFFF
00433941    53                  push ebx
00433942    FFB5 17FC4000       push dword ptr ss:[ebp+40FC17]
00433948    FF95 8CE24100       call dword ptr ss:[ebp+41E28C]
0043394E    3B9D 1FFC4000       cmp ebx,dword ptr ss:[ebp+40FC1F]
00433954    7C 0F               jl short NgaMy.00433965
00433956    90                  nop
00433957    90                  nop
00433958    90                  nop
00433959    90                  nop
0043395A    60                  pushad
0043395B    2BC0                sub eax,eax
0043395D    8803                mov byte ptr ds:[ebx],al
0043395F    43                  inc ebx
00433960    3803                cmp byte ptr ds:[ebx],al
00433962  ^ 75 F9               jnz short NgaMy.0043395D
00433964    61                  popad
00433965    0BC0                or eax,eax
00433967  ^ 0F84 15FFFFFF       je NgaMy.00433882
0043396D    3B85 9CE24100       cmp eax,dword ptr ss:[ebp+41E29C]          //处理MessageBoxA
00433973    74 20               je short NgaMy.00433995                    //NOP掉
00433975    90                  nop
00433976    90                  nop
00433977    90                  nop
00433978    90                  nop
00433979    3B85 9D014100       cmp eax,dword ptr ss:[ebp+41019D]          //处理RegisterHotKey
0043397F    74 09               je short NgaMy.0043398A                    //NOP掉
00433981    90                  nop
00433982    90                  nop
00433983    90                  nop
00433984    90                  nop
00433985    EB 14               jmp short NgaMy.0043399B
00433987    90                  nop
00433988    90                  nop
00433989    90                  nop
0043398A    8D85 0A024100       lea eax,dword ptr ss:[ebp+41020A]
00433990    EB 09               jmp short NgaMy.0043399B
00433992    90                  nop
00433993    90                  nop
00433994    90                  nop
00433995    8D85 24024100       lea eax,dword ptr ss:[ebp+410224]
0043399B    56                  push esi
0043399C    FFB5 17FC4000       push dword ptr ss:[ebp+40FC17]
004339A2    5E                  pop esi
004339A3    39B5 FA234000       cmp dword ptr ss:[ebp+4023FA],esi
004339A9    74 15               je short NgaMy.004339C0
004339AB    90                  nop
004339AC    90                  nop
004339AD    90                  nop
004339AE    90                  nop
004339AF    39B5 FE234000       cmp dword ptr ss:[ebp+4023FE],esi
004339B5    74 09               je short NgaMy.004339C0
004339B7    90                  nop
004339B8    90                  nop
004339B9    90                  nop
004339BA    90                  nop
004339BB    EB 63               jmp short NgaMy.00433A20
004339BD    90                  nop
004339BE    90                  nop
004339BF    90                  nop
004339C0    80BD D2594100 00    cmp byte ptr ss:[ebp+4159D2],0
004339C7    74 57               je short NgaMy.00433A20                //magic跳，改JMP
004339C9    90                  nop
004339CA    90                  nop
004339CB    90                  nop
004339CC    90                  nop
004339CD    EB 07               jmp short NgaMy.004339D6
004339CF    90                  nop
004339D0    90                  nop
004339D1    90                  nop
004339D2    0100                add dword ptr ds:[eax],eax
004339D4    0000                add byte ptr ds:[eax],al
004339D6    8BB5 E4FC4000       mov esi,dword ptr ss:[ebp+40FCE4]
004339DC    83C6 0D             add esi,0D
004339DF    81EE EA1B4000       sub esi,NgaMy.00401BEA
004339E5    2BF5                sub esi,ebp
004339E7    83FE 00             cmp esi,0
004339EA    7F 34               jg short NgaMy.00433A20
004339EC    90                  nop
004339ED    90                  nop
004339EE    90                  nop
004339EF    90                  nop
```



# 4、在00401000处内存访问断点，SHIFT+F9



```
00403D38    68 8C3D4000         push NgaMy.00403D8C             //中断在这里
00403D3D    64:A1 00000000      mov eax,dword ptr fs:[0]
00403D43    50                  push eax
00403D44    8B4424 10           mov eax,dword ptr ss:[esp+10]
00403D48    896C24 10           mov dword ptr ss:[esp+10],ebp
00403D4C    8D6C24 10           lea ebp,dword ptr ss:[esp+10]
00403D50    2BE0                sub esp,eax
00403D52    53                  push ebx
00403D53    56                  push esi
00403D54    57                  push edi
00403D55    8B45 F8             mov eax,dword ptr ss:[ebp-8]
00403D58    8965 E8             mov dword ptr ss:[ebp-18],esp
00403D5B    50                  push eax
00403D5C    8B45 FC             mov eax,dword ptr ss:[ebp-4]
00403D5F    C745 FC FFFFFFFF    mov dword ptr ss:[ebp-4],-1
00403D66    8945 F8             mov dword ptr ss:[ebp-8],eax
00403D69    8D45 F0             lea eax,dword ptr ss:[ebp-10]
00403D6C    64:A3 00000000      mov dword ptr fs:[0],eax
00403D72    C3                  retn                       //运行到这里，然后F8
```



# 5、接着SHIFT+F9



```
00405560    3D 00100000         cmp eax,1000                //中断在这里
00405565    73 0E               jnb short NgaMy.00405575
00405567    F7D8                neg eax
00405569    03C4                add eax,esp
0040556B    83C0 04             add eax,4
0040556E    8500                test dword ptr ds:[eax],eax
00405570    94                  xchg eax,esp
00405571    8B00                mov eax,dword ptr ds:[eax]
00405573    50                  push eax
00405574    C3                  retn                    //F4，然后F8
```



# 6。再SHIFT+F9



```
0040305C    83F9 02             cmp ecx,2              //中断在这里，这里就是假OEP
0040305F    74 0C               je short NgaMy.0040306D
00403061    81CE 00800000       or esi,8000
00403067    8935 B0DE4000       mov dword ptr ds:[40DEB0],esi
0040306D    C1E0 08             shl eax,8
00403070    03C2                add eax,edx
00403072    A3 B4DE4000         mov dword ptr ds:[40DEB4],eax
00403077    33F6                xor esi,esi
00403079    56                  push esi
0040307A    8B3D B0A04000       mov edi,dword ptr ds:[40A0B0]                   ; kernel32.GetModuleHandleA
00403080    FFD7                call edi
00403082    66:8138 4D5A        cmp word ptr ds:[eax],5A4D
```



# 7、脱壳，修复

# 8、重新载入，在pushad 下用ESP定律

hr 0012ffa4，SHIFT+F9（5次），来到这里



```
004365F4    8915 F5FD4100       mov dword ptr ds:[41FDF5],edx                   ; ntdll.KiFastSystemCallRet
004365FA    FF35 F5FD4100       push dword ptr ds:[41FDF5]
00436600    8F05 2DFE4100       pop dword ptr ds:[41FE2D]
00436606    FF35 2DFE4100       push dword ptr ds:[41FE2D]
0043660C    C70424 60000000     mov dword ptr ss:[esp],60
00436613    56                  push esi
00436614    890C24              mov dword ptr ss:[esp],ecx
00436617    68 8DFD4100         push NgaMy.0041FD8D
0043661C    59                  pop ecx
0043661D    8919                mov dword ptr ds:[ecx],ebx
0043661F    8B0C24              mov ecx,dword ptr ss:[esp]
00436622    8F05 ADFE4100       pop dword ptr ds:[41FEAD]
00436628    FF35 8DFD4100       push dword ptr ds:[41FD8D]
0043662E    C70424 48A24000     mov dword ptr ss:[esp],NgaMy.0040A248
00436635    8905 B9FD4100       mov dword ptr ds:[41FDB9],eax
0043663B    FF35 B9FD4100       push dword ptr ds:[41FDB9]
00436641    90                  nop
00436642    90                  nop
00436643    60                  pushad
00436644    E8 01000000         call NgaMy.0043664A             //F4到这里，然后用ESP
```



pushad 上面的就是Stolen Code（NOP可以不复制），复制下来：

89 15 F5 FD 41 00 FF 35 F5 FD 41 00 8F 05 2D FE 41 00 FF 35 2D FE 41 00 C7 04 24 60 00 00 00 56

89 0C 24 68 8D FD 41 00 59 89 19 8B 0C 24 8F 05 AD FE 41 00 FF 35 8D FD 41 00 C7 04 24 48 A2 40

00 89 05 B9 FD 41 00 FF 35 B9 FD 41 00

# 9、hr 0012ff98，F9



```
00436F16    68 1DFD4100         push NgaMy.0041FD1D
00436F1B    58                  pop eax
00436F1C    8930                mov dword ptr ds:[eax],esi
00436F1E    8F05 79FC4100       pop dword ptr ds:[41FC79]
00436F24    8B05 79FC4100       mov eax,dword ptr ds:[41FC79]
00436F2A    FF35 1DFD4100       push dword ptr ds:[41FD1D]
00436F30    56                  push esi
00436F31    891C24              mov dword ptr ss:[esp],ebx
00436F34    C70424 383D4000     mov dword ptr ss:[esp],NgaMy.00403D38
00436F3B    8B3424              mov esi,dword ptr ss:[esp]
00436F3E    8F05 A5FE4100       pop dword ptr ds:[41FEA5]
00436F44    8905 01FF4100       mov dword ptr ds:[41FF01],eax
00436F4A    FF35 01FF4100       push dword ptr ds:[41FF01]
00436F50    891C24              mov dword ptr ss:[esp],ebx
00436F53    56                  push esi
00436F54    C70424 45FE4100     mov dword ptr ss:[esp],NgaMy.0041FE45
00436F5B    8F05 31FE4100       pop dword ptr ds:[41FE31]
00436F61    90                  nop
00436F62    90                  nop
00436F63    60                  pushad                
00436F64    E8 01000000         call NgaMy.00436F6A
```



都是同样处理！！

68 1D FD 41 00 58 89 30 8F 05 79 FC 41 00 8B 05 79 FC 41 00 FF 35 1D FD 41 00 56 89 1C 24 C7 04

24 38 3D 40 00 8B 34 24 8F 05 A5 FE 41 00 89 05 01 FF 41 00 FF 35 01 FF 41 00 89 1C 24 56 C7 04

24 45 FE 41 00 8F 05 31 FE 41 00

# 10、hr 0012ff94，F9



```
0043783F    8B1D 31FE4100       mov ebx,dword ptr ds:[41FE31]                   ; NgaMy.0041FE45
00437845    8933                mov dword ptr ds:[ebx],esi
00437847    8F05 39FC4100       pop dword ptr ds:[41FC39]
0043784D    FF35 39FC4100       push dword ptr ds:[41FC39]
00437853    5B                  pop ebx
00437854    8F05 09FE4100       pop dword ptr ds:[41FE09]
0043785A    891D 21FC4100       mov dword ptr ds:[41FC21],ebx
00437860    FF35 21FC4100       push dword ptr ds:[41FC21]
00437866    C705 19FC4100 09FE4>mov dword ptr ds:[41FC19],NgaMy.0041FE09
00437870    8B1D 19FC4100       mov ebx,dword ptr ds:[41FC19]
00437876    8B33                mov esi,dword ptr ds:[ebx]
00437878    8F05 FDFB4100       pop dword ptr ds:[41FBFD]
0043787E    8B1D FDFB4100       mov ebx,dword ptr ds:[41FBFD]
00437884    FF15 45FE4100       call dword ptr ds:[41FE45]
0043788A    90                  nop
0043788B    90                  nop
0043788C    60                  pushad
0043788D    E8 01000000         call NgaMy.00437893
```



8B 1D 31 FE 41 00 89 33 8F 05 39 FC 41 00 FF 35 39 FC 41 00 5B 8F 05 09 FE 41 00 89 1D 21 FC 41

00 FF 35 21 FC 41 00 C7 05 19 FC 41 00 09 FE 41 00 8B 1D 19 FC 41 00 8B 33 8F 05 FD FB 41 00 8B

1D FD FB 41 00 FF 15 45 FE 41 00

# 11、hr 0012ff24，F9（多几次）



```
0043813D    890D B1FD4100       mov dword ptr ds:[41FDB1],ecx
00438143    FF35 B1FD4100       push dword ptr ds:[41FDB1]
00438149    8F05 B5FC4100       pop dword ptr ds:[41FCB5]
0043814F    FF35 B5FC4100       push dword ptr ds:[41FCB5]
00438155    56                  push esi
00438156    BE FDFC4100         mov esi,NgaMy.0041FCFD
0043815B    893E                mov dword ptr ds:[esi],edi
0043815D    5E                  pop esi
0043815E    FF35 FDFC4100       push dword ptr ds:[41FCFD]
00438164    68 94000000         push 94
00438169    8F05 E5FC4100       pop dword ptr ds:[41FCE5]
0043816F    FF35 E5FC4100       push dword ptr ds:[41FCE5]
00438175    5F                  pop edi
00438176    893D 3DFE4100       mov dword ptr ds:[41FE3D],edi
0043817C    FF35 3DFE4100       push dword ptr ds:[41FE3D]
00438182    8B0C24              mov ecx,dword ptr ss:[esp]
00438185    8F05 7DFE4100       pop dword ptr ds:[41FE7D]
0043818B    90                  nop
0043818C    90                  nop
0043818D    60                  pushad
0043818E    50                  push eax
```



89 0D B1 FD 41 00 FF 35 B1 FD 41 00 8F 05 B5 FC 41 00 FF 35 B5 FC 41 00 56 BE FD FC 41 00 89 3E

5E FF 35 FD FC 41 00 68 94 00 00 00 8F 05 E5 FC 41 00 FF 35 E5 FC 41 00 5F 89 3D 3D FE 41 00 FF

35 3D FE 41 00 8B 0C 24 8F 05 7D FE 41 00

# 12、hr 0012ff1c，F9（多试几次）



```
00438ACD    8B3C24              mov edi,dword ptr ss:[esp]
00438AD0    8F05 79FD4100       pop dword ptr ds:[41FD79]                       ; ntdll.7C930738
00438AD6    8935 25FC4100       mov dword ptr ds:[41FC25],esi
00438ADC    FF35 25FC4100       push dword ptr ds:[41FC25]
00438AE2    890C24              mov dword ptr ss:[esp],ecx
00438AE5    8B3C24              mov edi,dword ptr ss:[esp]
00438AE8    8F05 B9FC4100       pop dword ptr ds:[41FCB9]
00438AEE    8F05 19FE4100       pop dword ptr ds:[41FE19]
00438AF4    8905 89FD4100       mov dword ptr ds:[41FD89],eax
00438AFA    FF35 89FD4100       push dword ptr ds:[41FD89]
00438B00    57                  push edi
00438B01    BF 19FE4100         mov edi,NgaMy.0041FE19
00438B06    8BC7                mov eax,edi
00438B08    5F                  pop edi
00438B09    8B08                mov ecx,dword ptr ds:[eax]
00438B0B    8F05 95FC4100       pop dword ptr ds:[41FC95]
00438B11    8B05 95FC4100       mov eax,dword ptr ds:[41FC95]
00438B17    53                  push ebx
00438B18    90                  nop
00438B19    90                  nop
00438B1A    60                  pushad
00438B1B    50                  push eax
```



8B 3C 24 8F 05 79 FD 41 00 89 35 25 FC 41 00 FF 35 25 FC 41 00 89 0C 24 8B 3C 24 8F 05 B9 FC 41

00 8F 05 19 FE 41 00 89 05 89 FD 41 00 FF 35 89 FD 41 00 57 BF 19 FE 41 00 8B C7 5F 8B 08 8F 05

95 FC 41 00 8B 05 95 FC 41 00 53

# 13、hr 0012ff20，F9



```
004393FF    8F05 5DFE4100       pop dword ptr ds:[41FE5D]                       ; 0012FF40
00439405    FF35 5DFE4100       push dword ptr ds:[41FE5D]
0043940B    890C24              mov dword ptr ss:[esp],ecx
0043940E    893D 91FE4100       mov dword ptr ds:[41FE91],edi
00439414    FF35 91FE4100       push dword ptr ds:[41FE91]
0043941A    8F05 81FC4100       pop dword ptr ds:[41FC81]
00439420    891D 89FE4100       mov dword ptr ds:[41FE89],ebx
00439426    FF35 89FE4100       push dword ptr ds:[41FE89]
0043942C    68 81FC4100         push NgaMy.0041FC81
00439431    5B                  pop ebx
00439432    8B0B                mov ecx,dword ptr ds:[ebx]
00439434    8F05 C9FC4100       pop dword ptr ds:[41FCC9]
0043943A    8B1D C9FC4100       mov ebx,dword ptr ds:[41FCC9]
00439440    57                  push edi
00439441    890424              mov dword ptr ss:[esp],eax
00439444    890C24              mov dword ptr ss:[esp],ecx
00439447    8B0424              mov eax,dword ptr ss:[esp]
0043944A    90                  nop
0043944B    90                  nop
0043944C    60                  pushad
0043944D    76 03               jbe short NgaMy.00439452
```



8F 05 5D FE 41 00 FF 35 5D FE 41 00 89 0C 24 89 3D 91 FE 41 00 FF 35 91 FE 41 00 8F 05 81 FC 41

00 89 1D 89 FE 41 00 FF 35 89 FE 41 00 68 81 FC 41 00 5B 8B 0B 8F 05 C9 FC 41 00 8B 1D C9 FC 41

00 57 89 04 24 89 0C 24 8B 04 24

# 14、hr 0012ff1c，F9



```
00439D39    8F05 D5FD4100       pop dword ptr ds:[41FDD5]                       ; ntdll.KiFastSystemCallRet
00439D3F    8B0C24              mov ecx,dword ptr ss:[esp]
00439D42    8F05 4DFC4100       pop dword ptr ds:[41FC4D]
00439D48    50                  push eax
00439D49    891424              mov dword ptr ss:[esp],edx
00439D4C    8F05 BDFE4100       pop dword ptr ds:[41FEBD]
00439D52    FF35 BDFE4100       push dword ptr ds:[41FEBD]
00439D58    51                  push ecx
00439D59    B9 DDFD4100         mov ecx,NgaMy.0041FDDD
00439D5E    8939                mov dword ptr ds:[ecx],edi
00439D60    59                  pop ecx
00439D61    FF35 DDFD4100       push dword ptr ds:[41FDDD]
00439D67    C705 A9FE4100 60554>mov dword ptr ds:[41FEA9],NgaMy.00405560
00439D71    FF35 A9FE4100       push dword ptr ds:[41FEA9]
00439D77    8B3C24              mov edi,dword ptr ss:[esp]
00439D7A    8F05 95FD4100       pop dword ptr ds:[41FD95]
00439D80    891D 29FD4100       mov dword ptr ds:[41FD29],ebx
00439D86    90                  nop
00439D87    90                  nop
00439D88    60                  pushad
00439D89    E8 01000000         call NgaMy.00439D8F
```



8F 05 D5 FD 41 00 8B 0C 24 8F 05 4D FC 41 00 50 89 14 24 8F 05 BD FE 41 00 FF 35 BD FE 41 00 51

B9 DD FD 41 00 89 39 59 FF 35 DD FD 41 00 C7 05 A9 FE 41 00 60 55 40 00 FF 35 A9 FE 41 00 8B 3C

24 8F 05 95 FD 41 00 89 1D 29 FD 41 00

# 15、hr 0012ff1c，F9



```
0043A6FB    FF35 29FD4100       push dword ptr ds:[41FD29]
0043A701    8BDF                mov ebx,edi
0043A703    8BD3                mov edx,ebx
0043A705    5B                  pop ebx
0043A706    8F05 E9FE4100       pop dword ptr ds:[41FEE9]
0043A70C    8B3D E9FE4100       mov edi,dword ptr ds:[41FEE9]
0043A712    52                  push edx
0043A713    891C24              mov dword ptr ss:[esp],ebx
0043A716    68 9DFE4100         push NgaMy.0041FE9D
0043A71B    5B                  pop ebx
0043A71C    8913                mov dword ptr ds:[ebx],edx
0043A71E    8B1C24              mov ebx,dword ptr ss:[esp]
0043A721    8F05 49FE4100       pop dword ptr ds:[41FE49]
0043A727    8B1424              mov edx,dword ptr ss:[esp]
0043A72A    8F05 69FD4100       pop dword ptr ds:[41FD69]
0043A730    FF15 9DFE4100       call dword ptr ds:[41FE9D]
0043A736    8965 E8             mov dword ptr ss:[ebp-18],esp
0043A739    8925 C5FD4100       mov dword ptr ds:[41FDC5],esp
0043A73F    891D 21FD4100       mov dword ptr ds:[41FD21],ebx
0043A745    FF35 21FD4100       push dword ptr ds:[41FD21]
0043A74B    60                  pushad
0043A74C    74 03               je short NgaMy.0043A751
```



FF 35 29 FD 41 00 8B DF 8B D3 5B 8F 05 E9 FE 41 00 8B 3D E9 FE 41 00 52 89 1C 24 68 9D FE 41 00

5B 89 13 8B 1C 24 8F 05 49 FE 41 00 8B 14 24 8F 05 69 FD 41 00 FF 15 9D FE 41 00 89 65 E8 89 25

C5 FD 41 00 89 1D 21 FD 41 00 FF 35 21 FD 41 00

# 16、hr 0012fe8c，F9



```
0043B097    68 C5FD4100         push NgaMy.0041FDC5
0043B09C    5B                  pop ebx
0043B09D    8B33                mov esi,dword ptr ds:[ebx]
0043B09F    8B1C24              mov ebx,dword ptr ss:[esp]
0043B0A2    8F05 A9FC4100       pop dword ptr ds:[41FCA9]
0043B0A8    893E                mov dword ptr ds:[esi],edi
0043B0AA    57                  push edi
0043B0AB    8F05 F5FE4100       pop dword ptr ds:[41FEF5]
0043B0B1    FF35 F5FE4100       push dword ptr ds:[41FEF5]
0043B0B7    893424              mov dword ptr ss:[esp],esi
0043B0BA    FF15 BCA04000       call dword ptr ds:[40A0BC]                      ; NgaMy.0041F23F
0043B0C0    8B4E 10             mov ecx,dword ptr ds:[esi+10]
0043B0C3    50                  push eax
0043B0C4    B8 F9FB4100         mov eax,NgaMy.0041FBF9
0043B0C9    8910                mov dword ptr ds:[eax],edx
0043B0CB    58                  pop eax
0043B0CC    FF35 F9FB4100       push dword ptr ds:[41FBF9]
0043B0D2    56                  push esi
0043B0D3    C70424 ACDE4000     mov dword ptr ss:[esp],NgaMy.0040DEAC
0043B0DA    8B1424              mov edx,dword ptr ss:[esp]
0043B0DD    8F05 ADFD4100       pop dword ptr ds:[41FDAD]
0043B0E3    890A                mov dword ptr ds:[edx],ecx
0043B0E5    90                  nop
0043B0E6    90                  nop
0043B0E7    60                  pushad
0043B0E8    E8 01000000         call NgaMy.0043B0EE
```



68 C5 FD 41 00 5B 8B 33 8B 1C 24 8F 05 A9 FC 41 00 89 3E 57 8F 05 F5 FE 41 00 FF 35 F5 FE 41 00

89 34 24 FF 15 BC A0 40 00 8B 4E 10 50 B8 F9 FB 41 00 89 10 58 FF 35 F9 FB 41 00 56 C7 04 24 AC

DE 40 00 8B 14 24 8F 05 AD FD 41 00 89 0A

# 17、hr 0012fe8c，F9



```
0043B9DA    8F05 29FE4100       pop dword ptr ds:[41FE29]                       ; 7FFB0000
0043B9E0    FF35 29FE4100       push dword ptr ds:[41FE29]
0043B9E6    5A                  pop edx
0043B9E7    8B46 04             mov eax,dword ptr ds:[esi+4]
0043B9EA    A3 B8DE4000         mov dword ptr ds:[40DEB8],eax
0043B9EF    8B56 08             mov edx,dword ptr ds:[esi+8]
0043B9F2    52                  push edx
0043B9F3    8F05 3DFD4100       pop dword ptr ds:[41FD3D]
0043B9F9    FF35 3DFD4100       push dword ptr ds:[41FD3D]
0043B9FF    8F05 BCDE4000       pop dword ptr ds:[40DEBC]
0043BA05    8B76 0C             mov esi,dword ptr ds:[esi+C]
0043BA08    81E6 FF7F0000       and esi,7FFF
0043BA0E    53                  push ebx
0043BA0F    BB 35FE4100         mov ebx,NgaMy.0041FE35
0043BA14    8933                mov dword ptr ds:[ebx],esi
0043BA16    5B                  pop ebx
0043BA17    FF35 35FE4100       push dword ptr ds:[41FE35]
0043BA1D    8F05 B0DE4000       pop dword ptr ds:[40DEB0]
0043BA23    90                  nop
0043BA24    90                  nop
0043BA25    60                  pushad
0043BA26    E8 01000000         call NgaMy.0043BA2C
```



8F 05 29 FE 41 00 FF 35 29 FE 41 00 5A 8B 46 04 A3 B8 DE 40 00 8B 56 08 52 8F 05 3D FD 41 00 FF

35 3D FD 41 00 8F 05 BC DE 40 00 8B 76 0C 81 E6 FF 7F 00 00 53 BB 35 FE 41 00 89 33 5B FF 35 35

FE 41 00 8F 05 B0 DE 40 00

# 18、hr 0012fe90，F9



```
0043BE77   /EB 01               jmp short NgaMy.0043BE7A  //F8
0043BE79   |E8 FF25BCBE         call BEFFE47D
0043BE7E    43                  inc ebx
0043BE7F    0060 E8             add byte ptr ds:[eax-18],ah
0043BE82    0000                add byte ptr ds:[eax],al
0043BE84    0000                add byte ptr ds:[eax],al
0043BE86    5E                  pop esi
0043BE87    83EE 06             sub esi,6
0043BE8A    B9 66000000         mov ecx,66
0043BE8F    29CE                sub esi,ecx
0043BE91    BA 8A261D6A         mov edx,6A1D268A
0043BE96    C1E9 02             shr ecx,2
0043BE99    83E9 02             sub ecx,2
0043BE9C    83F9 00             cmp ecx,0
0043BE7A  - FF25 BCBE4300       jmp dword ptr ds:[43BEBC]                       ; NgaMy.0040305C //跳到OEP
```



# 19、把所有的代码汇总一下：

89 15 F5 FD 41 00 FF 35 F5 FD 41 00 8F 05 2D FE 41 00 FF 35 2D FE 41 00 C7 04 24 60 00 00 00 56

89 0C 24 68 8D FD 41 00 59 89 19 8B 0C 24 8F 05 AD FE 41 00 FF 35 8D FD 41 00 C7 04 24 48 A2 40

00 89 05 B9 FD 41 00 FF 35 B9 FD 41 00 68 1D FD 41 00 58 89 30 8F 05 79 FC 41 00 8B 05 79 FC 41  

00 FF 35 1D FD 41 00 56 89 1C 24 C7 04 24 38 3D 40 00 8B 34 24 8F 05 A5 FE 41 00 89 05 01 FF 41  

00 FF 35 01 FF 41 00 89 1C 24 56 C7 04 24 45 FE 41 00 8F 05 31 FE 41 00 8B 1D 31 FE 41 00 89 33  

8F 05 39 FC 41 00 FF 35 39 FC 41 00 5B 8F 05 09 FE 41 00 89 1D 21 FC 41 00 FF 35 21 FC 41 00 C7  

05 19 FC 41 00 09 FE 41 00 8B 1D 19 FC 41 00 8B 33 8F 05 FD FB 41 00 8B 1D FD FB 41 00 FF 15 45  

FE 41 00 89 0D B1 FD 41 00 FF 35 B1 FD 41 00 8F 05 B5 FC 41 00 FF 35 B5 FC 41 00 56 BE FD FC 41  

00 89 3E 5E FF 35 FD FC 41 00 68 94 00 00 00 8F 05 E5 FC 41 00 FF 35 E5 FC 41 00 5F 89 3D 3D FE  

41 00 FF 35 3D FE 41 00 8B 0C 24 8F 05 7D FE 41 00 8B 3C 24 8F 05 79 FD 41 00 89 35 25 FC 41 00  

FF 35 25 FC 41 00 89 0C 24 8B 3C 24 8F 05 B9 FC 41 00 8F 05 19 FE 41 00 89 05 89 FD 41 00 FF 35  

89 FD 41 00 57 BF 19 FE 41 00 8B C7 5F 8B 08 8F 05 95 FC 41 00 8B 05 95 FC 41 00 53 8F 05 5D FE  

41 00 FF 35 5D FE 41 00 89 0C 24 89 3D 91 FE 41 00 FF 35 91 FE 41 00 8F 05 81 FC 41 00 89 1D 89  

FE 41 00 FF 35 89 FE 41 00 68 81 FC 41 00 5B 8B 0B 8F 05 C9 FC 41 00 8B 1D C9 FC 41 00 57 89 04  

24 89 0C 24 8B 04 24 8F 05 D5 FD 41 00 8B 0C 24 8F 05 4D FC 41 00 50 89 14 24 8F 05 BD FE 41 00  

FF 35 BD FE 41 00 51 B9 DD FD 41 00 89 39 59 FF 35 DD FD 41 00 C7 05 A9 FE 41 00 60 55 40 00 FF  

35 A9 FE 41 00 8B 3C 24 8F 05 95 FD 41 00 89 1D 29 FD 41 00 FF 35 29 FD 41 00 8B DF 8B D3 5B 8F  

05 E9 FE 41 00 8B 3D E9 FE 41 00 52 89 1C 24 68 9D FE 41 00 5B 89 13 8B 1C 24 8F 05 49 FE 41 00  

8B 14 24 8F 05 69 FD 41 00 FF 15 9D FE 41 00 89 65 E8 89 25 C5 FD 41 00 89 1D 21 FD 41 00 FF 35  

21 FD 41 00 68 C5 FD 41 00 5B 8B 33 8B 1C 24 8F 05 A9 FC 41 00 89 3E 57 8F 05 F5 FE 41 00 FF 35  

F5 FE 41 00 89 34 24 FF 15 BC A0 40 00 8B 4E 10 50 B8 F9 FB 41 00 89 10 58 FF 35 F9 FB 41 00 56  

C7 04 24 AC DE 40 00 8B 14 24 8F 05 AD FD 41 00 89 0A 8F 05 29 FE 41 00 FF 35 29 FE 41 00 5A 8B  

46 04 A3 B8 DE 40 00 8B 56 08 52 8F 05 3D FD 41 00 FF 35 3D FD 41 00 8F 05 BC DE 40 00 8B 76 0C  

81 E6 FF 7F 00 00 53 BB 35 FE 41 00 89 33 5B FF 35 35 FE 41 00 8F 05 B0 DE 40 00

# 20、用工具申请一个新的区段

记下起始的地址：0043E000

# 21、OD打开创建完后的。找到0043E000，粘贴入代码，保存

记住，后面得加跳向假OEP的代码！！

JMP 0040305C  

# 22、修正入口点

# 手脱ASProtect 1.2及1.23

# 一.ASProtect 1.2

最后一次异常法：

在RET处下断

内存镜像00401000下断

直达OEP！！

脱壳，修复。

# 二.ASProtect1.23rc1

## 1.忽略除内存访问的所有断点，隐藏OD

## 2.来到最后一次异常

## 3.RET下断，运行到这。

## 4.下硬件断点 hr XXXX或在00401000下断，运行

## 5.直达OEP

## 6.修复

# 手脱ASProtect1.23 RC4

# 一.版本的判断：

判断版本：

ASProtect 1.23 RC4 按shift+f9键26次后来到典型异常，在最近处的retn处设断，跳过异常，f8步跟就会来到foep。

ASProtect 1.31 04.27 按shift+f9键36次后来到典型异常，在最近处的retn处设断，跳过异常，f8步跟就会来到foep。

ASProtect 1.31 05.18 按shift+f9键40次后来到典型异常，在最近处的retn处设断，跳过异常，f8步跟就会来到foep。

ASProtect 1.31 06.14 按shift+f9键38次后来到典型异常，在最近处的retn处设断，跳过异常，f8步跟就会来到foep。

# 二.

## 1、忽略除内存访问的所有异常，隐藏一下OD，SHIFT+F9 26次后来到最后一次异常



```
00C739EC    3100                xor dword ptr ds:[eax],eax        //断在这
00C739EE    64:8F05 00000000    pop dword ptr fs:[0]
00C739F5    58                  pop eax
00C739F6    833D B07EC700 00    cmp dword ptr ds:[C77EB0],0
00C739FD    74 14               je short 00C73A13
00C739FF    6A 0C               push 0C
00C73A01    B9 B07EC700         mov ecx,0C77EB0
00C73A06    8D45 F8             lea eax,dword ptr ss:[ebp-8]
00C73A09    BA 04000000         mov edx,4
00C73A0E    E8 2DD1FFFF         call 00C70B40
00C73A13    FF75 FC             push dword ptr ss:[ebp-4]
00C73A16    FF75 F8             push dword ptr ss:[ebp-8]
00C73A19    8B45 F4             mov eax,dword ptr ss:[ebp-C]
00C73A1C    8338 00             cmp dword ptr ds:[eax],0
00C73A1F    74 02               je short 00C73A23
00C73A21    FF30                push dword ptr ds:[eax]
00C73A23    FF75 F0             push dword ptr ss:[ebp-10]
00C73A26    FF75 EC             push dword ptr ss:[ebp-14]
00C73A29    C3                  retn
```



## 2、在RETN处下断，SHIFT+F9，取消断点

## 3、在堆栈窗口找显示程序名的下面2行，下HR 12FFA4

## 4、一直F8，到这里



```
00C85C47    BD 4D5CC800         mov ebp,0C85C4D
00C85C4C    FF55 03             call dword ptr ss:[ebp+3]          //到这F7跟
00C85C4F    E8 595CC800         call 0190B8AD
00C85C54    9A E969F29A 5DF3    call far F35D:9AF269E9
00C85C5B    EB 02               jmp short 00C85C5F
00C85C5D    CD20 1BE9EB02       vxdjump 2EBE91B
00C85C63    CD20 33E8EB02       vxdjump 2EBE833
00C85C69    CD20 EB010F8D       vxdcall 8D0F01EB
00C85C6F    6C                  ins byte ptr es:[edi],dx
00C85C70    75 37               jnz short 00C85CA9
00C85C72    5D                  pop ebp
00C85C73    EB 01               jmp short 00C85C76
00C85C75    C7                  ???                                     ; 未知命令
```



## 5、一直F7跟，找抽取代码

### (1)



```
00C84F0A    55                  push ebp
00C84F0B    8BEC                mov ebp,esp
00C84F0D    6A FF               push -1
00C84F0F    68 78E35300         push 53E378
00C84F14    68 407B4F00         push 4F7B40
00C84F19    64:A1 00000000      mov eax,dword ptr fs:[0]
```



55 8B EC 6A FF 68 78 E3 53 00 68 40 7B 4F 00 64 A1 00 00 00 00

### (2)



```
00C84F22    50                  push eax
00C84F23    64:8925 00000000    mov dword ptr fs:[0],esp
00C84F2A    83EC 58             sub esp,58
```



50 64 89 25 00 00 00 00 83 EC 58

### (3)



```
00C84F30    53                  push ebx
```



53

### (4)



```
00C84F34    56                  push esi
```



56

### (5)



```
00C84F38    57                  push edi
00C84F39    8965 E8             mov dword ptr ss:[ebp-18],esp
ptr ss:[ebp-18],esp
```

57 89 65 E8

## 6、把代码汇总一下

55 8B EC 6A FF 68 78 E3 53 00 68 40 7B 4F 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 58 53 56 57 89 65 E8

## 7、继续F7，来到这，就F8



```
00C85B74    51                  push ecx
00C85B75    57                  push edi
00C85B76    9C                  pushfd
00C85B77    FC                  cld
00C85B78    BF B55BC800         mov edi,0C85BB5
00C85B7D    B9 5E140000         mov ecx,145E
00C85B82    F3:AA               rep stos byte ptr es:[edi]
00C85B84    9D                  popfd
00C85B85    5F                  pop edi
00C85B86    59                  pop ecx
00C85B87    C3                  retn
```



## 8、一直F8到这里，然后向上看



```
004F27CF    FF15 9CC25200       call dword ptr ds:[52C29C]
004F27D5    33D2                xor edx,edx
004F27D7    8AD4                mov dl,ah
004F27D9    8915 34306900       mov dword ptr ds:[693034],edx
004F27DF    8BC8                mov ecx,eax
004F27E1    81E1 FF000000       and ecx,0FF
004F27E7    890D 30306900       mov dword ptr ds:[693030],ecx
004F27ED    C1E1 08             shl ecx,8
004F27F0    03CA                add ecx,edx
004F27F2    890D 2C306900       mov dword ptr ds:[69302C],ecx
004F27F8    C1E8 10             shr eax,10
004F27FB    A3 28306900         mov dword ptr ds:[693028],eax
004F2800    6A 01               push 1
004F2802    E8 933B0000         call SoWorker.004F639A
```



## 9、



```
004F27A4    C3                  retn
004F27A5    33C0                xor eax,eax
004F27A7  ^ EB F8               jmp short SoWorker.004F27A1
004F27A9    0000                add byte ptr ds:[eax],al                //这里就是真正的OEP了
004F27AB    0000                add byte ptr ds:[eax],al
004F27AD    0000                add byte ptr ds:[eax],al
004F27AF    0000                add byte ptr ds:[eax],al
004F27B1    0000                add byte ptr ds:[eax],al
004F27B3    0000                add byte ptr ds:[eax],al
004F27B5    0000                add byte ptr ds:[eax],al
004F27B7    0000                add byte ptr ds:[eax],al
004F27B9    0000                add byte ptr ds:[eax],al
004F27BB    0000                add byte ptr ds:[eax],al
004F27BD    0000                add byte ptr ds:[eax],al
004F27BF    0000                add byte ptr ds:[eax],al
004F27C1    0000                add byte ptr ds:[eax],al
004F27C3    0000                add byte ptr ds:[eax],al
004F27C5    0000                add byte ptr ds:[eax],al
004F27C7    0000                add byte ptr ds:[eax],al
004F27C9    0000                add byte ptr ds:[eax],al
004F27CB    0000                add byte ptr ds:[eax],al
004F27CD    0000                add byte ptr ds:[eax],al
004F27CF    FF15 9CC25200       call dword ptr ds:[52C29C]
004F27D5    33D2                xor edx,edx
```



## 10、补上代码，脱壳，修复

# ASProtect之以壳解壳

# 1、忽略除内存访问外的所有异常，SHIFT+F9，来到最后一次异常

# 2、来到最近的RETN

# 3、打开内存镜像，在00401000处下断，SHIFT+F9，到达假OEP，记下



```
004F27CF     FF15 9CC25200       call dword ptr ds:[52C29C]
004F27D5     33D2                xor edx,edx
004F27D7     8AD4                mov dl,ah
004F27D9     8915 34306900       mov dword ptr ds:[693034],edx
004F27DF     8BC8                mov ecx,eax
004F27E1     81E1 FF000000       and ecx,0FF
004F27E7     890D 30306900       mov dword ptr ds:[693030],ecx
004F27ED     C1E1 08             shl ecx,8
004F27F0     03CA                add ecx,edx
004F27F2     890D 2C306900       mov dword ptr ds:[69302C],ecx
004F27F8     C1E8 10             shr eax,10
004F27FB     A3 28306900         mov dword ptr ds:[693028],eax
004F2800     6A 01               push 1
004F2802     E8 933B0000         call SoWorker.004F639A
004F2807     59                  pop ecx
004F2808     85C0                test eax,eax
004F280A     75 08               jnz short SoWorker.004F2814
```



# 4、重新来，重复上面1和2步，下hr 0012ff68，SHIFT+F9，取消断点，F8，来到最佳的以壳解壳地方



```
00C85793     BB A2000000         mov ebx,0A2  //最佳地
00C85798     0BDB                or ebx,ebx
00C8579A     75 07               jnz short 00C857A3
00C8579C     894424 1C           mov dword ptr ss:[esp+1C],eax
00C857A0     61                  popad
00C857A1     50                  push eax
00C857A2     C3                  retn
00C857A3     E8 00000000         call 00C857A8
00C857A8     5D                  pop ebp
00C857A9     81ED 4DE14B00       sub ebp,4BE14D
```



# 5、完整转存，区域转存

# 6、修复