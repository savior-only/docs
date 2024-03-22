---
title: 挖洞经验 | 如何发现更多的 IDOR 漏洞（越权漏洞） - FreeBuf 网络安全行业门户
url: https://www.freebuf.com/vuls/223500.html
clipped_at: 2024-03-14 11:48:44
category: default
tags: 
 - www.freebuf.com
---


# 挖洞经验 | 如何发现更多的 IDOR 漏洞（越权漏洞） - FreeBuf 网络安全行业门户

**我非常喜欢搞 IDOR 漏洞，它通常被称为不安全的直接对象引用或是越权，一般来说它的发现手段相对简单，利用方式也不太难，但是对网站业务的危害影响却比较严重。就我来说，我此前发现的一些高危漏洞大部份都属于 IDOR 漏洞范畴之内。今天我们就来谈谈如何发现更多的 IDOR 漏洞。**

## IDOR 漏洞介绍

IDOR，Insecure Direct Object reference，即 "不安全的直接对象引用"，场景为基于用户提供的输入对象进行访问时，未进行权限验证。IDOR 漏洞其实在越权（Broken Access Control）漏洞的范畴之内，也可以说是逻辑漏洞，或是访问控制漏洞，国内通常被称为越权漏洞。具体可[点此参考](https://medium.com/@vickieli/intro-to-idor-9048453a3e5d)。

然而，IDOR 漏洞并不像增减和切换数字 ID 号那样简单，随着应用程序的功能变得越来越复杂，它们引用资源的方式也形式多样，这也意味着简单的数字形式的 IDOR 漏洞在大多数网络应用中变得越来越少。IDOR 在 Web 应用中会以不同的方式体现出来，除了通常的简单数字 ID 号之外，这里我们再来讨论几个值得注意的点。

## 去意想不到的地方寻找 IDOR 漏洞

### 别忘了编码或是哈希过的 ID 号

当我们面对的是一个编码 ID 时，总有可能用某种方法来把这个编码 ID 进行解码。如果 Web 应用使用的是哈希或随机的 ID 编码，此时我们就要看看这个 ID 是否是可猜测的。有时 Web 应用使用的是一些不充分信息熵的算法（algorithms that produce insufficient entropy），其实经过仔细分析后，我们是可以去预测 ID 号的。比如，我们可以注册几个账户去分析这种 ID 号的具体生成模式，然后就得总结得到其它用户 ID 号的生成方法。

另外，也可以通过其它的 API 接口中来窥探一些泄露的随机或编码 ID，比如 Web 应用的一些公开页面，如用户资料信息页面、referer 链接等。

比如，如果我找到一个 API 接口，它的功能是允许用户通过一个编码会话 ID 获取到属于自己的一些详细私信内容，其请求格式如下：

```bash
GET /api_v1/messages?conversation_id=SOME_RANDOM_ID
```

乍一看，其中的的会话 ID（conversation\_id）非常长，而且是随机的字母数字组合序列，但是之后我发现，可以使用用户 ID 号去获取属于每个用户对应的一个会话列表！如下所示：

```bash
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```

而在这个会话列表中就包含了属于用户的会话 ID 号（conversation\_id），又因为用户 ID（user\_id）可以在每个用户的资料页面中公开找到，因此，组合利用这两个 ID 号，我就能通过接口 /api\_v1/messages 去读取任意用户和私信会话内容了！

### 如果无法猜测，可以尝试创建

比如，如果对象引用号（object reference IDs）无法预测，可以看看能有什么操作来影响这种 ID 号的创建或链接过程。

### 给 Web 应用提供一个请求 ID，哪怕它没作要求

如果 Web 应用在请求动作中没有 ID 号要求，那么可以尝试给它添加一个 ID 号看看会发生什么。比如添加一个随机 ID 号、用户 ID、会话 ID，或是其它的对象引用参数，观察服务端的响应内容。如下列请求接口用于显示当前用户所属的私信会话内容：

```bash
GET /api_v1/messages
```

那要是把它换成这种样式，会不会显示出其它用户的会话内容？

```bash
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```

### 使用 HTTP 参数污染方法（HPP,HTTP parameter pollution）

用 HTTP 参数污染方式针对同一参数去给它多个不同的值，这样也是可以导致 IDOR 漏洞的。因为 Web 应用可能在设计时不会料想到用户会为某个参数提交多个不同值，因此，有时可能会导致 Web 后端接口的访问权限绕过。虽然这种情况非常少见，我也从来没遇到，但从理论上来说，它是有可能实现的。如以下请求接口：

```bash
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```

如果对它发起请求失败，那么我们可以尝试发起另外如下的请求：

```bash
GET /api_v1/messages?user_id=YOUR_USER_ID&user_id=ANOTHER_USERS_ID
```

或是这种请求：

```bash
GET /api_v1/messages?user_id=ANOTHER_USERS_ID&user_id=YOUR_USER_ID
```

又或是换成这种把参数列表化的请求：

```bash
GET /api_v1/messages?user_ids[]=YOUR_USER_ID&user_ids[]=ANOTHER_USERS_ID
```

### 使用盲 IDOR 法（Blind IDORs）

有些场景下，易受 IDOR 漏洞影响的接口不会直接响应出来请求查询的信息，但它们可以导致 Web 应用在其它方面泄露信息，如导出文件、邮件或是文本提醒等。

### 改变请求方法

如果某个请求方法无效，那么可以试试其它方法，如 GET, POST, PUT, DELETE, PATCH… 等，一个通常的技巧就是用 PUT 和 POST 进行互换，原因在于服务端的访问控制措施不够完善。

### 改变请求文件的类型

有时，切换请求文件的类型可能会导致 Web 服务端在授权处理上发生不同，如在请求 URL 后加上一个.json，看看响应结果如何。

## 如何提高 IDOR 漏洞的危害性

### 严重性 IDOR 优先

这里，找 IDOR 漏洞时首先要考虑的是其对目标网站关键业务的影响程度，所以，读写型 IDOR 漏洞都属于高危型 IDOR 漏洞。

按照状态改变型 state-changing (可写型) IDOR 漏洞来看，其导致的密码可重置、密码可更改或账户恢复等操作都会对目标网站关键业务造成严重影响，而那种更改邮件订阅设置的 IDOR 漏洞影响就较低。

而对于状态不可改变型 non-state-changing（可读型）IDOR 漏洞来看，我们就要去关注它其中对敏感信息的处理操作，比如，对用户私信会话的处理、对用户敏感信息的处理，或是非公开内容的处理。考虑 Web 应用中哪些功能会处理这些数据，然后有目标的去查找类似 IDOR 漏洞。

### 存储型 XSS（Stored-XSS）

如果把可写型 IDOR 和 self-XSS（需要受害者交互的 XSS）结合，那么就有可能形成针对某个特定用户的存储型 XSS（Stored-XSS）。它的用武之处在哪呢？比如，如果你发现了一个可以更改另外一个用户购物车列表的 IDOR 漏洞，实际来说，该 IDOR 漏洞危害并不高，充其量只会引起受害者的一些麻烦，但如果把它和 self-XSS 配合利用的话，在某个用户提交点，就能通过 IDOR 把 XSS 利用代码传递给受害者浏览器端，最后的效果是，它就形成了针对特定受害用户且无需交互的存储型 XSS 了！

好吧，就聊这么多了，希望大家批评指正。

**\* 参考来源：[medium](https://medium.com/@vickieli/how-to-find-more-idors-ae2db67c9489)，clouds 编译整理，转载请注明来自 FreeBuf.COM**
