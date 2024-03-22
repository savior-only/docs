

# sso 打法和思路

- - -

# [](#sso%E6%89%93%E6%B3%95%E6%89%8B%E6%AE%B5%E6%80%9D%E8%80%83)sso 打法 / 手段思考

[![Untitled](assets/1710206911-5fd94c82bfcf0161568b751f0ff9b0b3.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/1.PNG)

## [](#%E8%BF%90%E8%90%A5%E8%A7%92%E5%BA%A6)运营角度

sso 的建设、系统接入改造毕竟是需要成本的 既然目标使用了 sso 那么说明对很大程度上是说明有不小的公司规模

**社工**

1.  目标员工的在使用 sso 的账号通常要么为工号要么为员工姓名缩写甚至手机号
2.  可以从员工在互联网侧泄露的信息入手，例如微博、抖音、小红书、社工库，搜集目标公司人员的相关信息构造字典进一步利用

**有意开白的接口**

实际我接触的公司业务中的 sso，可能收敛了 oa 系统在 sso 后面，比如你访问 oa，自动 302 到 sso 系统 但是实际上该 oa 系统有不少 api 是单独开白访问的，不需要经过 sso

举个例子买的云上销售数据分析服务要从 oa 系统某个接口拉数据同步，它咋自动走 sso 认证？所以后台在 sso 配置中单独对该 oa 的部分路径比如 api，比如静态资源进行了放行

那么当攻击者对这些 api 接口的漏洞进行利用时，其实就是绕过了 sso 的认证撕开了口子

直接访问，直接 302 到 sso

[![Untitled](assets/1710206911-60fee6ca906073dfa6f2026fe80d82ec.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/2.PNG)

访问开白的 api，不会被重定向到 sso

[![Untitled](assets/1710206911-ff33900d7dced40b21349dd1b3d0d0c1.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/3.PNG)

## [](#%E5%BC%80%E5%8F%91%E8%A7%92%E5%BA%A6)开发角度

**从系统实现上看**

前面提到了 sso 的建设、系统接入改造是需要成本的 那么自建的情况下，开发人员往往是会从开源社区中寻求解决方案 比如比较成熟的，cas，或者 spring+shiro 这种形式，这种就是看框架漏洞利用面 如果纯自己实现那么可能就会出现各种奇奇怪怪的漏洞点

**找回密码等功能点的逻辑漏洞**

常规的注入、未授权访问、前端校验就和正常登录点一样测

**多身份源**

较大的企业，其 sso 系统登录时支持的认证来源也会比较丰富 这些不同渠道的认证账密也是一个利用的思路 比如你在内网域里，想控 sso，就可以通过拿域管账号的方式去控 sso

[![Untitled](assets/1710206911-6c7ae2b70f421881b25d07f1249cd27f.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/4.png)

[![Untitled](assets/1710206911-222617bb87a2918e1467643e96e63443.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/5.png)

**otp/mfa**

除了手机验证码外，可能业务会自己开发 otp 的应用，甚至一键扫码进行登陆 往往自研的系统都搭配自研的 otp 应用 如果自研的 otp 应用本身安全性不到位，也会存在被攻击的点，比如攻击者反编译后伪造 token，或者重放 token

还有些终端的场景，比如控了机器，抓浏览器账号密码，发现有 mfa，有账号密码就是登录不了 这种情况可以尝试抓 cookie，通过 cookie 去调用 api 去关自己账号的 mfa，然后登录上去在自己绑定，完成敏感操作

[![Untitled](assets/1710206911-3ed194f6daf3a2c46da32bc09ee0d572.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/6.png)

## [](#%E9%92%93%E9%B1%BC%E8%A7%92%E5%BA%A6)钓鱼角度

**跳转参数**

很多 sso 是登录 sso 跳转到子站点的功能逻辑

例如如下跳转特定业务 url

-   [https://sso.test.com/tologin?redirect=https://evil.com/](https://sso.test.com/tologin?redirect=https://evil.com/)

当 redirect 参数跳转 url 无校验时，我们可以将其改为自己的域名构造恶意链接钓鱼 从而获取目标的 sso cookie，登陆特定业务

**三方 iam 认证 钓鱼**

sso 是需要存认证数据的，当企业人员规模特别大的时候，就会考虑一个叫 iam 的业务

可以这么理解，sso 负责登录、注册、密码找回这些具体功能点实现，iam 负责认证账号、密码这些具体数据 当然有的 iam 业务做的比较大，sso 的功能都包了进去

常见的 iam 国内比如 authing、派拉、雪诺、竹云 国外就是 okta、auth0

一方面从这些厂商产品角度出发，可以挖掘厂商产品的漏洞 另一方面，企业员工在用这些第三方业务认证时常常会忘记密码，这时，忘记密码的邮件是由第三方业务的官方域名发给企业员工的邮箱或者 im 上的，这里就可以尝试购买与厂商类似的域名发钓鱼邮件

例如：authing, 他本身发邮件用的域名就是 .co 官方的看上去都感觉像是钓鱼邮件的后缀

[![Untitled](assets/1710206911-dbef6ce09c594a625cf296b51f14a4cd.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/7.png)

我找了个类似 authing 的域名，然后用 ewomail 进行搭建，格式如下，钓了波公司里的人，上钩率很高。

[![Untitled](assets/1710206911-c48f9963b113d3cb133201d88a7cee71.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/8.png)

- - -

剩下几个案例就放个拓扑吧，具体图不放了，码不好打

[![Untitled](assets/1710206911-6358d3623526a104d8251cc843110176.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/9.png)

[![Untitled](assets/1710206911-1f3e12cad4ab4c658015e9ea2a1940e6.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/10.png)

[![Untitled](assets/1710206911-f3abfc93b2263ff683cada2df10a8bb1.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/11.png)

[![Untitled](assets/1710206911-66050727ff711fabe423bb8a1f7c380e.png)](https://r0fus0d.blog.ffffffff0x.com/img/sso/12.png)
