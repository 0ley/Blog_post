---
title: fastbin_dup_consolidate
date: 2020-08-12 14:43:12
updated: 2020-08-12 14:43:12
tags:
    - heap
    - pwn
    - fastbin_dup_consolidate
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
#include <stdint.h>
#include <stdlib.h>

int main() {
  void* p1 = malloc(0x40);
  void* p2 = malloc(0x40);
  fprintf(stderr, "Allocated two fastbins: p1=%p p2=%p\n", p1, p2);
  fprintf(stderr, "Now free p1!\n");
  free(p1);

  void* p3 = malloc(0x400);
  fprintf(stderr, "Allocated large bin to trigger malloc_consolidate(): p3=%p\n", p3);
  fprintf(stderr, "In malloc_consolidate(), p1 is moved to the unsorted bin.\n");
  free(p1);
  fprintf(stderr, "Trigger the double free vulnerability!\n");
  fprintf(stderr, "We can pass the check in malloc() since p1 is not fast top.\n");
  fprintf(stderr, "Now p1 is in unsorted bin and fast bin. So we'will get it twice: %p %p\n", malloc(0x40), malloc(0x40));
}
```

# 运行结果

![image.png](https://i.loli.net/2020/11/17/1ltNjyrBWCYOXno.png)

# 分析&调试

流程就是先malloc(0x40)*2，之后free掉一个

然后malloc(0x400)

由于这个很大，fastbin中的chunk不适合，就会触发malloc_consolidate

合并fastbin中的chunk之后放到unsorted bin中，之后根据大小再放到small bin、large bin里

那么刚刚free的chunk就到了small bins中

那么再次free同样的指针，那么这同一块chunk就到了fast bin中，同时存在两个不同的bin中了

然后两次malloc，就分别取出了fast bin、small bin中的同一块chunk

调试：

malloc两次之后：
![image.png](https://i.loli.net/2020/11/17/Rpqs9tLOCwPyvQA.png)

free一次后

加到了fast bin中：

![image.png](https://i.loli.net/2020/11/17/w1hmA2v6kcRf5qY.png)

之后malloc一个大一点的数：

可以发现chunk从fast  bin变成了small bin中，并且fd、bk都被修改了（指向头节点）
![image.png](https://i.loli.net/2020/11/17/Pr3UOHMLFdDG95o.png)

再次free p1：

又加入了fast bin，现在存在于两个bin中了！

同时fd指针也被修改了

![image.png](https://i.loli.net/2020/11/17/HaVK8xezJgd5AbR.png)

之后两次malloc就会得到相同的chunk了

# 总结

- 申请一个较大的chunk时，会触发malloc_consolidate，会合并fast bin中的chunk放到unsorted bin中，之后还会根据大小分类放到small bins、large bins中
- chunk链入相应的bins中时，会自动更改一些指针的值。
- 放在不同的bins中时连续两次free检测不到