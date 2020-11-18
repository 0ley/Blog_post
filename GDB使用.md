---
title: GDB使用
date: 2020-08-07 13:35:20
updated: 2020-08-07 13:35:20
tags:
    - pwn
	- gdb
categories: 
	- 二进制安全
	- gdb
use:
  - Valine
text: true
lazyload: true
count: true
---

# 基本

- s step，si步入
- n 执行下一条指令 一般装了pwndbg之后下一行指的是下一条汇编，但是如果调试的程序是带调试信息的，一般会跳几行汇编，ni步入，这个就是真正的下一条汇编指令
- b 在某处下断点，可以用

- - b * adrress
  - b function_name
  - info b 查看断点信息
  - delete 1 删除第一个断点

- c 继续
- r 执行
- **disas addr 查看addr处前后的反汇编代码**
- readelf 文件信息

一般c,n,ni后面都可以跟数字，

ni 10

就代表下10行指令

# 显示数据

## p

- p function 显示某个函数地址
- p $esp 某个寄存器的值
- **p 0xdd-0x55 可以当计算器（常用，不用另外开python了）**
- **p &VarName 查看变量地址（貌似蛮有用的）**
- p * 0xffffebac 查看某个地址处的值

## x

- x/10wx addr 显示某个地址处开始的16进制内容，如果有符号表会加载符号表
- x/40gx addr 跟上面一样，一般上面32位用，这个64位
- x/10s addr 查看addr开始的10个字符串
- x/x $esp 查看esp寄存器中的值
- x/b addr 查看addr处的字符
- **x/10i addr 查看addr处的反汇编结果（addr可以为函数）（常用）**

## info(i)

- **info register $ebp 查看寄存器ebp中的内容 (简写为 i r ebp)**
- i r eflags 查看状态寄存器
- i r ss 查看段寄存器
- i b 查看断点信息
- i functions 查看所有的函数

# 查找数据

- **find 查找字符串 peda带有（常用）**
- searchmem 查找字符串 peda带有
- ropsearch "xor eax,eax;ret" 0x08048080 0x08050000 查找某段的rop peda带有
- ropgadget 提供多个pop|ret可行结果 peda带有

# peda

- **dumpargs 查看调用函数的参数**
- dumprop addr1 addr2 显示addr1-addr之间的ropgadgets
- plt pwntools中的ELF(elf).symbols(function_name)
- got pwntools中的ELF(elf).got(function_name)
- elfsymbol 非调试符号信息