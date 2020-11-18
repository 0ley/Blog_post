---
title: first fit
date: 2020-08-12 13:07:39
updated: 2020-08-12 13:07:39
tags:
    - heap
    - pwn
    - first fit
categories: 
	- 二进制安全
	- how2heap
use:
  - Valine
text: true
lazyload: true
count: true
---

# 源码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
    fprintf(stderr, "This file doesn't demonstrate an attack, but shows the nature of glibc's allocator.\n");
    fprintf(stderr, "glibc uses a first-fit algorithm to select a free chunk.\n");
    fprintf(stderr, "If a chunk is free and large enough, malloc will select this chunk.\n");
    fprintf(stderr, "This can be exploited in a use-after-free situation.\n");

    fprintf(stderr, "Allocating 2 buffers. They can be large, don't have to be fastbin.\n");
    char* a = malloc(0x512);
    char* b = malloc(0x256);
    char* c;

    fprintf(stderr, "1st malloc(0x512): %p\n", a);
    fprintf(stderr, "2nd malloc(0x256): %p\n", b);
    fprintf(stderr, "we could continue mallocing here...\n");
    fprintf(stderr, "now let's put a string at a that we can read later \"this is A!\"\n");
    strcpy(a, "this is A!");
    fprintf(stderr, "first allocation %p points to %s\n", a, a);

    fprintf(stderr, "Freeing the first one...\n");
    free(a);

    fprintf(stderr, "We don't need to free anything again. As long as we allocate smaller than 0x512, it will end up at %p\n", a);

    fprintf(stderr, "So, let's allocate 0x500 bytes\n");
    c = malloc(0x500);
    fprintf(stderr, "3rd malloc(0x500): %p\n", c);
    fprintf(stderr, "And put a different string here, \"this is C!\"\n");
    strcpy(c, "this is C!");
    fprintf(stderr, "3rd allocation %p points to %s\n", c, c);
    fprintf(stderr, "first allocation %p points to %s\n", a, a);
    fprintf(stderr, "If we reuse the first allocation, it now holds the data from the third allocation.\n");
}
```

# 输出结果

![image.png](https://i.loli.net/2020/11/17/TmyVQncJ8MuXgvk.png)

# 分析&调试

首先

a=malloc(0x512)

b=malloc(0x256)

a='This is A'

free(a)

c=malloc(0x500)

c='This is C'

之后调用a/c都会输出'This is C'

先记几个命令

**heapbase**，基地址

**heapinfo** top、lastreminder和bins信息

![image.png](https://i.loli.net/2020/11/17/4RlI1K3LZeSWFpu.png)

**parseheap** 查看堆信息

![image.png](https://i.loli.net/2020/11/17/rTeW3fYI1cxyMEt.png)

**chunkinfo addr** 查看具体chunk信息

![image.png](https://i.loli.net/2020/11/17/lcUJ6CENwDGjTpF.png)

**magic**  一些有用的地址

![image.png](https://i.loli.net/2020/11/17/h9dCxvMpyZ2DqtP.png)

还有一些

**arenainfo**

**chunkptr**

**printfastbin**

**tracemalloc**

**mergeinfo**

**...**

free之后的变化：

![image.png](https://i.loli.net/2020/11/17/Afnv8aqIjCG3gtz.png)

![image.png](https://i.loli.net/2020/11/17/K5RHlcLMovXJe7n.png)

再次malloc时：

![image.png](https://i.loli.net/2020/11/17/yHGasFq7QJSlWgR.png)

unsorted bin空了

![image.png](https://i.loli.net/2020/11/17/gSujyhUXvFE5JGM.png)

此时a和c共用同一块chunk

![image.png](https://i.loli.net/2020/11/17/WQmlZVx68RtAJNI.png)

# 总结

- glibc使用一种first-fit算法来选择空闲的chunk.如果分配时存在一个大小满足要求的空闲chunk的话,glibc就会选择这个chunk.（并不是最优）
- malloc（b）的作用是为了防止free（a）的时候不和top chunk合并