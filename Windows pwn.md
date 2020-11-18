---
title: Windows-ROP
date:   2020-07-15 14:10:35
updated: 2020-07-15 14:10:35
tags:
    - windows
    - wiki
    - pwn
categories: 
	- 二进制安全
	- wiki-pwn
use:
  - Valine
text: true
lazyload: true
count: true
---
# 例1

```c
#include <stdio.h>
#define PASSWORD "666666"
int verify_password(char *password)
{
    int authenticated;
    char buffer[8];
    authenticated = strcmp(password,PASSWORD);
    strcpy(buffer,password); //溢出
    return authenticated;
}
void main()
{
    int valid_flag =0;
    char password[128];
    while(1)
    {
        printf("please input password:  ");
        scanf("%s",password);
        valid_flag = verify_password(password);
        if (valid_flag !=0)
        {
            printf("incorrect password!\n");
        }
        else
        {
            printf("Congratulation! You have passed the verification!\n");
            break;
        }
    }
}
```

使用 vc6.0 来编译这个程序，成功后使用 winchecksec 查看所开的防护。

winchecksec安装了半天没装上....以后再说吧

![image.png](https://i.loli.net/2020/11/17/TvE57bBky4iUex6.png)

NX(DEP):栈上不可执行保护

Control Flow Guard(CFG):所有的直接call都会被CFG检查，有一个预先设置的read-only的bitmap。

编译器将间接函数调用的目的地址保存在GuardCFFunction Table中。在执行间接调用函数之前，会检查跳转的这些地址是否存在于表里

GS:类似于Canary

safeSEH:在call异常处理handler之前检查handler是否合法

SEH:异常处理

OD调试：

输入 aaaaaa 看一下程序正常的执行流程。为了方便理解整个过程，在 **strcmp** 函数和 **strcpy** 执行完后下一个断点。

![image.png](https://i.loli.net/2020/11/17/5Qxmav4kgB1HNZb.png)

进入 **strcmp** 这个函数，观察它的返回值。因为 a 的 ascii 码值大于 6 的 ascii 码值，不出意外函数会返回 **1** ，x86 下返回值保存在 EAX 寄存器中，函数正常返回后，由于程序完成它的其余功能还会使用这些寄存器，所以这个返回值会保存在栈上，也就是 **ss:[0012FEA0]** （下图左下角）这个地方。

![image.png](https://i.loli.net/2020/11/17/WFJXlsOVD5wCy1m.png)

当执行到第二个断点时，看一下栈结构。其中 61 是我们输入 a 的 ascii 码形式，**00** 是字符串结束符。那么 **buffer** 的大小是 8 字节，如果我们输入 8 个 a 的话，最后的字符串结束符会溢出到 **0012FEA0** 这个位置把原来的值覆盖为 0，这样我们就可以改变程序的执行流程，输出 Congratulation! You have passed the verification!

![image.png](https://i.loli.net/2020/11/17/qwUuHohvEVXe8b9.png)

# 例2

```c
#include <stdio.h>
#include <windows.h>

#define PASSWORD "1234567"

int verify_password(char *password)
{
    int authenticated;
    char buffer[50];
    authenticated = strcmp(password,PASSWORD);
    memcpy(buffer,password,strlen(password)); //溢出
    return authenticated;
}

void main()
{
    int valid_flag =0;
    char password[1024];
    FILE *fp;

    LoadLibrary("user32.dll");

    if (!(fp=fopen("password.txt","rw+")))
    {
        exit(0);
    }
    fscanf(fp,"%s",password);

    valid_flag 0.= verify_password(password);

    if (valid_flag !=0)
    {
        printf("incorrect password!\n\n");
    }
    else
    {
        printf("Congratulation! You have passed the verification!\n");
    }
    fclose(fp);
    getchar();
}
```

依旧是溢出

od调试：

先在 **memcpy** 处下一个断点，可以先生成 50 BYTES 的 padding 比较与返回地址的距离，最后确定为 60 BYTES 后为返回地址。

![image.png](https://i.loli.net/2020/11/17/BIX3yNz9UrjQdoe.png)

输入的字符串会被复制到栈中 **0012FAE4** （dest）的位置。

![image.png](https://i.loli.net/2020/11/17/7UARK2SzLYgeW9w.png)

所以输入shellcode，然后溢出劫持程序执行流到shellcode即可