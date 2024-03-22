---
title: TUN_TAP
url: https://jasper1024.com/jasper/20240305094009/
clipped_at: 2024-03-14 11:37:32
category: default
tags: 
 - jasper1024.com
---

# TUN\_TAP

发表于 2024-03-05 分类于 [计算机基础 / 计算机网络](https://jasper1024.com/categories/%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9F%BA%E7%A1%80-%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/) 阅读次数：Disqus： [](#disqus_thread "disqus") 本文字数：4.9k

-   Linux 下 tun/tap 介绍；使用流程；简单例程 (c/rust);
    
-   资料来源：
    
    > <>
    
-   更新
    
    |     |     |
    | --- | --- |
    | ```bash<br>1<br>``` | ```bash<br>2024.03.05 初始<br>``` |
    

## [](#%E5%AF%BC%E8%AF%AD "导语")导语

尝试配合 gpts 写一些总结，效果似乎没我更好 😢😢😢

## [](#%E7%AE%80%E8%BF%B0 "简述")简述

tun/tap 设备是 Linux 内核提供的虚拟网络设备。它们允许用户空间程序像处理物理网络设备一样，处理网络数据包。其中：

-   tun (network TUNnel) 设备模拟了基于 IP 的网络设备，操作第三层数据包如 IP 数据包。
-   tap (network tap) 设备模拟了以太网设备，操作第二层数据包如以太网数据帧。

使用 tun/tap 设备，可以实现许多功能，例如：

1.  虚拟专用网络 (VPN): 利用 tun 设备，可以实现点对点 VPN, 加密并通过公网隧道传输两端的网络流量。
2.  网络流量监控：tap 设备可以配置为混杂 (promiscuous) 模式，对网络上所有经过它的数据包进行监听和分析。
3.  协议模拟仿真：利用 tun/tap, 可以在用户空间实现一些自定义或新的网络协议栈，如一些物联网通信协议等。
4.  网络功能虚拟化：tun/tap 是实现 OpenFlow, Open vSwitch 等软件定义网络方案的基础。

## [](#%E4%BD%BF%E7%94%A8 "使用")使用

使用 tun/tap 设备，~需要内核支持，并加载相应的内核模块.~ 一般都支持了。

命令行：`ip tuntap` 命令创建 tun/tap 设备

|     |     |
| --- | --- |
| ```bash<br>1<br>2<br>3<br>4<br>5<br>``` | ```bash<br># 创建类型为tun、名为tun0的设备<br>ip tuntap add dev tun0 mode tun<br># 创建tap0并配置IP和子网掩码<br>ip tuntap add dev tap0 mode tap<br>ip addr add 192.168.1.10/24 dev tap0<br>``` |

编程：

1.  **创建 tun/tap 设备**： `open` 系统调用访问 `/dev/net/tun` 来创建。
2.  **配置设备**：使用 `ioctl` 系统调用来配置新创建的设备，如分配设备名、设置设备模式（tun 或 tap）等。
3.  **设置网络参数**：为 tun/tap 设备分配 IP 地址、路由等，可以使用 `ifconfig` 或 `ip` 命令。
4.  **数据读写**：通过读写文件描述符来接收发送数据。

N 多语言都有非常良好的包装库，使用包装库是个更好的选择。

### [](#%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8%E7%BB%86%E8%8A%82 "系统调用细节")系统调用细节

以 c 为例

|     |     |
| --- | --- |
| ```bash<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>``` | ```bash<br>int tun_alloc(char *dev)<br>{<br>    struct ifreq ifr;<br>    int fd, err;<br><br>    if ((fd = open("/dev/net/tun", O_RDWR)) < 0)<br>        return fd;<br><br>    memset(&ifr, 0, sizeof(ifr));<br>    ifr.ifr_flags = IFF_TUN \| IFF_NO_PI;<br>    strncpy(ifr.ifr_name, dev, IFNAMSIZ);<br><br>    if ((err = ioctl(fd, TUNSETIFF, (void *)&ifr)) < 0) {<br>        close(fd);<br>        return err;<br>    }<br><br>    return fd;<br>}<br>``` |

`open` 正常打开一个文件，模式一般是 `O_RDWR` 可读可写。

`ioctl` 配置参数

-   第二个参数是命令码，这里是 `TUNSETIFF` 设置 tun/tap 设备的工作模式。
-   最后一个是参数，`TUNSETIFF` 命令的参数是 `struct ifreq` 类型
    -   ifr\_name 是 tun/tap 设备名
    -   ifr\_flags 是字符设备标志
        -   IFF\_TUN / IFF\_TAP 代表是 tun 还是 tap
        -   IFF\_NO\_PI 代表不包含附加的包头，默认会带上额外 4 字节。

读取 / 写入即标准的 read 和 write;

## [](#%E4%BE%8B%E7%A8%8B "例程")例程

最小的 tun 设备使用程序，它使用 tun0 设备，打印收到的所有数据包，并回送它们。

### [](#C "C")C

|     |     |
| --- | --- |
| ```bash<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55<br>56<br>57<br>58<br>59<br>``` | ```bash<br>#include <fcntl.h><br>#include <stdlib.h><br>#include <stdio.h><br>#include <string.h><br>#include <unistd.h><br>#include <sys/ioctl.h><br>#include <linux/if.h><br>#include <linux/if_tun.h><br><br>int tun_alloc(char *dev)<br>{<br>    struct ifreq ifr;<br>    int fd, err;<br><br>    if ((fd = open("/dev/net/tun", O_RDWR)) < 0)<br>        return fd;<br><br>    memset(&ifr, 0, sizeof(ifr));<br>    ifr.ifr_flags = IFF_TUN \| IFF_NO_PI;<br>    strncpy(ifr.ifr_name, dev, IFNAMSIZ);<br><br>    if ((err = ioctl(fd, TUNSETIFF, (void *)&ifr)) < 0) {<br>        close(fd);<br>        return err;<br>    }<br><br>    return fd;<br>}<br><br>int main(int argc, char *argv[])<br>{<br>    char tun_name[IFNAMSIZ];<br>    unsigned char buf[1518];<br>    int tun_fd;<br>    int nread;<br><br>    strcpy(tun_name, "tun0");<br><br>    tun_fd = tun_alloc(tun_name);<br><br>    if (tun_fd < 0) {<br>        perror("Could not open tun interface");<br>        exit(1);<br>    }<br><br>    while (1) {<br>        nread = read(tun_fd, buf, sizeof(buf));<br>        if (nread < 0) {<br>            close(tun_fd);<br>            perror("Reading from tun interface");<br>            exit(1);<br>        }<br>        printf("Read %d bytes from tun interface\n", nread);<br><br>        nread = write(tun_fd, buf, nread);<br>        printf("Write %d bytes to tun interface\n", nread);<br>    }<br>    return 0;<br>}<br>``` |

### [](#Rust "Rust")Rust

rust 的 [tun\_tap](https://docs.rs/tun-tap/latest/tun_tap/index.html#)

|     |     |
| --- | --- |
| ```bash<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>``` | ```bash<br>use std::io;<br>use tun_tap::{Iface, Mode};<br><br>fn main() -> io::Result<()> {<br>    let mut config = tun_tap::Configuration::default();<br>    config.name("tun0".into())<br>          .mode(Mode::Tun)<br>          .address((10, 0, 0, 1))<br>          .netmask((255, 255, 255, 0))<br>          .up();<br><br>    let mut nic = Iface::from_configuration(&config)?;<br>    let mut buf = [0u8; 1504];<br><br>    loop {<br>        let nbytes = nic.recv(&mut buf[..])?;<br>        eprintln!("Read {} bytes: {:?}", nbytes, &buf[..nbytes]);<br>        nic.send(&buf[..nbytes])?;<br>    }<br>}<br>``` |

上面的例子仅仅展示了 tun/tap 设备最基本的使用，实际编程中，还需要根据具体应用场景，实现复杂的网络功能。

|     |     |
| --- | --- |
| ```bash<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>``` | ```bash<br>use std::ffi::CString;<br>use std::io;<br>use std::io::prelude::*;<br>use std::os::unix::io::AsRawFd;<br><br>use libc;<br><br>fn main() -> io::Result<()> {<br>    let tun_name = CString::new("tun0").unwrap();<br>    let mut buf = [0u8; 1504];<br><br>    let tun_fd = unsafe {<br>        let fd = libc::open(<br>            "/dev/net/tun\0".as_ptr() as *const libc::c_char,<br>            libc::O_RDWR,<br>        );<br>        if fd < 0 {<br>            panic!("failed to open /dev/net/tun");<br>        }<br><br>        let mut ifr: libc::ifreq = std::mem::zeroed();<br>        ifr.ifr_ifru.ifru_flags = (libc::IFF_TUN \| libc::IFF_NO_PI) as libc::c_short;<br>        std::ptr::copy_nonoverlapping(<br>            tun_name.as_ptr(),<br>            ifr.ifr_ifrn.ifrn_name.as_mut_ptr() as *mut libc::c_char,<br>            tun_name.as_bytes().len(),<br>        );<br><br>        let ret = libc::ioctl(fd, libc::TUNSETIFF, &ifr as *const libc::ifreq);<br>        if ret < 0 {<br>            panic!("ioctl TUNSETIFF failed");<br>        }<br><br>        fd<br>    };<br><br>    let tun = unsafe { std::fs::File::from_raw_fd(tun_fd) };<br><br>    loop {<br>        let nbytes = tun.read(&mut buf)?;<br>        eprintln!("Read {} bytes: {:?}", nbytes, &buf[..nbytes]);<br>        tun.write_all(&buf[..nbytes])?;<br>    }<br>}<br>``` |

## [](#Faq "Faq")Faq

1.  tun 与 tap 设备的区别是什么？
    
    tun 设备是三层设备，操作 IP 数据包；tap 是二层设备，模拟以太网设备，操作链路层帧。
    
2.  tun/tap 收发的数据包包含哪些内容？
    
    对 tun 设备，需要自行构造 IP 头部，操作系统只处理 IP 层之上的部分。对 tap 设备，需要构造完整的以太网数据帧，包括 MAC 地址等。
    
3.  可以在 tun/tap 设备上运行 TCP/IP 协议栈吗？
    
    完全可以。事实上，一个常见的做法是，在 tap 设备上运行一个用户层的轻量级 TCP/IP 协议栈，实现可定制、易调试的虚拟网络。
    
4.  tun/tap 设备是否支持多队列？
    
    Linux 内核从 3.8 版本开始，支持多队列 tun/tap 设备。通过 `TUNSETQUEUE` 配置项，可以请求内核创建多个收发队列。这对于高性能应用很有帮助。
    

除了上面的内容，对 tun/tap 编程，还有一些值得注意的地方：

-   设备何时收到数据包？如果没有数据包，读取设备文件会阻塞。所以实现一个高效的事件循环很重要。`epoll` 或类似机制配合非阻塞 IO, 是常用方法。
-   多线程 / 多进程模型：可以利用多线程 / 进程，充分利用多核性能。通常一个线程用于收包，多个工作线程处理具体业务。
-   用户空间 TCP/IP 协议栈的性能优化：实现用户空间协议栈时，要注意缓存、算法、数据结构的优化。一些性能关键点如 TCP/IP 校验和的计算等，可以用 SIMD 指令集如 AVX512 加速。
-   安全性问题:tun/tap 设备是一种特殊文件，具有较高的特权，不当使用可能带来安全隐患。比如在 VPN 场景下，用户的所有流量都会经过 tun/tap 设备，设备的访问控制和程序的安全性需要仔细设计。

相关文章

-   [BGP 收敛](https://jasper1024.com/jasper/d95c238cv/)
    
-   [BGP 路径属性](https://jasper1024.com/jasper/7892d23vik/)
