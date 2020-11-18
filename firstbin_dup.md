---
title: firstbin_dup
date: 2020-08-12 13:49:09
updated: 2020-08-12 13:49:09
tags:
    - heap
    - pwn
    - firstbin_dup
categories: 
	- 二进制安全
	- how2heap
use:
  - Valine
text: true
lazyload: true
count: true
---

# firstbin_dup

# 源码

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    fprintf(stderr, "This file demonstrates a simple double-free attack with fastbins.\n");

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

    fprintf(stderr, "Now the free list has [ %p, %p, %p ]. If we malloc 3 times, we'll get %p twice!\n", a, b, a, a);
    fprintf(stderr, "1st malloc(8): %p\n", malloc(8));
    fprintf(stderr, "2nd malloc(8): %p\n", malloc(8));
    fprintf(stderr, "3rd malloc(8): %p\n", malloc(8));
}
```

# 输出结果

![image.png](https://i.loli.net/2020/11/17/y54hVxT8ae3ck6j.png)

# 分析&调试

流程很简单，malloc（8）分别给a,b,c

然后free(a)->free(b)->free(a)

(若是直接free(a)*2会检测到并报错)

之后再malloc就会得到a、b、a chunk

两次malloc的指针指向了同一块chunk

调试：
三次malloc之后：
![image.png](https://i.loli.net/2020/11/17/5tc4PKuSgWNhACj.png)

第一次free：

可以看到free的chunk被链入了fastbin中，但是prev_size低位还是1（防止被合并）
![image.png](https://i.loli.net/2020/11/17/YpdoPe4NKMivS9D.png)

![image.png](https://i.loli.net/2020/11/17/CbB1DXFLPeY78gK.png)

第二次free：

可以看到第二次free的chunk从头进入了fastbin，fd指针指向了第一个chunk

![image.png](https://i.loli.net/2020/11/17/bPhXpL3asuHGqVi.png)

第三次free：

可以看到a的chunk又一次从头部链入fastbin，并将其fd改成了chunk2的地址

![image.png](https://i.loli.net/2020/11/17/Pqe9tyhQSpusrYV.png)

之后三次malloc就依次从链表头取chunk（先进后出），第一次和第三次取出的chunk一样

# 总结

- 依旧需要防止和top chunk合并
- fastbin的use位一直是1
- fastbin插入从头部插入
- fastbin是先进后出的