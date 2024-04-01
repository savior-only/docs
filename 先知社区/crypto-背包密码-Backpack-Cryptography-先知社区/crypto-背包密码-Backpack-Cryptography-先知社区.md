---
title: crypto - 背包密码（Backpack Cryptography） - 先知社区
url: https://xz.aliyun.com/t/14209?time__1311=mqmx9QiQi%3DDQ0%3DeDs%3DOSpbwPDvhx7K0eqx
clipped_at: 2024-04-01 16:12:10
category: default
tags: 
 - xz.aliyun.com
---


# crypto - 背包密码（Backpack Cryptography） - 先知社区

在 1978 年，Merkle 和 Hellman 两位杰出的密码学家正式提出了背包问题，并将其确立为一个重要且富有挑战性的研究课题。他们的这一贡献不仅深化了我们对背包问题的理解，也推动了密码学领域的发展。

Merkle 和 Hellman 之所以将背包问题引入密码学领域，是因为他们发现了背包问题的特殊性质：即它具有天然的隐藏信息的潜力。通过精心构造背包问题的参数和条件，Merkle 和 Hellman 设计出了一种基于背包问题的加密系统。这种系统可以将明文信息转化为背包中的物品组合，而解密过程则需要解决背包问题以恢复原始信息。

Merkle 和 Hellman 的工作引起了密码学界的广泛关注。他们的研究不仅为背包问题在密码学中的应用奠定了理论基础，也为后续的研究者提供了宝贵的启示。越来越多的学者开始致力于背包问题的研究，探索其在密码学中的更多可能性。。

今天，就来深入探讨基于背包问题构造的密码 —— 背包密码。

# 1、背包问题

我们都知道密码问题是依靠难解的问题来确保其加密性，背包问题就是背包密码的困难问题。问题如下：

假设一个背包的最多可以承重为 W，有 n 个物体，重量分别为 a1,a2,a3……,an, 并且每个物品只能被装一次，问装哪些物品可以恰好使得背包装满。

用式子表达就是：

b\_1a\_1+b\_2a\_2+b\_3a\_3+……+b\_na\_n=W

a 为物体重量，b 表示有或没有，即 0 或 1，0 表示没有，1 表示有  
背包问题是一个历史悠久且极具挑战性的数学问题。在这个问题中，我们有一个固定容量的背包和一系列不同重量的物品。目标是确定一个物品的组合，使得这些物品的总重量恰好等于背包的容量，同时每个物品只能被选择一次。这个问题的关键在于，随着物品数量和背包容量的增加，找到正确组合的难度会急剧上升，使得问题变得极为复杂和难以解决。

在背包密码的应用中，这个问题被巧妙地转化为加密和解密的过程。加密者可以利用背包问题的困难性，通过构造特定的背包和物品组合来隐藏明文信息。解密者则需要解决这个背包问题，才能恢复出原始的明文信息。由于背包问题的复杂性和难以预测性，使得背包密码具有很高的安全性，能够有效地抵御各种攻击。

# 2、背包密码

根据上面介绍的背包问题，我们精心构造了背包密码体系。在这个体系中，我们利用所有 b 的组合作为二进制表示的明文，而 a 的组合则构成了公钥。这样的设计使得背包密码具有高度的安全性，因为对于大多数人来说，背包难题几乎无法解决，从而无法破解出密文。所以我们也无法破解密文。为了使背包难题可以简单的解决，于是我们就要引入一个特殊的概念 —— **超递增序列**

## 2-1、超递增序列

要求序列中每一项都大于前面所有项之和，这样的序列我们称为超递增序列

例如 \[1,3,6,11,23,45\] 就是一个超递增序列

## 2-2 解决方法

如果 a 的组合为一个超递增序列，则 an 应该大于前面所有的数，因为 b 为 0 或 1，a 不是有就是没有，我们就将 W 与 an 进行比较分两种情况：

### 1、W 大于或等于 an

如果 an 没有，前面的所有项之和小于 an 不可能等于 W，所以 an 一定有

### 2、W 小于 an

W 小于 an 则必然是没有 an 的

于是通过比较 W 与 an 的大小判断出 bn 是 0 还是 1，然后去掉 an 后对 an-1 同样操作，直到变成 0 为止

此外，因为 an 前面所有的数之和小于 an，所以 W 的最大值不会大于或等于 an 的两倍，

## 2-3 例子

\[1,3,6,11,23,45\] 为物体重量，背包承重为 65，按照上面的方法判断，

\[b1,b2,b3,b4,b5\] 应该为 \[0,1,1,1,0,1\]

代码如下：

```plain
def resolve(a,w):
    b = []
    for i in range(len(a)):
        an=a[len(a)-i-1]
        if w<an:
            b.insert(0,0)
        else:
            w=w-an
            b.insert(0,1)
    return b
a=[1,3,6,11,23,45]
w=65
resolve(a,w)
#[0, 1, 1, 1, 0, 1]
```

超递增序列的引入，使得我们在设计背包密码时能够找到一个平衡点。它既保留了背包问题的难解性，保证了密码的安全性，又使得在合法持有者手中，解密过程变得相对简单。这种巧妙的设计使得背包密码成为一种既安全又实用的加密方式，为信息安全领域的发展做出了重要贡献。

值得注意的是，虽然超递增序列使得背包密码在某些情况下变得可解，但这并不意味着背包密码可以被轻易攻破。在实际应用中，我们还需要结合其他安全措施和技术手段，来确保背包密码的安全性得到充分保障。

# 3 背包加密

## 3-1 用私钥构建公钥

私钥：

-   超递增序列 \[a1,a2,a3……，an\]（所装物体重量）
-   例如 \[1,3,6,11,23,45\]

公钥：

-   取模数 m，要求 m 大于序列中所有数之和
-   取临时密钥 n，要求 n 与 m 互素
-   d=n\*amod m
-   \[d1,d2,d3……，dn\] 为公钥
-   \[1,3,6,11,23,45\] 为例，m 取 95，n 取 41，经计算得到的公钥为 \[41, 28, 56, 71, 88, 40\]

脚本如下：

```plain
import gmpy2
def create_pubkey(data,n,m):
    assert m>sum(data),gmpy2.gcd(n,m)==1
    for i in range(len(data)):
        data[i] = data[i] * n % m
    print(data)
    return data, m, n
data=[1,3,6,11,23,45]
n=41
m=95
create_pubkey(data,n,m)
#[41, 28, 56, 71, 88, 40]
```

## 3-2 用公钥进行加密

假如我们取一个二进制数据 011001001011（不够可以补位），加密过程过程如下

-   根据公钥是 6 位，将二进制数分为 011001 001011 分别加密
-   公钥为 \[41, 28, 56, 71, 88, 40\]

011001 加密后为：28+56+40=124

001011 加密后为：56+88+40=184

-   密文为 \[124,184\]
    
    ```plain
    def encrypto(message,pubkey):
      cipher_list = []
      for i in range(len(message)):
          if message[i] == 1:
              cipher_list.append(message[i] * pubkey[i])
      cipher = sum(cipher_list)
      print("加密后的密文为",cipher)
      return cipher
    message=[0,1,1,0,0,1]
    pubkey=[41, 28, 56, 71, 88, 40]
    encrypto(message,pubkey)
    加密后的密文为 124
    ```
    

## 3-3 用私钥解密

当得到密文后，我们需要用私钥和前面的 m 和 n 进行解密，过程如下：

-   计算出 n 的关于 m 的逆元（即 n\*n 的逆元 % m=1）
-   将每个密文乘 n 的逆元在模 m，就能用私钥进行求解
-   仍以上面的数据为例:n 的逆元为 51，对密文进行处理

124\*51%95=54

184\*51%95=74

再用私钥 \[1,3,6,11,23,45\] 进行解密（比较 an 和 W 的那个方法）得到：

54=\[0,1,1,0,0,1\]\*\[1,3,6,11,23,45\]

74=\[0,0,1,0,1,1\]\*\[1,3,6,11,23,45\]

（注意这里不是对 54,74 进行二进制转换哦）

得到明文 011001 001011

```plain
import gmpy2
def decrypto(cipher,n,m,private_key):
    phi_n=gmpy2.invert(n,m)
    cip=cipher*phi_n%m
    mes=resolve(private_key,cip)
    print(mes)
    return mes
private_key=[1,3,6,11,23,45]
cipher=124
decrypto(cipher,n,m,private_key)
```

## 3-4 加密解密过程

```plain
import gmpy2
#解决背包问题
def resolve(a,w):
    b = []
    for i in range(len(a)):
        an=a[len(a)-i-1]
        if w<an:
            b.insert(0,0)
        else:
            w=w-an
            b.insert(0,1)
    return b

#以私钥生成公钥
def create_pubkey(private_key,n,m):
    pubkey=[]
    for i in range(len(private_key)):
        pubkey.append(1)
    assert m>sum(private_key),gmpy2.gcd(n,m)==1
    for i in range(len(private_key)):
        pubkey[i] =private_key[i] * n%m
    print('公钥为：',pubkey)
    return pubkey

#用生成的公钥加密
def encrypto(message,pubkey):
    cipher_list = []
    for i in range(len(message)):
        if message[i] == 1:
            cipher_list.append(message[i] * pubkey[i])
    cipher = sum(cipher_list)
    print("加密后的密文为",cipher)
    return cipher

#处理密文，再用私钥解密
def decrypto(cipher,n,m,private_key):
    phi_n=gmpy2.invert(n,m)
    cip=cipher*phi_n%m
    mes=resolve(private_key,cip)
    print('明文为：',mes)
    return mes
private_key=[1,3,6,11,23,45]
n=41
m=95
message=[0,1,1,0,0,1]
pubkey=create_pubkey(private_key,n,m)
cipher=encrypto(message,pubkey)
mes=decrypto(cipher,n,m,private_key)
#公钥为： [41, 28, 56, 71, 88, 40]
#加密后的密文为 124
#明文为： [0, 1, 1, 0, 0, 1]
```

## 3-5 补充（一些可能有用的函数）

一些题目中可能会用到的函数

### 3-5-1 对乱序列表进行重排

```plain
#对乱序列表进行重排
def relist(pub):
    a = pub[:]
    c = []
    while a:  # 当a不为空时
        current_min = min(a)
        c.append(current_min)
        a.remove(current_min)
    return c
a=relist(pub)
```

### 3-5-2 列表二进制转成十进制数字

```plain
def list_to_int(a):
    m = 0
    for i in range(len(a)):
        m += a[i] * 2 ** (len(a) - i - 1)
    return m
```

### 3-5-3 用乱序私钥进行解密

```plain
#对私钥重排
def relist(pub):
    a = pub[:]
    c = []
    while a:  # 当a不为空时
        current_min = min(a)
        c.append(current_min)
        a.remove(current_min)
    return c
#解密
def resolve(pub,a,w):
    b=[]
    for j in range(len(a)):
        b.append(9)
    for i in range(len(a)):
        an=a[len(a)-i-1]
        id=pub.index(an)
        if w<an:
            b[id:id+1]=[0]
        elif w==an:
            w = w - an
            b[id:id + 1] = [1]
        else:
            w=w-an
            b[id:id+1]=[1]
    if w==0:
        print('成功')
    else:
        print('失败,w=',w)
    return b

def decrypto(pub,w):
    a=relist(pub)
    m=resolve(pub,a,w)
    return m
pub=[10256,299892 , 78079, 158237,  325493403,39577, 2705097, 5549493, 914840, 56822699, 109710911, 19185489]
w=329321410
m=decrypto(pub,w)
print(m)
#[1, 0, 0, 1, 1, 1, 1, 0, 1, 0, 0, 0]
```

上面的一些函数希望能在解题过程中帮助 daon
