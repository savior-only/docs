---
title: 挖洞经验 | 利用越权漏洞窃取 Airbnb 房东的收款资金 - FreeBuf 网络安全行业门户
url: https://www.freebuf.com/vuls/224431.html
clipped_at: 2024-03-14 11:49:29
category: default
tags: 
 - www.freebuf.com
---


# 挖洞经验 | 利用越权漏洞窃取 Airbnb 房东的收款资金 - FreeBuf 网络安全行业门户

**今天分享的 Writeup 是作者在 2017 年发现，在最近披露的 Airbnb 平台漏洞，漏洞类型为越权（IDOR），攻击者可以利用漏洞向 Airbnb 平台中的房东收款信息中添加进自己的银行账户，从而窃取房东的收款资金。**

> Airbnb 是 AirBed and Breakfast ("Air-b-n-b") 的缩写，中文名爱彼迎，它是旅游人士和房屋房东的中间服务平台，通过该平台，可以为家有空房的房东提供短期租房服务，形式包括假期租赁、公寓租赁、寄宿家庭、招待所床位或酒店客房等，也能让旅行者可以通过网站或手机预订世界各地的各种独特房源，是近年来共享经济发展的代表之一。爱彼迎公司不拥有任何住宿房间，它仅只是住客与房东之间的中间经纪平台，收入来源为每次预订发生时从住客与房东双方收取的一定比例的服务费（佣金），爱彼迎在全球 65,000 个城市和 191 个国家有超过 3,000,000 个预订住宿清单，具体住宿费用由房东根据爱彼迎公司的建议来确定。

## 漏洞介绍

IDOR，Insecure Direct Object reference，即 "不安全的直接对象引用"，也叫越权漏洞，场景为基于用户提供的输入对象进行访问时，Web 应用未进行权限验证，不检查当前访问请求是否有对目标对象的访问权限，因此导致了 IDOR 漏洞。IDOR 漏洞属于失效的访问控制范畴，也可以说是逻辑漏洞，或是访问控制漏洞。测试者可以通过变化请求参数的值来确定该类型漏洞，开发者可以通过源代码分析来确定权限验证是否合理。

房东在 Airbnb 平台进行收款设置时，首先 Airbnb 需要添加收款账户，如果添加成功，接下来它就会为房东生成一个收款 ID-payout\_ID，这个收款 ID 用于之后跨境收款服务商派安盈（Payoneer）生成的收款账户链接，通过该链接则会跳转到一个银行账户添加的页面，在这个页面中，房东可以把自己的银行收款账户填入其中以备后续对住客的收款。然而，就是在房东收款 ID（payout\_ID）生成和银行账户添加链接的跳转过程中，存在 IDOR 漏洞，Airbnb 只确认了收款 ID（payout\_ID）的有效性，却没对用户实际权限做验证，因此，攻击者如果获得了房东的银行账户添加页面链接，攻击者可以用自己的 Airbnb 身份验证 Token (authenticity\_token) 替换掉房东的 Token (authenticity\_token)，成功跳转到属于房东的银行账户添加页面，然后可以把自己的银行账户添加进入，实现窃取房东收款资金的目的。

## 漏洞复现

1、在 Airbnb 平台创建一个受害者账户（Victim），在设置里面添加收款账户信息；

2、开启 BurpSuite 抓包，当收款账户添加之后，可以在 BurpSuite 中捕获到以下 POST 数据包：

```bash
POST /users/payoneer_account_redirect/[payout_ID] HTTP/1.1
Host: www.airbnb.co.in
User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,/;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://www.airbnb.co.in/users/payout_preferences/115687601/new
Cookie: [cookie_values]
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 118
authenticity_token=&user_id=[user_id]
```

3、这里先不着急填写收款账户信息；

4、我们转到 Airbnb 平台再创建一个攻击者账户（Attacker），同样在该攻击者账户中添加收款信息，然后抓取该过程中的数据包，其中会产生一个包括收款 ID（payout\_ID）的收款链接；；

5、注意，在此过程中攻击者账户同样会产生一个包括收款 ID（payout\_ID）的收款链接；

6、这里，我们把攻击者收款链接中的收款 ID（payout\_ID）替换成受害者的收款 ID（payout\_ID），

7、然后把该请求发向 Airbnb 服务端，之后攻击者会收到 Airbnb 服务端响应回来的有效 Payoneer 收款链接；

8、通过这个收款链接可以打开收款账户的填写页面，在其中填入攻击者自己的银行账户，那么受害者的所有收款都会转入到攻击者银行账户。

PoC 视频如下，其中 Chrome 中的 Airbnb 账户为受害者账户，而 Opera 中的账户为攻击者账户：

![Thumbplayer Poster Plugin Image](assets/1710388169-91e61a4c1fcaf342bb30d310a6818b34.jpg)

Notice：上述攻击过程中涉及替换的受害者收款 ID（payout\_ID）必须是没用过的，也就是说作为房东的受害者自己还没有添加过收款信息，针对这样的房东受害者，攻击才能有效。

## 漏洞影响

以上存在漏洞将会影响 Airbnb 平台中至少 20% 到 30% 的房东账户，攻击者只需利用漏洞往其中添加进自己的银行账户，那么房东的后续收款将流向攻击者银行账户，变为攻击者所有。

## 漏洞上报及处理进程

> 2017.2.13 漏洞初报
> 
> 2017.2.15 告知漏洞细节
> 
> 2017.2.28 漏洞分类
> 
> 2017.3.3 漏洞修复
> 
> 2017.3.21 Airbnb 奖励我 3000$
