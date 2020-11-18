---
title: tcache
date: 2020-08-10 15:39:57
updated: 2020-08-10 15:39:57
tags:
    - wiki
    - pwn
    - heap
	- tcache
categories: 
	- 二进制安全
	- wiki-pwn-heap
use:
  - Valine
text: true
lazyload: true
count: true
---

tcache 是 glibc 2.26 (ubuntu 17.10) 之后引入的一种技术（see [commit](https://sourceware.org/git/?p=glibc.git;a=commitdiff;h=d5c3fafc4307c9b7a4c7d5cb381fcdbfad340bcc)），目的是提升堆管理的性能。但提升性能的同时舍弃了很多安全检查，也因此有了很多新的利用方式。

# 相关结构体

tcache 引入了两个新的结构体，`tcache_entry` 和 `tcache_perthread_struct`。

这其实和 fastbin 很像，但又不一样。

## tcache_entry

`tcache_entry` 用于链接空闲的 chunk 结构体，其中的 `next` 指针指向下一个大小相同的 chunk。

需要注意的是这里的 next 指向 chunk 的 user data，而 fastbin 的 fd 指向 chunk 开头的地址。

而且，tcache_entry 会复用空闲 chunk 的 user data 部分。

## tcache_perthread_struct

每个 thread 都会维护一个 `tcache_prethread_struct`，它是整个 tcache 的管理结构，一共有 `TCACHE_MAX_BINS` 个计数器和 `TCACHE_MAX_BINS`项 tcache_entry，其中

- `tcache_entry` 用单向链表的方式链接了相同大小的处于空闲状态（free 后）的 chunk，这一点上和 fastbin 很像。
- `counts` 记录了 `tcache_entry` 链上空闲 chunk 的数目，每条链上最多可以有 7 个 chunk。

用图表示大概是：

![image.png](https://i.loli.net/2020/11/17/UP8IdrROqeE6TGZ.png)

# 基本工作方式

- 第一次 malloc 时，会先 malloc 一块内存用来存放 `tcache_prethread_struct` 。
- free 内存，且 size 小于 small bin size 时
- tcache 之前会放到 fastbin 或者 unsorted bin 中
- tcache 后：

- - 先放到对应的 tcache 中，直到 tcache 被填满（默认是 7 个）
  - tcache 被填满之后，再次 free 的内存和之前一样被放到 fastbin 或者 unsorted bin 中
  - tcache 中的 chunk 不会合并（不取消 inuse bit）

- malloc 内存，且 size 在 tcache 范围内

- - 先从 tcache 取 chunk，直到 tcache 为空
  - tcache 为空后，从 bin 中找
  - tcache 为空时，如果 `fastbin/smallbin/unsorted bin` 中有 size 符合的 chunk，会先把 `fastbin/smallbin/unsorted bin` 中的 chunk 放到 tcache 中，直到填满。之后再从 tcache 中取；因此 chunk 在 bin 中和 tcache 中的顺序会反过来