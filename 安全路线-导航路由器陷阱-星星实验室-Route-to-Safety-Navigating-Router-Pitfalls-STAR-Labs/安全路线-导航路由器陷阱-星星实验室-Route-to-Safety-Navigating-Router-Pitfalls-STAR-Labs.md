---
title: 安全路线：导航路由器陷阱|星星实验室 --- Route to Safety: Navigating Router Pitfalls | STAR Labs
url: https://starlabs.sg/blog/2024/route-to-safety-navigating-router-pitfalls/
clipped_at: 2024-03-19 09:50:48
category: default
tags: 
 - starlabs.sg
---


# 安全路线：导航路由器陷阱|星星实验室 --- Route to Safety: Navigating Router Pitfalls | STAR Labs

## Introduction 介绍

Wi-Fi routers have always been an attractive target for attackers. When taken over, an attacker may gain access to a victim’s internal network or sensitive data. Additionally, there has been an ongoing trend of attackers continually [incorporating new router exploits into their arsenal for use in botnets, such as the Mirai Botnet](https://www.zerodayinitiative.com/blog/2023/4/21/tp-link-wan-side-vulnerability-cve-2023-1389-added-to-the-mirai-botnet-arsenal).  
Wi-Fi 路由器一直是攻击者的目标。当被接管时，攻击者可以访问受害者的内部网络或敏感数据。此外，攻击者不断将新的路由器漏洞利用纳入其武器库以用于僵尸网络（例如米拉伊僵尸网络）的趋势一直存在。

Consumer grade devices are especially attractive to attackers, due to many security flaws in them. Devices with lower security often contain multiple bugs that attackers can exploit easily, rendering them vulnerable targets. On the other hand, there are more secure devices that offer valuable insights and lessons to learn from.  
消费级设备对攻击者特别有吸引力，因为它们存在许多安全缺陷。安全性较低的设备通常包含多个漏洞，攻击者可以很容易地利用这些漏洞，使它们成为易受攻击的目标。另一方面，有更安全的设备可以提供有价值的见解和经验教训。

This article gives a technical overview of vulnerabilities in routers for the awareness of security teams and developers, and provides suggestions in ways to avoid making mistakes that could result in such vulnerabilities. We will also look at past vulnerabilities affecting devices of various vendors to learn from their mistakes. Although the following content focuses on routers, the lessons learnt can be applied to other network devices as well.  
本文对路由器中的漏洞进行了技术概述，以提高安全团队和开发人员的意识，并提供了一些建议，以避免犯下可能导致此类漏洞的错误。我们还将研究过去影响各种供应商设备的漏洞，以从他们的错误中吸取教训。虽然以下内容主要集中在路由器上，但所学到的经验教训也可以应用于其他网络设备。

*Disclaimer: This article does not cover all bug classes.  
免责声明：本文并不涵盖所有的 bug 类。*

## Attack Surface 攻击面

A router’s attack surface may be larger than one might expect. This is because there are various services running on it. Every service that receives requests from an external host, either on the local-area network (LAN) or wide-area network (WAN) interface, presents an attack surface as malformed requests may execute a vulnerable code path in the service. Below, we briefly explore the common services on routers that process external requests.  
路由器的攻击面可能比人们预期的要大。这是因为有各种服务在其上运行。无论是在局域网（LAN）还是广域网（WAN）接口上，从外部主机接收请求的每个服务都存在攻击面，因为格式错误的请求可能会执行服务中易受攻击的代码路径。下面，我们将简要介绍路由器上处理外部请求的常见服务。

### Admin Panel 管理面板

The admin panel of a network device hosts a large variety of configurations that could be changed by the device owner/administrator. Every input field is an attack surface as they are processed by the web service running on the device. For example, there may be an input field for blocking traffic to/from a certain IP address. The web service may handle this request in a way that is vulnerable to command injection. In short, the admin panel presents a huge attack surface to an attacker.  
网络设备的管理面板托管了设备所有者 / 管理员可以更改的各种配置。每个输入字段都是一个攻击面，因为它们由设备上运行的 Web 服务处理。例如，可以存在用于阻止去往 / 来自某个 IP 地址的业务的输入字段。Web 服务处理此请求的方式可能容易受到命令注入的攻击。简而言之，管理面板为攻击者提供了一个巨大的攻击面。

One may argue that this is not a very concerning attack surface, because an attacker would need to authenticate into the admin panel first in order to access this attack surface. This is true, and therefore many CVEs start with the term “authenticated”, e.g. “authenticated command injection”, which states that authentication is needed to exploit the vulnerability. However, an attacker may find an authentication bypass or there may be some endpoints on the admin panel that do not verify if the user is authenticated. An authentication bypass can be chained with an “authenticated vulnerability”; vulnerabilities on endpoints that do not require authentication are categorized with the term “unauthenticated”, e.g. “unauthenticated buffer overflow”.  
有人可能会说，这不是一个非常值得关注的攻击面，因为攻击者需要首先在管理面板中进行身份验证才能访问此攻击面。这是真的，因此许多 CVE 都以术语 “经过身份验证” 开头，例如 “经过身份验证的命令注入”，这表明需要进行身份验证才能利用漏洞。但是，攻击者可能会找到绕过身份验证的方法，或者管理面板上的某些端点无法验证用户是否经过身份验证。认证绕过可以与 “认证漏洞” 链接；端点上不需要认证的漏洞被归类为 “未经认证”，例如 “未经认证的缓冲区溢出”。

### Other Services 其他服务

Besides the admin panel, a router usually also runs other services that process requests for various protocols such as FTP, Telnet, Dynamic Host Configuration Protocol (DHCP) or Universal Plug and Play (UPnP). These services also present an attack surface on the router.  
除了管理面板，路由器通常还运行其他服务，处理各种协议的请求，如 FTP，DHCP，动态主机配置协议（DHCP）或通用即插即用（UPnP）。这些服务也会在路由器上构成攻击面。

Some services such as DHCP or UPnP do not require authentication. Furthermore, on some devices, some services are accessible from the WAN interface, which means that a remote attacker that is not on the local network can also access these services and exploit any vulnerability on them. For services that are so accessible, it is especially important to ensure that they are secure.  
某些服务（如 DHCP 或 UPnP）不需要身份验证。此外，在某些设备上，可以从 WAN 接口访问某些服务，这意味着不在本地网络上的远程攻击者也可以访问这些服务并利用它们上的任何漏洞。对于如此容易获得的服务，确保它们的安全尤为重要。

## Poor Configurations 可怜的小家伙

First, we discuss some configuration mistakes present on some routers. The ones that we will discuss all follow the theme of access, namely  
首先，我们讨论一些路由器上存在的一些配置错误。我们将讨论的所有问题都遵循访问的主题，即

-   Access via hardcoded credentials  
    通过硬编码凭据访问
-   Access to services from a remote network  
    从远程网络访问服务
-   Access to root privileges by a running service or normal user  
    正在运行的服务或普通用户访问 root 权限

### Hardcoded Credentials 硬编码凭据

The firmware of a device contains an archive of programs and configuration files that are used by the device for its operations. The same firmware is distributed to and installed on all devices of the same model. Hence, if the credentials for any service (e.g. FTP or Telnet) running on the device is hardcoded in the firmware, the same credentials will be used by every device.  
设备的固件包含设备用于其操作的程序和配置文件的存档。相同的固件分发并安装在同一型号的所有设备上。因此，如果在设备上运行的任何服务（例如 FTP 或 FTP）的凭据在固件中硬编码，则每个设备都将使用相同的凭据。

#### Impact 影响

An attacker who has access to the firmware, which can usually be downloaded from the vendor’s website, can extract the files and inspect them to obtain the credentials. If such services are exposed to the Internet, anyone from anywhere in the world may be able to gain a shell on the device (Telnet), access sensitive data (FTP), or manipulate the device’s settings (httpd), depending on which service is vulnerable.  
可以访问固件（通常可以从供应商的网站下载）的攻击者可以提取文件并检查它们以获取凭据。如果这些服务暴露在互联网上，世界上任何地方的任何人都可以获得设备上的外壳（Shell），访问敏感数据（FTP）或操纵设备的设置（httpd），这取决于哪种服务容易受到攻击。

In a less dangerous situation, such vulnerable services may not be exposed to the Internet, but just the LAN instead. It is less devious, but anyone in the same network as the device will be able to access these exposed services. In the case of a router, anyone who is connected to its Wi-Fi network can abuse the hardcoded credentials on the exposed services. This is arguably fine for a router in a home network, since everyone that knows the Wi-Fi password is a family member or trusted friend. However, this is precarious for routers in public spaces like cafes or restaurants .  
在不太危险的情况下，这种易受攻击的服务可能不会暴露在互联网上，而只是暴露在局域网上。它不那么狡猾，但与设备处于同一网络中的任何人都能够访问这些公开的服务。在路由器的情况下，任何连接到其 Wi-Fi 网络的人都可以滥用暴露服务上的硬编码凭据。对于家庭网络中的路由器来说，这可以说是很好的，因为知道 Wi-Fi 密码的每个人都是家庭成员或值得信赖的朋友。然而，这对于咖啡馆或餐馆等公共场所的路由器来说是不稳定的。

#### Examples 示例

Below, we share some examples of hardcoded credentials on routers that were reported by others in the past.  
下面，我们分享一些过去其他人报告的路由器上硬编码凭据的示例。

[CVE-2022-46637](https://0xsp.com/security%20research%20%20development%20srd/backdoor-discovered-in-pldt-home-fiber-routers/) reports that the ProLink PLDT home fiber PRS1841 U V2 router contains hardcoded Telnet and FTP credentials. Furthermore, these services are exposed to the Internet, and vulnerable devices could be found through Shodan.  
CVE-2022-46637 报告 ProLink PLDT 家用光纤 PRS 1841 U V2 路由器包含硬编码的 FTP 和 FTP 凭据。此外，这些服务暴露在互联网上，可以通过 Shodan 找到易受攻击的设备。

[CVE-2020-29322](https://www.securin.io/zerodays/cve-2020-29322-telnet-hardcoded-credentials/) details that the telnet service of D-Link router DIR-880L with version 1.07 is initiated with hardcoded username and password. From the advisory, it is not conclusive whether the service is always initiated on startup, and whether the service is exposed to the WAN/Internet. However, if both were to be true, this would serve as an easy backdoor for any attacker to gain a shell on the device.  
CVE-2020-29322 详细说明了 1.07 版 D-Link 路由器 Telnet-880 L 的 telnet 服务是使用硬编码的用户名和密码启动的。从咨询中，无法确定服务是否总是在启动时启动，以及服务是否暴露于 WAN/Internet。然而，如果两者都是真的，这将成为任何攻击者在设备上获得外壳的简单后门。

[CVE-2019-14919](https://github.com/InnotecSystem/Device-Reversing/wiki/Firmware-Inspection) is about the Billion Smart Energy Router (SG600R2. Firmw v3.02.rc6) containing a hardcoded password in the login management binary used by Telnet. It is not mentioned in the writeup whether the service is exposed to WAN or LAN only. Besides this, unrelated to hardcoded credentials, a web shell that runs with root privileges is also found in the admin panel.  
CVE-2019-14919 是关于十亿智能能源路由器（SG 600 R2. Firmw v3.02.rc6），其中包含一个硬编码的密码，该密码位于 IBM 使用的登录管理二进制文件中。文章中没有提到服务是只暴露于 WAN 还是 LAN。除此之外，与硬编码的凭据无关，在管理面板中还可以找到一个以 root 权限运行的 web shell。

#### Suggestions 建议

We recommend that all services that require authentication such as FTP or Telnet be disabled before an administrator sets a strong password for them through the admin panel.  
我们建议在管理员通过管理面板设置强密码之前，禁用所有需要身份验证的服务（如 FTP 或 FTP）。

As for the admin panel password, common practice is to randomly generate a password for every device in the factory, and have the password attached as a sticker to the bottom of the device or as a note in the packaging. Upon first login, the administrator would be required to change the password before the admin panel could be used.  
至于管理面板密码，通常的做法是在工厂为每个设备随机生成一个密码，并将密码作为贴纸贴在设备底部或作为包装中的说明。在第一次登录时，管理员需要在使用管理面板之前更改密码。

### Services Exposed to the Internet  
向互联网公开的服务

As mentioned above, attackers can use hardcoded credentials to access services that are exposed to the Internet instead of just being accessible from the LAN. However, if there are no hardcoded credentials, is it then acceptable for services such as the admin panel or FTP server to be exposed to the Internet?  
如上所述，攻击者可以使用硬编码的凭据访问暴露于 Internet 的服务，而不仅仅是从 LAN 访问。但是，如果没有硬编码的凭据，那么可以接受将管理面板或 FTP 服务器等服务暴露给 Internet 吗？

There are many services that may run on a router, such as FTP, SSH, UPnP, admin panel, VPN. Although the credentials may not be hardcoded, there could be vulnerabilities in these services that may not require authentication and may lead to RCE on the device. As such, it is still safer to not expose all services to the Internet unless necessary.  
有许多服务可以在路由器上运行，如 FTP，SSH，UPnP，管理面板，VPN。虽然凭据可能没有硬编码，但这些服务中可能存在不需要身份验证的漏洞，并可能导致设备上的 RCE。因此，除非必要，否则不向 Internet 公开所有服务仍然更安全。

#### Suggestions 建议

It is desirable that services like FTP, Telnet, or SSH not be turned on by default, but only upon the device administrator’s request through the admin panel. If an administrator would like to enable a service, he should be required to specify whether the services are to be exposed to the Internet, or just to the LAN. The device should not be exposing these services to the Internet without the administrator’s knowledge or consent. It is even more desirable for the administrator to be fully aware of such risks and willing to bear them before exposing these services to the Internet.  
最好不要在默认情况下打开 FTP、FTP 或 SSH 等服务，而应在设备管理员通过管理面板提出请求时才打开。如果管理员希望启用某项服务，则应要求他指定该服务是暴露于 Internet，还是仅暴露于 LAN。设备不应在管理员不知情或未经其同意的情况下将这些服务暴露给互联网。更可取的做法是，管理员在将这些服务提供给因特网之前，充分意识到这种风险，并愿意承担这些风险。

### Services Running as `root` 服务运行为 `root`

On a PC, it is common to separate normal users from superusers (root/administrator). However, this is not the case for many consumer routers. Many of them only have the `root` user, and every service runs as `root`. On such devices, the FTP/Telnet/SSH/admin panel web services are running as `root`. So, when an attacker obtains RCE on a service, he also has a shell with `root` privileges. There is no privilege separation to prevent the whole device from being compromised when a service is exploited.  
在 PC 上，通常将普通用户与超级用户（root/administrator）分开。然而，对于许多消费者路由器来说，情况并非如此。他们中的许多人只有 `root` 用户，每个服务都以 `root` 运行。在此类设备上，FTP/SSH/admin 面板 Web 服务作为 `root` 运行。因此，当攻击者在服务上获得 RCE 时，他还拥有一个具有 `root` 权限的 shell。没有特权分离，以防止整个设备在服务被利用时受到损害。

We list some examples below:  
我们在下面列出一些例子：

-   [↓↓↓](https://blog.thalium.re/posts/rooting-xiaomi-wifi-routers/#binary-usrbinmessagingagent---command-injection-cve-2023-26317:~:text=All%20binaries%20are%20executed%20as%20root)  
      
    MI AIoT Router AC2350  
    MI AIoT 路由器 AC 2350  
      
    [↑↑↑](https://blog.thalium.re/posts/rooting-xiaomi-wifi-routers/#binary-usrbinmessagingagent---command-injection-cve-2023-26317:~:text=All%20binaries%20are%20executed%20as%20root)
    
-   [↓↓↓](https://devttys0.com/2015/04/hacking-the-d-link-dir-890l/#:~:text=unauthenticated%20root%20shell)  
      
    D-Link DIR-890L  D-Link 双链机 - 890 L  
      
    [↑↑↑](https://devttys0.com/2015/04/hacking-the-d-link-dir-890l/#:~:text=unauthenticated%20root%20shell)
    
-   [NETGEAR Nighthawk RAX30](https://research.nccgroup.com/2022/12/22/puckungfu-a-netgear-wan-command-injection/#root-shell)

The writeups above show that a `root` shell could be obtained without a privilege escalation exploit, which means that there is no privilege separation.  
上面的文章表明，可以在没有特权提升漏洞的情况下获得 `root` shell，这意味着没有特权分离。

#### Suggestions 建议

Every process/service should be running as a different user, applying the principle of least privileges. For example, the `httpd` service runs as the `web` user; the `upnpd` service runs as the `upnp` user; the `ftp` service runs as the `ftp` user, and so on. Under this configuration, when a service is exploited, the attacker cannot compromise other services or the whole device without a privilege escalation exploit.  
每个进程 / 服务都应该以不同的用户身份运行，应用最小特权原则。例如， `httpd` 服务以 `web` 用户的身份运行；`upnpd` 服务以 `upnp` 用户的身份运行；`ftp` 服务以 `ftp` 用户的身份运行等等。在此配置下，当服务被利用时，攻击者无法在不利用权限提升漏洞的情况下危害其他服务或整个设备。

### Password-less `sudo`? 无密码 `sudo` ？

On some of the routers that we have inspected, they do not run their admin panel web service as `root` but a normal user. This is good. However, to perform some system-level operations, they want to make use of shell commands which require `root` privileges. For example, the `iptables` command.  
在我们检查过的一些路由器上，它们不以 `root` 的身份运行管理面板 Web 服务，而是以普通用户的身份运行。这很好但是，为了执行一些系统级操作，他们希望使用需要 `root` 特权的 shell 命令。例如， `iptables` 命令。

#### Bad Example 坏榜样

Consider the scenario where the admin panel supports blocking traffic to and from a certain IP address using `iptables` (requires superuser privileges). To achieve this, developers may be tempted to introduce a SUID binary that works like `sudo` but does not need a password. In other words, a SUID binary that takes a command string as argument and runs that command string as a shell command with `root` privileges. A simple example of this is shown below (let’s refer to it as `custom_sudo` in the following examples):  
考虑管理面板支持使用 `iptables` （需要超级用户权限）阻止来自或去往某个 IP 地址的流量的场景。为了实现这一点，开发人员可能会引入一个像 `sudo` 一样工作但不需要密码的 SUID 二进制文件。换句话说，一个 SUID 二进制文件，它接受一个命令字符串作为参数，并将该命令字符串作为具有 `root` 权限的 shell 命令运行。下面是一个简单的例子（在下面的例子中，我们将其称为 `custom_sudo` ）：

```c
int main(int argc, char** argv)
{
  setuid(geteuid());
  system(argv[1]);
  return 0;
}
```

Then, the developers may use `custom_sudo` to run an `iptables` command as follows.  
然后，开发者可以使用 `custom_sudo` 来运行如下的 `iptables` 命令。

```c
snprintf(cmd, 255, "iptables -A INPUT -s '%s' -j DROP", ip_addr_to_block);
execve("/usr/sbin/custom_sudo", { "/usr/sbin/custom_sudo", cmd, 0 }, 0);
```

Obviously, having such a program defeats the purpose of having different users, when all users can just use this program to run any commands as `root`, violating the principle of least privileges.  
显然，拥有这样一个程序违背了拥有不同用户的目的，当所有用户都可以使用这个程序来运行任何命令时，就像 `root` 一样，违反了最小特权原则。

#### Suggestions 建议

It is not recommended that operations requiring `root` privileges be executed by means of providing a whole command string to such a SUID program as `sudo` or `custom_sudo`. Instead, we suggest having a SUID program that only takes in the necessary values as arguments and then use them in carrying out the desired operations.  
不建议通过向 `sudo` 或 `custom_sudo` 等 SUID 程序提供整个命令串的方式执行需要 `root` 权限的操作。相反，我们建议有一个 SUID 程序，它只接受必要的值作为参数，然后在执行所需的操作时使用它们。

For the example above on blocking an IP address, we suggest having a SUID program called `block_ip_addr` that just takes in the IP address as an argument, then performs the `iptables` operation with the given IP address string internally. For example:  
对于上面关于阻止 IP 地址的示例，我们建议使用一个名为 `block_ip_addr` 的 SUID 程序，该程序只接受 IP 地址作为参数，然后在内部使用给定的 IP 地址字符串执行 `iptables` 操作。举例来说：

```c
execve("/usr/sbin/block_ip_addr", { "/usr/sbin/block_ip_addr", ip_addr_to_block, 0 }, 0);
```

Then, the implementation of `block_ip_addr` can be as follows:  
然后， `block_ip_addr` 的实现可以如下：

```c
char* ip_addr_to_block = argv[1];
execve("/bin/iptables", { "/bin/iptables", "-A", "INPUT", "-s", ip_addr_to_block, "-j", "DROP", 0 }, 0);
```

If this is done this for all commands that require `root` privileges, the `custom_sudo` program is no longer necessary and can be removed entirely from the system.  
如果对所有需要 `root` 权限的命令都这样做，则不再需要 `custom_sudo` 程序，并且可以从系统中完全删除。

However, if a developer strongly insists on using such a `custom_sudo` program, please at least verify that the program that will be executed is among a list of allowed programs. For example:  
但是，如果开发人员坚持使用这样的 `custom_sudo` 程序，请至少验证将执行的程序是否在允许的程序列表中。举例来说：

```c
if (strncmp(argv[1], "/bin/iptables ", strlen("/bin/iptables "))) {
  // if strncmp returns a non-zero value, the command string does not start with the expected `iptables `
  // abort
  ...
}
```

Note that in the check above, `/bin/iptables` ends with a space character. This is to make sure that the command string indeed is running the `/bin/iptables` program, and not something like `iptablesfake`. Also, use the absolute path (e.g. `/bin/iptables`) and not just the program name (e.g. `iptables`), to prevent the invocation of the wrong program, as the program path is determined according to the `PATH` environment variable. For example `/home/user/iptables` created by an attacker may be executed instead of `/bin/iptables` if the `PATH` variable is configured as so.  
请注意，在上面的检查中， `/bin/iptables` 以空格字符结尾。这是为了确保命令字符串确实在运行 `/bin/iptables` 程序，而不是类似于 `iptablesfake` 的东西。此外，使用绝对路径（例如 `/bin/iptables` ）而不仅仅是程序名（例如 `iptables` ），以防止调用错误的程序，因为程序路径是根据 `PATH` 环境变量确定的。例如，如果 `PATH` 变量被配置为这样，则攻击者创建的 `/home/user/iptables` 可以被执行而不是 `/bin/iptables` 。

Also, make sure that there are no command injection vulnerabilities that could be exploited to bypass such checks. This could be done by running the desired command with `execve` instead of `system`.  
此外，请确保不存在可被利用来绕过此类检查的命令注入漏洞。这可以通过使用 `execve` 而不是 `system` 运行所需的命令来完成。

In short, when considering the implementation of operations that require superuser privileges, the principle of least privileges should be followed to ensure that the chosen implementation does not provide a user with more privileges than necessary.  
简而言之，在考虑需要超级用户权限的操作的实现时，应该遵循最小权限原则，以确保所选择的实现不会为用户提供超出必要的权限。

### Summary 总结

All of the misconfigurations above have straightforward solutions. However, the implementation of these solutions incur extra development and testing time. For the consumers, it is desirable that all router vendors consider these improvements as “must have” and not just “good to have”.  
上面所有的错误配置都有简单的解决方案。但是，这些解决方案的实现会导致额外的开发和测试时间。对于消费者来说，希望所有路由器供应商都将这些改进视为 “必须拥有”，而不仅仅是 “好拥有”。

## Vulnerability Classes 漏洞类别

In this section, we will discuss the following vulnerabilities affecting services running on routers.  
在本节中，我们将讨论以下影响路由器上运行的服务的漏洞。

-   Authentication Bypass 身份验证绕行
-   Command Injection 命令注入
-   Buffer Overflow 缓冲区溢出
-   Format String Bug 格式字符串缺陷

Command injection and buffer overflow are the two main vulnerability classes present in routers that lead to RCE. Format string vulnerabilities may also result in RCE, but are very rarely seen in the recent years.  
命令注入和缓冲区溢出是路由器中导致 RCE 的两个主要漏洞类别。格式字符串漏洞也可能导致 RCE，但近年来很少见到。

### Authentication Bypass 身份验证绕行

The attack surface of a service is greatly reduced if the service requires authentication. Typically, an unauthenticated client would only be able to send requests related to authentication, or for querying some information about the service or the device. Therefore, an authentication bypass vulnerability is valuable to attackers as it opens up the whole remaining attack surface.  
如果服务需要身份验证，则服务的攻击面将大大减少。通常，未经身份验证的客户端只能发送与身份验证相关的请求，或者查询有关服务或设备的某些信息。因此，身份验证绕过漏洞对攻击者很有价值，因为它打开了整个剩余的攻击面。

Besides that, even without RCE, a non-administrator could still perform an authentication bypass to disclose and control sensitive settings on the admin panel. An authentication bypass on other services that require authentication such as FTP, SSH or Telnet could also lead to shell access or disclosure of sensitive information. Hence, authentication bypass should not be taken lightly.  
除此之外，即使没有 RCE，非管理员仍然可以执行身份验证绕过，以披露和控制管理面板上的敏感设置。在其他需要身份验证的服务（如 FTP、SSH 或 SSL）上绕过身份验证也可能导致 shell 访问或敏感信息泄露。因此，身份验证旁路不应掉以轻心。

#### Examples 示例

In the following sub-sections, we share examples of authentication bypass bugs on routers that were reported in the past.  
在下面的小节中，我们将分享过去报告的路由器上的身份验证绕过错误的示例。

##### CVE-2021-32030: Mistake in authentication logic  
CVE-2021-32030：身份验证逻辑错误

[CVE-2021-32030](https://github.com/atredispartners/advisories/blob/master/ATREDIS-2020-0010.md) reports an authentication bypass on the ASUS GT-AC2900 router. In short, the vulnerability arises due to a mistake in the authentication flow as simplified below:  
CVE-2021-32030 报告了华硕 GT-AC 2900 路由器上的身份验证绕过。简而言之，该漏洞是由于身份验证流程中的错误引起的，简化如下：

1.  A header field `asus_token` should be supplied by the client for authenticating into the admin panel.  
    客户端应提供一个头字段 `asus_token` ，以便在管理面板中进行身份验证。
2.  `asus_token` is compared with the `ifttt_token` value retrieved from \*`nvram`.  
    `asus_token` 与从 \* `nvram` 检索的 `ifttt_token` 值进行比较。
3.  If `nvram` does not contain `ifttt_token`, it returns an empty string, i.e. a null-byte.  
    如果 `nvram` 不包含 `ifttt_token` ，则返回空字符串，即空字节。
4.  If `asus_token` is a null byte, the comparison succeeds and the client is authenticated successfully.  
    如果 `asus_token` 为空字节，则比较成功，客户端认证成功。

\*`nvram` stands for non-volatile RAM. It is used by many routers to store configuration values that should persist over device reboot. Hence the term non-volatile, as the contents persist, unlike for normal RAM in which all its contents will be cleared when the device is turned off.  
\* `nvram` 代表非易失性 RAM。许多路由器都使用它来存储配置值，这些配置值在设备重新启动后应保持不变。因此，术语非易失性，因为内容持续存在，不像普通 RAM，当设备关闭时，其所有内容都将被清除。

###### Suggestions 建议

In this case, the vulnerability is caused by programmer error, in which an unexpected edge case input breaks the authentication logic. The relevant function should first ensure that `ifttt_token` is not an empty string, before comparing it with the client-supplied `asus-token`. To be extra careful, the developers may also add an additional check to ensure that `asus-token` is not an empty string, in case it may be compared with another token that is also retrieved from nvram in the future.  
在这种情况下，漏洞是由程序员错误引起的，其中意外的边缘情况输入会破坏身份验证逻辑。相关函数应该首先确保 `ifttt_token` 不是空字符串，然后再将其与客户端提供的 `asus-token` 进行比较。为了更加小心，开发人员还可以添加一个额外的检查，以确保 `asus-token` 不是空字符串，以防将来它可能会与另一个也从 nvram 中检索的令牌进行比较。

##### CVE-2020-8864: Mistake in authentication logic  
CVE-2020-8864：身份验证逻辑错误

[CVE-2020-8864](https://www.zerodayinitiative.com/blog/2020/9/30/the-anatomy-of-a-bug-door-dissecting-two-d-link-router-authentication-bypasses#:~:text=ZDI%2D20%2D268/-,CVE%2D2020%2D8864,-This%20authentication%20bypass) reports an authentication bypass on the D-Link DIR-882, DIR-878 and DIR-867 routers. The exploit is the same as the one above in the ASUS router, although the implementation of authentication logic is different, as simplified below:  
CVE-2020-8864 报告了 D-Link 路由器 <$-882、<$-878 和 <$-867 上的身份验证绕过。该漏洞与 ASUS 路由器中的漏洞相同，尽管身份验证逻辑的实现不同，简化如下：

1.  `LoginPassword` is provided by the client for authentication.  
    `LoginPassword` 由客户端提供用于身份验证。
2.  `strncmp` is used to compare the client-supplied password with the correct password, as shown below:  
    `strncmp` 用于将客户端提供的密码与正确的密码进行比较，如下图所示：
    
    ```c
    strncmp(db_password, attacker_provided_password, strlen(attacker_provided_password));
    ```
    
3.  If an attacker submits a login request with an empty password, `strncmp` will have 0 as its 3rd argument (length), and that returns 0, meaning the two strings compared are equal, which is correct because the first 0 characters of both strings are the same. As a result, authentication is successful.  
    如果攻击者提交了一个空密码的登录请求， `strncmp` 将以 0 作为其第三个参数（长度），并返回 0，这意味着比较的两个字符串是相等的，这是正确的，因为两个字符串的前 0 个字符是相同的。因此，身份验证成功。

###### Suggestions 建议

Again, the vulnerability is caused by programmer error. The fix here is by passing `strlen(db_password)` as the 3rd argument, instead of `strlen(attacker_provided_password)`.  
同样，漏洞是由程序员错误引起的。这里的修复是通过传递 `strlen(db_password)` 作为第三个参数，而不是 `strlen(attacker_provided_password)` 。

##### CVE-2020-8863: Expected password value is controlled by attacker  
CVE-2020-8863：预期密码值由攻击者控制

[CVE-2020-8863](https://www.zerodayinitiative.com/blog/2020/9/30/the-anatomy-of-a-bug-door-dissecting-two-d-link-router-authentication-bypasses#:~:text=ZDI%2D20%2D267/-,CVE%2D2020%2D8863,-The%20title%20for) reports another authentication bypass on the D-Link DIR-882, DIR-878 and DIR-867 routers.  
CVE-2020-8863 报告了 D-Link 路由器 <$-882、<$-878 和 <$-867 上的另一个身份验证旁路。

The subsection above on CVE-2020-8864 was simplified by omitting some details about the authentication flow. Here, we describe it in detail so that we can accurately describe this authentication bypass later.  
上面关于 CVE-2020-8864 的小节通过省略有关身份验证流程的一些细节进行了简化。在这里，我们详细描述它，以便我们稍后可以准确地描述这种身份验证绕过。

These routers use the Home Network Administration Protocol (HNAP), a SOAP-based protocol for the requests on the admin panel from the client to the web server. The authentication process is as follows:  
这些路由器使用家庭网络管理协议（HNAP），这是一种基于 SOAP 的协议，用于管理面板上从客户端到 Web 服务器的请求。认证过程如下：

1.  The client sends a request message and obtains an authentication challenge from the server.  
    客户端发送请求消息，并从服务器获取身份验证质询。
2.  The server responds to the request with the values: `Challenge` and `PublicKey`.  
    服务器使用值 `Challenge` 和 `PublicKey` 响应请求。
3.  The client should combine the `PublicKey` with the password to create the `PrivateKey`. Then, use the `PrivateKey` and `Challenge` to generate a challenge response that is to be submitted as `LoginPassword` to the server.  
    客户端应将 `PublicKey` 与密码组合联合收割机以创建 `PrivateKey` 。然后，使用 `PrivateKey` 和 `Challenge` 生成一个质询响应，并将其作为 `LoginPassword` 提交给服务器。
4.  The server will perform the same computations, and if the `LoginPassword` matches, it means that the client knows the correct password, and authentication succeeds.  
    服务器将执行相同的计算，如果 `LoginPassword` 匹配，则意味着客户端知道正确的密码，并且身份验证成功。

The description above is taken from this [writeup on the ZDI blog](https://www.zerodayinitiative.com/blog/2020/9/30/the-anatomy-of-a-bug-door-dissecting-two-d-link-router-authentication-bypasses#:~:text=HNAP%20Authentication%20Process). Check it out for more details and code examples.  
上面的描述摘自 ZDI 博客上的这篇文章。查看更多细节和代码示例。

In the web server binary, it is discovered that there is a code path that checks a `PrivateLogin` field in the login request. It is as follows:  
在 Web 服务器二进制文件中，发现有一个代码路径检查登录请求中的 `PrivateLogin` 字段。具体如下：

```c
//  If PrivateLogin != NULL && PrivateLogin  == "Username"  Then Password = Username
    if ((PrivateLogin == (char *)0x0) || (iVar1 = strncmp(PrivateLogin,"Username",8), iVar1 != 0)) {
      GetPassword(Password,0x40);  // [1]
    }
    else {
      strncpy(Password,Username,0x40);  // [2]
    }
    GenPrivateKey(Challenge,Password,Publickey,PrivateKey,0x80);
```

If the submitted `PrivateLogin` field contains “Username”, then the submitted `Username` value is used as the password (\[2\]) for generating the expected `LoginPassword` value (challenge response), instead of using the user’s actual password by calling `GetPassword` (\[1\]). To put it in simple terms, the client can control the password that is used to generate the expected challenge response, therefore bypassing authentication.  
如果提交的 `PrivateLogin` 字段中包含 “”，则提交的 `Username` 值将用作生成预期 `LoginPassword` 值（质询响应）的密码（\[2\]），而不是通过调用 `GetPassword` （\[1\]）使用用户的实际密码。简而言之，客户端可以控制用于生成预期质询响应的密码，从而绕过身份验证。

It is unclear what the purpose of the `PrivateLogin` field is. As HNAP is an obsolete proprietary protocol with no documentation online, it is hard for us to determine the original purpose of this field.  
目前还不清楚 `PrivateLogin` 字段的用途。由于 HNAP 是一个过时的专有协议，没有在线文档，我们很难确定这个领域的原始目的。

As a takeaway, ensure that when implementing an authentication protocol, secrets such as passwords should not be mixed with values submitted by the client.  
作为一个要点，请确保在实现身份验证协议时，密码等秘密不应与客户端提交的值混合。

#### Summary 总结

Authentication bypass vulnerabilities on the admin panel arise from programmer mistakes, as they fail to consider edge case inputs such as empty passwords that may break the authentication logic. Besides that, there may also be flawed implementations of a protocol. Special attention should be given to reviewing the implementation of an authentication routine to catch unintended bypasses due to such mistakes.  
管理面板上的身份验证绕过漏洞源于程序员的错误，因为他们没有考虑可能破坏身份验证逻辑的边缘情况输入，例如空密码。除此之外，协议的实现也可能有缺陷。应特别注意检查身份验证例程的实现，以发现由于此类错误而导致的意外绕过。

### Command Injection 命令注入

Command injection is a commonly seen vulnerability in routers or other network and IoT devices. In this section, we discuss the vulnerable code pattern, reason behind the ubiquity of such vulnerability, guidelines to prevent them, and some examples of them in various routers in the past.  
命令注入是路由器或其他网络和物联网设备中常见的漏洞。在本节中，我们将讨论易受攻击的代码模式，这种漏洞普遍存在的原因，防止它们的指导方针，以及过去在各种路由器中的一些示例。

#### Root Cause 根本原因

Command injection is possible because a command string that contains unsanitized user input is executed as a shell command, by means of `system` or `popen` in C, `os.system` in Python, `os.execute` or `io.popen` in Lua. For example,  
命令注入是可能的，因为包含未经清理的用户输入的命令字符串是作为 shell 命令执行的，在 C 中是 `system` 或 `popen` ，在 Python 中是 `os.system` ，在 Lua 中是 `os.execute` 或 `io.popen` 。比如说，

```c
sprintf(cmd, "ping %s", ip_addr);
system(cmd);
```

```python
os.system(f"ping {ip_addr}")
```

In the examples above, an input IP address such as `127.0.0.1; reboot` will result in a command string of `ping 127.0.0.1; reboot`, which escapes the intended `ping` command and executes arbitrary commands such as `reboot`.  
在上面的示例中，输入 IP 地址（如 `127.0.0.1; reboot` ）将导致命令字符串 `ping 127.0.0.1; reboot` ，这将转义预期的 `ping` 命令并执行任意命令（如 `reboot` ）。

#### Rationale for using shell commands  
使用 shell 命令的基本原理

Running shell commands to perform various system-level operations is definitely not the norm in software development. For performance and compatibility reasons, in software running on personal computers, it is very rare to see system-level operations such as filesystem or network operations being carried out through running shell commands. However, this is almost ubiquitous in the world of embedded devices such as routers.  
运行 shell 命令来执行各种系统级操作绝对不是软件开发中的规范。出于性能和兼容性的原因，在个人计算机上运行的软件中，很少看到通过运行 shell 命令执行系统级操作（如文件系统或网络操作）。然而，这在路由器等嵌入式设备中几乎无处不在。

The reasons behind this phenomenon are somewhat acceptable. In routers, performance is not a concern because said operations are not very frequently performed. Compatibility is also not affected because programs only run on the vendor’s own devices. Without such factors in mind, it is tempting to find the simplest way to implement a required feature, and such simplest way may be flawed in security.  
这种现象背后的原因是可以接受的。在路由器中，性能不是问题，因为所述操作不是非常频繁地执行。兼容性也不受影响，因为程序只在供应商自己的设备上运行。如果没有考虑到这些因素，就很容易找到最简单的方法来实现所需的功能，而这种最简单的方法可能在安全性方面存在缺陷。

For example, look at the following function from \[NETGEAR’s `pufwUpgrade` binary\]([https://research.nccgroup.com/2024/02/09/puckungfu-2-another-netgear-wan-command-injection/#:~:text=int-,saveCfuLastFwpath,-(char%20](https://research.nccgroup.com/2024/02/09/puckungfu-2-another-netgear-wan-command-injection/#:~:text=int-,saveCfuLastFwpath,-(char%20)\*).  
例如，查看 \[NETGEAR 的 `pufwUpgrade` binary\]（https：//research.nccgroup.com/2024/02/09/puckungfu-2-another-netgear-wan-command-injection/#：~：text=int-，saveCfuLastFwpath，-（char%20\*）中的以下函数。

```c
int saveCfuLastFwpath(char *fwPath)
{
    char command [1024];
    memset(command, 0, 0x400);
    snprintf(command, 0x400, "rm %s", "/data/cfu_last_fwpath");
    system(command);
    // Command injection vulnerability
    snprintf(command, 0x400, "echo \"%s\" > %s", fwPath, "/data/cfu_last_fwpath");
    DBG_PRINT(DAT_0001620f, command);
    system(command);
    return 0;
}
```

There are proper C library functions for deleting a file ([`unlink`](https://man7.org/linux/man-pages/man2/unlink.2.html)), as well as for writing to a file ([`fopen`](https://cplusplus.com/reference/cstdio/fopen/) and [`fwrite`](https://cplusplus.com/reference/cstdio/fwrite/)). But the developers instead chose to create shell command strings starting with `rm` and `echo` and run them with `system`. Admittedly, shell commands are easier to remember and use without needing to refer back to the C documentation.  
有适当的 C 库函数用于删除文件（ `unlink` ），以及写入文件（ `fopen` 和 `fwrite` ）。但是开发人员选择创建以 `rm` 和 `echo` 开头的 shell 命令字符串，并使用 `system` 运行它们。不可否认，shell 命令更容易记忆和使用，而不需要回头参考 C 文档。

Another such example is when a router firmware developer inserts a user-entered password into a command string to calculate the password hash using `md5sum`. It takes more time and effort to write C code that achieves the same goal.  
另一个这样的示例是当路由器固件开发人员将用户输入的密码插入到命令字符串中以使用 `md5sum` 计算密码散列时。编写实现相同目标的 C 代码需要花费更多的时间和精力。

To compensate for such reduced effort, at least some device vendors do sanitize their inputs before inserting them into a command string and calling `system`. This is good. However, it just takes a small mistake, such as forgetting to sanitize an input field, to introduce a command injection vulnerability allowing RCE on the device. A vulnerability may also be introduced due to certain manipulations performed on the command string, or some unexpected reasons due to the way the command is written. For example, CVE-2024-1837 for which we will publish the advisory soon. Found by me :)  
为了补偿这种减少的工作量，至少一些设备供应商在将它们插入到命令字符串并调用 `system` 之前对其输入进行净化。这很好然而，它只需要一个小错误，例如忘记清理输入字段，就可以引入允许设备上 RCE 的命令注入漏洞。由于对命令字符串执行的某些操作，或由于命令编写方式引起的某些意外原因，也可能引入漏洞。例如，CVE-2024-1837，我们将很快发布咨询。找到了：）

From my limited exposure, I have noticed that the more expensive Cisco and ASUS routers do not take shortcuts by performing OS-level operations through shell commands, but instead they properly implement them with the corresponding API functions. If they were to execute external programs with user inputs, they only use safe functions such as `execve` that are not susceptible to command injection. With such efforts, they eradicate even the smallest possibility of command injection on the admin panel.  
从我有限的接触，我注意到，更昂贵的思科和华硕路由器不走捷径，通过 shell 命令执行操作系统级的操作，而是正确地实现了相应的 API 功能。如果它们要执行带有用户输入的外部程序，它们只使用安全的函数，如 `execve` ，这些函数不容易受到命令注入的影响。通过这样的努力，他们甚至消除了管理面板上命令注入的最小可能性。

In the next section, we share some suggestions to prevent command injection in the event where external programs need to be executed.  
在下一节中，我们将分享一些建议，以防止在需要执行外部程序的情况下发生命令注入。

#### Prevention 预防

In this subsection, we share secure code design guidelines to prevent command injection. First of all, as mentioned repeatedly in the subsections above, not all operations need to be performed through shell commands, so please avoid doing so unless necessary.  
在本小节中，我们将分享安全代码设计指南，以防止命令注入。首先，正如上面的小节中反复提到的，并非所有操作都需要通过 shell 命令执行，因此除非必要，请避免这样做。

##### Avoid `system` commands 避免 `system` 命令

The decision to run external programs using a shell command is the cause for potential command injection bugs. On top of just recommending developers to not write such code, security teams could help them avoid the usage of functions that run shell commands by raising warnings where such functions are called.  
使用 shell 命令运行外部程序的决定是潜在的命令注入错误的原因。除了建议开发人员不要编写此类代码之外，安全团队还可以通过在调用此类函数时发出警告来帮助他们避免使用运行 shell 命令的函数。

We list below such functions that should be avoided from the codebase. Note that the list is not exhaustive. Security teams should check if there are other such functions supported by the standard library or any third party libraries imported by the codebase.  
我们在下面列出了应该从代码库中避免的函数。请注意，该列表并非详尽无遗。安全团队应检查标准库或代码库导入的任何第三方库是否支持其他此类函数。

-   C: `system`, `popen` C： `system` 、 `popen`
-   Python: `os.system`, `subprocess.Popen`/`subprocess.call`/`subprocess.run` with `shell=True` argument  
    Python： `os.system` 、 `subprocess.Popen` / `subprocess.call` / `subprocess.run` ，带 `shell=True` 参数

Do not worry about whether such a rule may break compatibility, because it won’t. We provide the safe alternative below, which can do the same things that a shell command can do.  
不要担心这样的规则是否会破坏兼容性，因为它不会。我们在下面提供了一个安全的替代方案，它可以做 shell 命令可以做的事情。

##### Run executable with argument list  
使用参数列表运行可执行文件

There is a safe way to run external scripts or binaries, by specifying a program and providing an argument list, by using `execve` in C or `subprocess.Popen`/`subprocess.run`/`subprocess.call` (without the `shell=True` argument) in Python. For example,  
有一种安全的方式来运行外部脚本或二进制文件，通过指定一个程序并提供一个参数列表，在 C 中使用 `execve` 或在 Python 中使用 `subprocess.Popen` / `subprocess.run` / `subprocess.call` （没有 `shell=True` 参数）。比如说，

```c
execve("/bin/ping", { "/bin/ping", ip_addr, 0 }, 0);
```

```python
subprocess.Popen(["/bin/ping", ip_addr])
```

This eliminates the possibility of command injection on the command directly. However, beware that this does not make any guarantees about whether there will be command injection in the target executable, for example `/bin/ping` as in the example above.  
这消除了直接在命令上注入命令的可能性。但是，请注意，这并不能保证目标可执行文件中是否会有命令注入，例如上面示例中的 `/bin/ping` 。

Note that in C, `execve` replaces the current running process with the target executable. That is, if `execve` is called with `/bin/ping` by the admin panel server, the whole service will be gone, replaced by `ping`. This is certainly not the intended behaviour. **Remember to `fork` the process before calling `execve`.**  
请注意，在 C 中， `execve` 将当前正在运行的进程替换为目标可执行文件。也就是说，如果管理面板服务器使用 `/bin/ping` 调用 `execve` ，则整个服务将消失，取而代之的是 `ping` 。这当然不是预期的行为。请记住在调用 `execve` 之前先执行 `fork` 过程。

However, watch out for code as in the example below. It defeats the purpose since it runs a shell command again.  
但是，请注意下面示例中的代码。它违背了目的，因为它再次运行 shell 命令。

```c
sprintf(cmd, "ping %s", ip_addr);
execve("/bin/sh", { "/bin/sh", "-c", cmd, 0 }, 0);
```

##### Custom `execve` 自定义 `execve`

In languages such as Lua, there may not be a library function such as `execve` for running a specific program with an argument list. In such unfortunate scenario, there is no choice but to use the `system` or `popen` equivalent that is available in this language. In Lua, that is `os.execute`.  
在 Lua 等语言中，可能没有像 `execve` 这样的库函数来运行带有参数列表的特定程序。在这种不幸的情况下，除了使用这种语言中可用的 `system` 或 `popen` 等效项之外别无选择。在 Lua 中，这是 `os.execute` 。

To protect the developers from crafting command strings prone to command injection, the development team may create a function similar to `execve` that takes in an executable path and argument list, then crafts the command string with these values, and passes it to `system` for execution. The executable path can be concatenated together with all the arguments, but there are two important things to take note:  
为了防止开发人员制作易于命令注入的命令字符串，开发团队可以创建类似于 `execve` 的函数，该函数接受可执行路径和参数列表，然后用这些值制作命令字符串，并将其传递给 `system` 以供执行。可执行路径可以与所有参数连接在一起，但有两件重要的事情需要注意：

**Wrap every argument with single quotes.** This is to prevent command substitution, because in a shell command, contents within single quotes will be passed verbatim as an argument. With single quotes, any sequence of characters in the form `$(...)` or `` `...` `` will not be evaluated. Do not wrap the arguments in double quotes. Command substitution will still apply for contents within double quotes.  
将所有参数用单引号括起来。这是为了防止命令替换，因为在 shell 命令中，单引号内的内容将作为参数逐字传递。使用单引号时，格式为 `$(...)` 或 `` `...` `` 的任何字符序列都不会被计算。不要用双引号将参数括起来。命令替换仍然适用于双引号内的内容。

**Escape the single quotes in every argument.** If an argument contains single quotes, it will close the opening single quote before it, and any command substitution payload after it will be evaluated. Make sure all single quotes in every argument are escaped by prepending them with a backslash character, so that they do not close the opening single quote before the argument.  
在每个参数中转义单引号。如果一个参数包含单引号，它将关闭它之前的开始单引号，并且它之后的任何命令替换有效负载都将被计算。确保每个参数中的所有单引号都通过在它们前面加上一个反斜杠字符进行转义，这样它们就不会在参数之前关闭开始的单引号。

The example below demonstrates how the operations above could be implemented in Lua.  
下面的例子演示了如何在 Lua 中实现上述操作。

```lua
function custom_execute(executable_path, args)
    -- Escape single quotes within arguments
    local escape_single_quotes = function(str)
        return string.gsub(str, "'", "\\'")
    end

    -- Quote and escape each argument
    local quoted_args = {}
    for _, arg in ipairs(args) do
        table.insert(quoted_args, "'" .. escape_single_quotes(tostring(arg)) .. "'")
    end

    -- Concatenate executable path and quoted arguments
    local command = executable_path .. " " .. table.concat(quoted_args, " ")

    -- Execute the command using os.execute
    os.execute(command)
end

-- Example usage
local echo_path = "/bin/echo"
local args = {"hello", "world", "hey"}
custom_execute(echo_path, args)
```

The `custom_execute` function above removes the possibility of command injection, regardless of any user input that may be part of the argument list. Such a function gives developers a peace of mind when it is necessary to execute external programs, and removes the burden from them to consider any sanitization that is required for the user input.  
上面的 `custom_execute` 函数消除了命令注入的可能性，而不管任何用户输入可能是参数列表的一部分。当需要执行外部程序时，这样的功能使开发人员安心，并消除了他们考虑用户输入所需的任何清理的负担。

##### Avoid `eval` in shell scripts  
避免在 shell 脚本中使用 `eval`

The suggestions above are applicable to services that receive input from a client request and use this input value in the execution of another program on the system. The protections above ensure that handlers for client requests are safe from command injection. However, they do not make any guarantees about the safety of the external program that is executed.  
上述建议适用于从客户端请求接收输入并在系统上执行另一个程序时使用此输入值的服务。上述保护措施可确保客户端请求的处理程序不会受到命令注入的影响。但是，它们不保证执行的外部程序的安全性。

Consider the following example where /usr/sbin/custom\_script is a shell script that is given a user input value as an argument. There is no command injection in executing the script. However, there could be command injection within the script that is being executed.  
考虑下面的示例，其中 /usr/sbin/custom\_script 是一个 shell 脚本，它被赋予一个用户输入值作为参数。在执行脚本时没有命令注入。但是，在正在执行的脚本中可能存在命令注入。

```c
execve("/usr/sbin/custom_script", { "/usr/sbin/custom_script", user_input, 0 }, 0);
```

Consider the following shell script that inserts an argument (`$1`) into a command string and executes it in various ways.  
考虑下面的 shell 脚本，它将一个参数（ `$1` ）插入到命令字符串中，并以各种方式执行它。

1.  Using `eval`. 使用 `eval` 。
2.  Using `$(...)` (command substitution).  
    使用 `$(...)` （命令替换）。
3.  Using `` `...` `` (command substitution).  
    使用 `` `...` `` （命令替换）。

```bash
#!/bin/sh
cmd="echo $1"

files=`eval "$cmd"`
echo $files

files=$($cmd)
echo $files

files=`$cmd`
echo $files
```

The output of the script above when executed with `aaa;whoami` as an argument is as follows.  
当使用 `aaa;whoami` 作为参数执行上述脚本时，输出如下。

```bash
$ ./script 'aaa;whoami'
aaa user        // command injection occured
aaa;whoami
aaa;whoami
```

Notice that when command substitution (2nd and 3rd example) is performed, there is no command injection. This is because the argument `$1` is passed as an argument to the `echo` program as written in `cmd`. This means that the whole argument string containing `aaa;whoami` is passed to `echo` as an individual argument.  
请注意，当执行命令替换（第 2 和第 3 个示例）时，没有命令注入。这是因为参数 `$1` 作为参数传递给了 `echo` 程序，就像在 `cmd` 中写的那样。这意味着包含 `aaa;whoami` 的整个参数字符串作为单个参数传递给 `echo` 。

On the other hand, in the case of `eval`, `$1` is interpolated, that is, expanded as a string and inserted into the command string, resulting in `echo aaa;whoami` being the command that is executed. Command injection is present in this case, as seen in the output attached above.  
另一方面，在 `eval` 的情况下， `$1` 被内插，即，被扩展为字符串并被插入到命令字符串中，导致 `echo aaa;whoami` 成为被执行的命令。在这种情况下会出现命令注入，如上面所附的输出中所示。

Therefore, **avoid the usage of `eval` in shell scripts**, to prevent any potential command injection vulnerabilities due to mishandling of arguments which may come from user input.  
因此，请避免在 shell 脚本中使用 `eval` ，以防止由于错误处理可能来自用户输入的参数而导致的任何潜在命令注入漏洞。

#### Actionable Steps 可操作的步骤

To summarize the suggestions above, we recommend development and security teams to impose the following rules on their codebase:  
总结以上建议，我们建议开发和安全团队在他们的代码库中实施以下规则：

1.  Avoid dangerous functions that execute a command string directly, e.g. `system`, `popen`, `eval`. Only allow the execution of an external program by passing an argument list.  
    避免直接执行命令字符串的危险函数，例如 `system` ， `popen` ， `eval` 。只允许通过传递参数列表执行外部程序。
2.  A thorough review should be conducted to decide whether a function is safe or necessary.  
    应进行彻底的审查，以确定某项功能是否安全或必要。
3.  If an unsafe function is necessary (such as `os.execute` in Lua), use custom wrapper functions that call the function in a safe manner. For example, the `custom_execute` wrapper for `os.execute` in Lua shown above.  
    如果需要一个不安全的函数（比如 Lua 中的 `os.execute` ），使用自定义的包装函数，以安全的方式调用该函数。例如，上面显示的 Lua 中 `os.execute` 的 `custom_execute` 包装器。

#### Examples 示例

In the following sub-sections, we show examples of command injection in various routers, in implementations using different programming languages, in various services, to show that this vulnerability can manifest itself under different contexts.  
在下面的小节中，我们将展示在各种路由器中、在使用不同编程语言的实现中以及在各种服务中的命令注入示例，以说明此漏洞可以在不同的上下文中表现出来。

##### D-Link (C) D-Link（C）

In 2021, I discovered some command injection vulnerabilities in the [DIR-1960/1360/2660/3060](https://supportannouncement.us.dlink.com/announcement/publication.aspx?name=SAP10377) and [DIR-X1560](https://supportannouncement.us.dlink.com/announcement/publication.aspx?name=SAP10249) devices. The vulnerabilities were found in code that handles web requests from the admin panel.  
在 2021 年，我在 DIR-1960/1360/2660/3060 和 DIR-X1560 设备中发现了一些命令注入漏洞。这些漏洞存在于处理来自管理面板的 Web 请求的代码中。

Some were due to complete lack of sanitization of user input, inserting them in command strings to run programs like `sendmail`, `smbpasswd` and `iptables`. Such code is written as operations related to email or SMB can be rather complicated to implement, and a simpler solution would be to use the relevant programs that are already present on the system. The code for running such commands with user input had insufficient or no sanitization performed on the input values, resulting in command injection attacks being possible through the corresponding web endpoints.  
有些是由于完全缺乏对用户输入的清理，将它们插入命令字符串以运行 `sendmail` ， `smbpasswd` 和 `iptables` 等程序。编写此类代码是因为与电子邮件或 SMB 相关的操作实现起来可能相当复杂，而更简单的解决方案是使用系统上已经存在的相关程序。使用用户输入运行此类命令的代码对输入值执行的清理不足或没有执行清理，导致可能通过相应的 Web 端点进行命令注入攻击。

###### Failed validation of IP range string  
IP 范围字符串验证失败

There was a rather interesting case of flawed input validation done before inserting a user input string into an `iptables` command string. The user input value is an IP address range, e.g. `123.123.123.123/24`. The handler function did check if the user input follows the `a.b.c.d/subnet` format, but not correctly.  
在将用户输入字符串插入到 `iptables` 命令字符串之前，有一个相当有趣的错误输入验证案例。用户输入值是 IP 地址范围，例如 `123.123.123.123/24` 。handler 函数确实检查了用户输入是否遵循 `a.b.c.d/subnet` 格式，但不正确。

1.  It calls the [`inet_addr`](https://linux.die.net/man/3/inet_addr) C function to verify that the front part (`a.b.c.d`) is a valid IP address.  
    它调用 `inet_addr` C 函数来验证前面的部分（ `a.b.c.d` ）是有效的 IP 地址。
2.  Then, it calls [`strtol`](https://linux.die.net/man/3/strtol)on the part after the `/` (`subnet`) to check if it is a positive number.  
    然后，它在 `/` （ `subnet` ）之后的部分上调用 `strtol` ，以检查它是否为正数。
3.  On first glance, this is useful because a string that starts with alphabets or symbols will result in 0 being returned.  
    乍一看，这很有用，因为以字母或符号开头的字符串将返回 0。
4.  However, a string like `16 abc` will let `strtol` return 16 which is considered valid.  
    然而，像 `16 abc` 这样的字符串会让 `strtol` 返回 16，这被认为是有效的。

As a result, a user input like `123.123.123.123/16 $(reboot)` will successfully perform command injection when it is inserted into an `iptables` command string.  
因此，像 `123.123.123.123/16 $(reboot)` 这样的用户输入将在插入到 `iptables` 命令字符串中时成功执行命令注入。

This is an example of failed validation that could be potentially caused by an incomplete understanding of how library functions such as `strtol` works.  
这是一个验证失败的例子，可能是由于对库函数（如 `strtol` ）的工作方式不完全理解而导致的。

##### Zyxel (Python) Zyxel（Python）

Now, let’s look at the Zyxel NAS whose web management interface runs on Python. This is not a router, but serves as a good case study.  
现在，让我们看看合勤 NAS，其 Web 管理界面运行在 Python 上。这不是一个路由器，但作为一个很好的案例研究。

[CVE-2023-4473](https://securityintelligence.com/x-force/ibm-identifies-zero-day-vulnerability-zyxel-nas-devices/) was reported by IBM X-Force regarding an OS command injection in the following form:  
CVE-2023-4473 由 IBM X-Force 报告，涉及以下形式的操作系统命令注入：

```python
mail_hour = pyconf.get_conf_value(MAINTENANCE_LOG_MAIL, 'hour')
mail_minute = pyconf.get_conf_value(MAINTENANCE_LOG_MAIL, 'miniute')
cmd = '/usr/sbin/zylog_config mail 1 schedule daily hour %s minute %s' % (mail_hour, mail_minute)
os.system(cmd)
```

The values `mail_hour` and `mail_minute` are obtained from user input as provided in the POST request below.  
值 `mail_hour` 和 `mail_minute` 从用户输入中获得，如下面的 POST 请求中所提供的。

```plain
curl -s -X POST \
  –data-binary 'schedulePeriod=daily&scheduleHour=0&scheduleMinute=0%60cmd60' \
  'http://10.20.17.122/cmd,/ck6fup6/zylog_main/configure_mail_syslog'
```

###### Fix

**`os.system` should never ever be used with a string that contains user input.** As suggested in the earlier section, use `subprocess.Popen` instead, by providing the executable path and its arguments as an argument list. Doing so removes the possibility of command injection because the command string is no longer executed as a shell command. For example:  
`os.system` 永远不应该与包含用户输入的字符串一起使用。如前一节所建议的，使用 `subprocess.Popen` ，通过提供可执行路径及其参数作为参数列表。这样做可以消除命令注入的可能性，因为命令字符串不再作为 shell 命令执行。举例来说：

```python
cmd = '/usr/sbin/zylog_config mail 1 schedule daily hour %s minute %s' % (mail_hour, mail_minute)
subprocess.Popen(cmd.split(" "))
```

##### TP-Link (LuCI) TP-Link（LuCI）

TP-Link’s routers use a fork of OpenWrt’s LuCI, a configuration interface based on Lua, as the backend for their admin panel.  
TP-Link 的路由器使用 OpenCart 的 LuCI 的一个分支，一个基于 Lua 的配置接口，作为其管理面板的后端。

Their Lua source code is compiled to bytecode and stored on the router, and the code is invoked according to the requests made by the client. Any security researcher who is interested in analyzing the admin panel backend code would need to decompile the Lua bytecode. Decompiling Lua bytecode is not as difficult a task as decompiling binaries compiled from C, as bytecode-based languages such Lua, Python, or Java contain more information that make them easier to decompile. There is an open source Lua decompiler [luadec](https://github.com/viruscamp/luadec).  
他们的 Lua 源代码被编译成字节码并存储在路由器上，并根据客户端发出的请求调用代码。任何对分析管理面板后端代码感兴趣的安全研究人员都需要反编译 Lua 字节码。反编译 Lua 字节码并不像反编译从 C 编译的二进制文件那样困难，因为基于字节码的语言（如 Lua，Python 或 Java）包含更多的信息，使它们更容易反编译。有一个开源的 Lua 反编译器 luadec。

However, as mentioned, TP-Link uses a fork of LuCI, and they made some changes to it, including the underlying Lua compiler. According to this article [Unscrambling Lua](https://vovohelo.medium.com/unscrambling-lua-7bccb3d5660), TP-Link changed the bytecode opcodes emitted by its Lua compiler so that luadec would not work properly. One would have to reverse engineer the changes made, and apply the same changes to luadec, to have a working decompiler for TP-Link’s LuCI’s bytecode. There is another open source project [luadec-tplink by superkhung](https://github.com/superkhung/luadec-tplink) which is not perfect, but works somewhat okay for reverse engineering TP-Link’s request handlers stored in bytecode form.  
然而，如上所述，TP-Link 使用 LuCI 的一个分支，他们对它做了一些修改，包括底层的 Lua 编译器。根据这篇文章解读 Lua，TP-Link 改变了 Lua 编译器发出的字节码操作码，使得 luadec 无法正常工作。人们必须对所做的更改进行逆向工程，并将相同的更改应用于 luadec，以获得 TP-Link 的 LuCI 字节码的工作反编译器。还有另一个由 superkhung 开发的开源项目 luadec-tplink，它并不完美，但对于以字节码形式存储的 TP-Link 的请求处理程序进行逆向工程，它可以工作得很好。

[ZDI-23-451/CVE-2023-1389](https://www.zerodayinitiative.com/advisories/ZDI-23-451/) reports an unauthenticated command injection that could be exploited by attackers on the LAN interface to gain RCE. The vulnerable endpoint is responsible for setting the admin panel’s locale, in particular through the country parameter.  
ZDI-23-451/CVE-2023-1389 报告了一个未经验证的命令注入，攻击者可以在 LAN 接口上利用该命令注入来获取 RCE。易受攻击的端点负责设置管理面板的区域设置，特别是通过国家参数。

```http
POST /cgi-bin/luci/;stok=/locale?form=country HTTP/1.1
Host: <target router>
Content-Type: application/x-www-form-urlencoded

operation=write&country=$(id>/tmp/out)
```

An RCE can be gained from a request to change the country setting without requiring authentication.  
RCE 可以从更改国家设置的请求中获得，而无需身份验证。

The situation is made worse by [ZDI-23-452/CVE-2023-27359](https://www.zerodayinitiative.com/advisories/ZDI-23-452/) which is a race condition vulnerability in the firewall service `hotplugd`, that allows an attacker to access the vulnerable endpoint above through the WAN.  
ZDI-23-452/CVE-2023-27359 是防火墙服务 `hotplugd` 中的竞争条件漏洞，使情况变得更糟，该漏洞允许攻击者通过 WAN 访问上述易受攻击的端点。

###### Suggestions 建议

Although we do not have the code, it is very likely that the `country` parameter was inserted into a command string, then passed to either `os.execute` or `io.popen`, resulting in a command injection vulnerability. As recommended in the earlier section, it is best to create a wrapper for `os.execute` that wraps all arguments with single quotes and escapes all single quotes within them.  
虽然我们没有代码，但很可能是将 `country` 参数插入到命令字符串中，然后传递给 `os.execute` 或 `io.popen` ，从而导致命令注入漏洞。如前所述，最好为 `os.execute` 创建一个包装器，用单引号包装所有参数，并转义其中的所有单引号。

On this vulnerable router, LuCI may be running as `root` (according to the linked Tenable advisory), allowing any injected commands to be executed as `root`. As described in [Services Running as `root`](#Services-Running-as-root) above, it is a poor practice to run any service with `root` privileges, as that would mean a compromise of the whole device once that service is exploited.  
在这个易受攻击的路由器上，LuCI 可能以 `root` 运行（根据链接的 Tenable 建议），允许任何注入的命令以 `root` 执行。如上面的服务运行 `root` 中所述，使用 `root` 权限运行任何服务都是不好的做法，因为一旦该服务被利用，这将意味着整个设备的危害。

###### Remarks 言论

TP-Link may obfuscate their Lua bytecode to prevent others from reverse engineering their code. This adds extra work for not only threat actors, but also security researchers in detecting vulnerabilities in their devices. It may form an illusion that these devices are secure when in reality there were just very few people who spent time inspecting these devices.  
TP-Link 可能会混淆他们的 Lua 字节码，以防止其他人对其代码进行逆向工程。这不仅为威胁行为者增加了额外的工作，还为安全研究人员检测其设备中的漏洞增加了额外的工作。这可能会形成一种错觉，即这些设备是安全的，而实际上只有很少的人花时间检查这些设备。

##### DHCP server (C) DHCP 服务器（C）

Command injection is not limited to just the admin panel. Earlier, our team discovered a command injection vulnerability in the DHCP server of the NETGEAR RAX30 as shared in [this writeup](https://starlabs.sg/blog/2022/12-the-last-breath-of-our-netgear-rax30-bugs-a-tragic-tale-before-pwn2own-toronto-2022/).  
命令注入不仅限于管理面板。早些时候，我们的团队在 NETGEAR RAX30 的 DHCP 服务器中发现了一个命令注入漏洞。

The vulnerable code is as follows:  
易受攻击的代码如下：

```c
int __fastcall send_lease_info(int a1, dhcpOfferedAddr *lease)
{
// truncated...
  if ( !a1 )
  {
// truncated ...
    if ( body.hostName[0] )
    {
      strncpy((char *)HostName, body.hostName, 0x40u); // [1]
      snprintf((char *)v11, 0x102u, "%s", body.vendorid);
    }
    else
    {
      strncpy((char *)v10, "unknown", 0x40u);
      strncpy((char *)v11, "dhcpVendorid", 0x102u);
    }
    sprintf(
      command,
      "pudil -a %s %s %s %s \"%s\"",
      body.macAddr,
      body.ipAddr,
      (const char *)HostName,  // [2]
      body.option55,
      (const char *)v11);
    system(command);   // [3]
  }
//...
}
```

At `[1]`, the `hostName` parameter of the DHCP request body is copied into `HostName`, then inserted into `command` at `[2]`, and executed by `system` at `[3]`. There is no sanitization done on this user input.  
在 `[1]` 处，DHCP 请求体的 `hostName` 参数被复制到 `HostName` 中，然后在 `[2]` 处插入到 `command` 中，并在 `[3]` 处由 `system` 执行。没有对该用户输入进行清理。

###### Patch 贴片

The vulnerability was fixed by calling `execve` instead of `system` to execute the command.  
通过调用 `execve` 而不是 `system` 来执行命令，修复了该漏洞。

### Buffer Overflow 缓冲区溢出

Buffer overflow is a common issue for programs written in the C programming language, and most services in routers are written in C. Buffer overflow vulnerabilities could be exploited by an attacker to gain RCE on the underlying device.  
缓冲区溢出是用 C 语言编写的程序的常见问题，路由器中的大多数服务都是用 C 语言编写的。攻击者可以利用缓冲区溢出漏洞在底层设备上获得 RCE。

Over the years, buffer overflow has gotten increasingly difficult to exploit due to mitigations such as ASLR, PIE, RELRO, stack canary and NX. ASLR is enforced by the OS, while PIE, RELRO, stack canary and NX are protections added by the compiler by default. However, in the recent years, many routers are still observed to lack such mitigations. This could be due to the vendors’ still using very old versions of GCC (or other compilers) in their deployment process, which could be missing the ability to add said mitigations to the resulting binary.  
多年来，由于 ASLR、PIE、STARRO、堆栈金丝雀和 NX 等缓解措施，缓冲区溢出越来越难以利用。ASLR 是由操作系统强制执行的，而 PIE、STARRO、stack canary 和 NX 是由编译器默认添加的保护。然而，近年来，仍然观察到许多路由器缺乏这样的缓解措施。这可能是由于供应商在其部署过程中仍然使用非常旧的 GCC 版本（或其他编译器），这可能无法将上述缓解措施添加到生成的二进制文件中。

With the mitigations listed above, attempts to exploit a buffer overflow vulnerability could be prevented most of the time, unless the vulnerability satisfies conditions that are favorable for exploitation. However, in successfully preventing the exploitation attempt, the mitigations will halt the running service because its internal state has been corrupted. This results in a DoS which is also not desirable. Hence, knowing that the mitigations do not guarantee full protection against all exploitation attempts, it is still most preferable that buffer overflow vulnerabilities are avoided through secure coding practices and design, which we will discuss in detail in this section.  
使用上面列出的缓解措施，可以在大多数情况下防止利用缓冲区溢出漏洞的尝试，除非该漏洞满足有利于利用的条件。但是，在成功阻止利用漏洞的尝试时，缓解措施将停止正在运行的服务，因为其内部状态已损坏。这会导致 DoS，这也是不希望的。因此，尽管缓解措施不能保证完全防止所有攻击尝试，但最好还是通过安全的编码实践和设计来避免缓冲区溢出漏洞，我们将在本节中详细讨论。

#### Root Cause 根本原因

The root cause of a buffer overflow bug is straightforward. A buffer is allocated a certain size, but data longer than that size is written into the buffer. As a result, other values in adjacent memory are corrupted with user-controlled values.  
缓冲区溢出错误的根本原因很简单。缓冲区被分配了一定的大小，但是比该大小更长的数据被写入缓冲区。因此，相邻内存中的其他值会被用户控制的值损坏。

In routers, this mistake is commonly observed in the usage of functions that copy memory contents. We list examples in the code snippet below. For the examples below, suppose that `websGet` is a function that takes in a key and returns the corresponding value in the query string of an incoming web request of the router admin panel.  
在路由器中，这种错误通常在复制内存内容的函数的使用中观察到。我们在下面的代码片段中列出了示例。对于下面的示例，假设 `websGet` 是一个函数，它接受一个键并返回路由器管理面板传入 Web 请求的查询字符串中的相应值。

```c
char* contents = websGet("contents");
int size = websGet("size");

char buffer[128];

strcpy(buffer, contents);
strncpy(buffer, contents, size);

sprintf(buffer, "%s", contents);
snprintf(buffer, "%s", contents);

buffer[0] = 0;
strcat(buffer, contents);
strncat(buffer, content, size);

memcpy(buffer, contents, size);
```

In the code snippet above, `strcpy`, `sprintf` and `strcat` copy the whole user-supplied `contents` string into `buffer` without restricting the length. If the length of `contents` exceeds the size allocated for `buffer` (i.e. 128), buffer overflow occurs.  
在上面的代码片段中， `strcpy` 、 `sprintf` 和 `strcat` 将用户提供的整个 `contents` 字符串复制到 `buffer` 中，而不限制长度。如果 `contents` 的长度超过分配给 `buffer` 的大小（即 128），则发生缓冲区溢出。

The example above also assumes a scenario in which the user also specifies the `size` of `contents` to copy through `strncpy`, `snprintf`, `strncat` and `memcpy`. Similarly, if `size` exceeds 128, buffer overflow occurs.  
上面的示例还假设用户还指定 `contents` 中的 `size` 以通过 `strncpy` 、 `snprintf` 、 `strncat` 和 `memcpy` 进行复制的场景。类似地，如果 `size` 超过 128，则发生缓冲区溢出。

Aside from functions that perform copying, code for manually copying memory contents can also be vulnerable to buffer overflow, as shown in the example below, in which the `size` field is user-supplied and could be greater than the size allocated for `buffer`.  
除了执行复制的函数之外，手动复制内存内容的代码也容易受到缓冲区溢出的攻击，如下例所示，其中 `size` 字段是用户提供的，可能大于为 `buffer` 分配的大小。

```c
for (int i = 0; i < size; ++i) buffer[i] = contents[i];
```

All of the vulnerable code examples above share a common theme. They do not restrict the size of the contents being copied/written. Implicitly, they had allowed the user (attacker) to choose the size. To prevent buffer overflow, do not use functions that do not restrict the length, e.g. `strcpy`, `sprintf` and `strcat`. Instead, use `strncpy`, `snprintf`, `strncat` or `memcpy`, and make sure that the length argument is not a user-controlled value.  
上面所有易受攻击的代码示例都有一个共同的主题。它们不限制正在复制 / 写入的内容的大小。这意味着，他们允许用户（攻击者）选择大小。为了防止缓冲区溢出，请不要使用不限制长度的函数，例如 `strcpy` 、 `sprintf` 和 `strcat` 。相反，使用 `strncpy` 、 `snprintf` 、 `strncat` 或 `memcpy` ，并确保长度参数不是用户控制的值。

The examples above are contrived, as they are just intended for demonstrating the insecure code patterns. In a more complex codebase, a user-submitted value may be stored and retrieved and manipulated repeatedly, at various locations in the code, before finally being used in the copying operation. Under such situations, it can be difficult to ensure that all such memory-writing operations are immune to buffer overflow. We provide suggestions below in the form of secure design and practices to systematically protect your code against buffer overflow bugs.  
上面的例子是人为的，因为它们只是为了演示不安全的代码模式。在更复杂的代码库中，用户提交的值可以在代码中的各个位置重复地存储、检索和操纵，然后最终用于复制操作。在这样的情况下，可能难以确保所有这样的存储器写入操作不受缓冲区溢出的影响。我们在下面以安全设计和实践的形式提供建议，以系统地保护您的代码免受缓冲区溢出错误的影响。

#### Prevention 预防

Buffer overflow bugs can be considered as caused by human mistakes. The complexity of a codebase increases the likelihood of the occurrence of such mistakes. Through careful design and restrictions imposed on the codebase, we can do our best to protect developers against unintentionally introducing buffer overflow bugs into the program.  
缓冲区溢出错误可以被认为是由人为错误引起的。代码库的复杂性增加了发生此类错误的可能性。通过仔细的设计和对代码库的限制，我们可以尽最大努力保护开发人员避免无意中将缓冲区溢出错误引入程序。

##### Use bounded functions for copying  
使用有界函数进行复制

The usage of memory-copying functions without a length limit such as `strcpy`, `strcat` and `sprintf` should not be allowed in the codebase. Instead, use the bounded alternatives such as `strncpy`, `strncat`, `snprintf` or `memcpy`. There is no imaginable scenario where the unbounded functions (e.g. `strcpy`) will be more useful than their bounded alternatives (e.g. `strncpy`).  
不允许在代码库中使用没有长度限制的内存复制函数，例如 `strcpy` 、 `strcat` 和 `sprintf` 。相反，使用有界的替代方案，如 `strncpy` ， `strncat` ， `snprintf` 或 `memcpy` 。没有任何可以想象的场景，其中无界函数（例如 `strcpy` ）将比它们的有界替代品（例如 `strncpy` ）更有用。

Be mindful that when using functions like `strncpy` or `memcpy`, do not use `strlen` to determine the length to copy, as listed in the example below, as this is no different from just calling `strcpy`. The length argument should be independent of the user input, but based on the allocated size of the destination buffer instead.  
请注意，当使用像 `strncpy` 或 `memcpy` 这样的函数时，不要使用 `strlen` 来确定要复制的长度，如下面的示例所示，因为这与调用 `strcpy` 没有什么不同。长度参数应该独立于用户输入，而是基于目标缓冲区的分配大小。

```c
char buffer[128];
// bad
strncpy(buffer, contents, strlen(contents));
// good
strncpy(buffer, contents, 128);
strncpy(buffer, contents, sizeof(buffer));
```

##### Pass buffer size as function argument  
将缓冲区大小作为函数参数传递

Even when developers put in conscious effort to ensure that memory is only copied into within a buffer’s bounds, there is another challenge: it is difficult to know what exactly is the size allocated for a buffer. The following example illustrates this problem.  
即使开发人员有意识地努力确保内存只在缓冲区的范围内复制，也存在另一个挑战：很难知道分配给缓冲区的确切大小。下面的例子说明了这个问题。

```c
void get_name(char* buf)
{
  char* name = websGet("name");
  memcpy(buf, name, ???);
}

void f2(char* buf) { get_name(buf); }
void f3(char* buf) { f2(buf); }

void store_input() {
  char* buf = (char*) malloc(64);
  f3(buf);
}
```

In the example above, `store_input` allocates 64 bytes for `buf`. Then, `buf` is passed through `f3` then `f2` to `get_name` which copies a user-provided string into it. In `get_name`, it is no longer obvious what was the size allocated for `buf`. The developer would need to find references to `get_name`, see that it is called by `f2`, then again find references to `f2`, see that it is called by `f3`, then finally get to `store_input` and learn that the allocated size is 64. If there are way more functions calling `get_name`, `f2` or `f3`, this would be madness. The developer would have to make sure that the length specified in `get_name` is safe for all its callers.  
在上面的例子中， `store_input` 为 `buf` 分配了 64 个字节。然后， `buf` 通过 `f3` ，然后 `f2` 到 `get_name` ，这将复制用户提供的字符串。在 `get_name` 中，分配给 `buf` 的大小不再明显。开发人员需要找到对 `get_name` 的引用，看到它被 `f2` 调用，然后再次找到对 `f2` 的引用，看到它被 `f3` 调用，最后到达 `store_input` 并了解分配的大小是 64。如果有更多的函数调用 `get_name` ， `f2` 或 `f3` ，这将是疯狂的。开发人员必须确保 `get_name` 中指定的长度对所有调用者都是安全的。

Furthermore, any changes made to the allocation size in `store_input` also has to be made to `get_name`. If the developer modifying `store_input` was not aware of `get_name`, he will miss this out, resulting in `get_name` calling `memcpy` with the wrong length. In general, it is bad practice to have values serving the same purpose hardcoded in different locations.  
此外，对 `store_input` 中的分配大小进行的任何更改也必须对 `get_name` 进行更改。如果开发人员修改 `store_input` 时没有意识到 `get_name` ，他将错过这一点，导致 `get_name` 以错误的长度调用 `memcpy` 。通常，在不同的位置硬编码用于相同目的的值是不好的做法。

One may consider having a global size constant `SIZE` that is used by the allocation in `store_input` and `memcpy` in `get_name`. This works well, if `get_name` is ever only given a buffer that is allocated `SIZE` bytes. This may be the case in the short term. However, could this still hold 2 years later if most of the development team has changed? For example, a new developer may decide to call `get_name` with a buffer of a smaller size, without being aware of the `memcpy` length.  
可以考虑具有全局大小常数 `SIZE` ，其由 `store_input` 中的分配和 `get_name` 中的 `memcpy` 使用。如果 `get_name` 只被分配了一个缓冲区，而这个缓冲区被分配了 `SIZE` 个字节，那么这个方法就可以很好地工作。短期内可能是这样。然而，如果开发团队的大部分成员都发生了变化，两年后这一点还能成立吗？例如，一个新的开发人员可能决定用一个较小的缓冲区来调用 `get_name` ，而不知道 `memcpy` 的长度。

We suggest designing a codebase that is resilient to the changes above, that is, by passing the destination buffer’s allocated size as a function argument. The following code snippet shows how this can be applied on the example above.  
我们建议设计一个能够适应上述变化的代码库，也就是说，通过将目标缓冲区的分配大小作为函数参数传递。下面的代码片段显示了如何将其应用于上面的示例。

```c
void get_name(char* buf, size_t size)
{
  char* name = websGet("name");
  memcpy(buf, name, size);
}

void f2(char* buf, size_t size) { get_name(buf, size); }
void f3(char* buf, size_t size) { f2(buf, size); }

void store_input() {
  size_t size = 64;
  char* buf = (char*) malloc(size);
  f3(buf, size);
}
```

In doing so, there is no room for any uncertainties in the size when `memcpy` is called by `get_name`. It is guaranteed to `get_name` that the size given to `memcpy` must be the allocated size for the buffer. When a developer works on the code in `get_name`, he does not need to check all of its callers to ensure that the size is suitable. Similarly, a developer working on `store_input` also does not need to worry whether `f2`, `f3` or `get_name` will write out of bounds, assuming that the developers of those functions have used the provided `size` argument correctly.  
这样，当 `get_name` 调用 `memcpy` 时，大小没有任何不确定性。向 `get_name` 保证给予 `memcpy` 的大小必须是为缓冲区分配的大小。当开发人员处理 `get_name` 中的代码时，他不需要检查其所有调用者以确保大小合适。类似地，在 `store_input` 上工作的开发人员也不需要担心 `f2` 、 `f3` 或 `get_name` 是否会写出越界，假设这些函数的开发人员正确地使用了所提供的 `size` 参数。

Also, note that `store_input` passes the same `size` variable to `malloc` and `f3`, instead of hardcoding the value 64 in both function calls. This ensures that when the allocation size is changed, the change will immediately apply to both `malloc` and `f3`. Such practice removes the possibility of mistakes.  
另外，请注意， `store_input` 将相同的 `size` 变量传递给 `malloc` 和 `f3` ，而不是在两个函数调用中都硬编码值 64。这确保了当分配大小改变时，该改变将立即应用于 `malloc` 和 `f3` 。这种做法消除了出错的可能性。

For code reviewers, potentials bugs are also easier to detect. In the original example, the reviewer would have to follow the flow from `store_input` to `get_name` to ensure that the size given to `memcpy` is within bounds. In a big codebase, there may be tens or hundreds of such flows to review whenever the implementation of a function such as `get_name` changes.  
对于代码审查者来说，潜在的 bug 也更容易检测。在最初的示例中，审阅者必须遵循从 `store_input` 到 `get_name` 的流程，以确保指定给 `memcpy` 的大小在界限内。在一个大的代码库中，每当一个函数（如 `get_name` ）的实现发生变化时，可能会有几十个或几百个这样的流程需要审查。

In this improved implementation, the reviewer just needs to ensure that functions like `get_name` correctly uses the `size` argument that is given to it, and ensure that functions like `store_input` provide the correct size.  
在这个改进的实现中，审阅者只需要确保像 `get_name` 这样的函数正确地使用了给它的 `size` 参数，并确保像 `store_input` 这样的函数提供了正确的大小。

##### Caveat: `strncpy` 警告： `strncpy`

There are caveats for using `strncpy` and `strncat`. We will discuss `strncpy` first.  
使用 `strncpy` 和 `strncat` 有一些注意事项。我们先来讨论一下 `strncpy` 。

Note that for `strncpy`, if the requested length to copy is smaller than the length of the source string, the copied string will not be null-terminated. Consider the following example.  
请注意，对于 `strncpy` ，如果请求复制的长度小于源字符串的长度，则复制的字符串将不会以 null 结尾。考虑下面的例子。

```c
char* hello = "HELLO WORLD";
char dest[10];
memset(dest, '\xAA', 10);
strncpy(dest, hello, 5);

// to print contents of `dest` in hex
for (int i = 0; i < 10; ++i) printf("%hhx ", dest[i]);
printf("\n");
```

The result of running the code above is as follows.  
运行上面代码的结果如下。

```plain
48 45 4c 4c 4f aa aa aa aa aa
```

As `strncpy` was given 5 as the length to copy, it correctly copies 5 characters, and does nothing more than that, such as adding a null byte to terminate the destination buffer. The string in `dest` is therefore not null-terminated as shown in the program output above.  
由于 `strncpy` 被指定为要复制的长度 5，因此它正确地复制了 5 个字符，并且没有做更多的事情，例如添加一个空字节来终止目标缓冲区。因此， `dest` 中的字符串不是空终止的，如上面的程序输出所示。

The consequences of this may not be directly observed. By itself, there is no memory corruption, because nothing is read or written out of bounds. However, if a future operation that uses `dest` assumes that it is null-terminated, but in reality it may not be so, unexpected behaviour may occur. Referring to the example above, if `strlen` was applied on `dest`, it does not return 5 because there is no null-byte at the 6th position (i.e. right after the 5th position), even though it is expected to return 5. This discrepancy may affect subsequent operations in unpredicted ways.  
其后果可能无法直接观察到。就其本身而言，不存在内存损坏，因为没有任何内容被读取或写入超出界限。但是，如果使用 `dest` 的未来操作假设它是空终止的，但实际上可能并非如此，则可能会发生意外行为。参考上面的例子，如果在 `dest` 上应用 `strlen` ，它不会返回 5，因为在第 6 个位置（即紧接在第 5 个位置之后）没有空字节，即使它预期返回 5。这种差异可能会以不可预测的方式影响后续操作。

Another example would be a scenario where the contents in `dest` were to be copied as part of the service’s response to the client. Similarly, as `dest` was not null-terminated, the program may copy adjacent memory contents (contents of other variables) or leftover memory contents in the buffer written by previous operations (i.e. `\xaa\xaa\xaa` in the example above). This constitutes an information leakage bug. Since there is no out-of-bounds memory write, there is no danger of RCE. However, sensitive values could be leaked, for example pointers to bypass ASLR, or secrets like passwords to bypass authentication.  
另一个例子是将 `dest` 中的内容作为服务响应的一部分复制到客户端的场景。类似地，由于 `dest` 不是空终止的，所以程序可以复制相邻的存储器内容（其他变量的内容）或由先前操作写入的缓冲区中的剩余存储器内容（即，上面示例中的 `\xaa\xaa\xaa` ）。这就构成了一个信息泄露漏洞。由于没有越界的内存写入，因此不存在 RCE 的危险。但是，敏感值可能会泄露，例如绕过 ASLR 的指针，或者绕过身份验证的密码等秘密。

In a more severe scenario, a buffer overflow may be possible too, although unlikely if proper precautions were already enforced to ensure that memory-copying operations do not rely on the position or presence of the null byte, as per the advice given above.  
在更严重的情况下，缓冲区溢出也可能发生，尽管如果已经实施了适当的预防措施以确保内存复制操作不依赖于空字节的位置或存在，则不太可能发生。

##### Custom `strncpy` 自定义 `strncpy`

With this knowledge about `strncpy` and the implications, the developer should remember to insert a null byte to terminate the copied string, while also making sure not to write the null byte beyond the buffer’s bounds. For example:  
有了关于 `strncpy` 及其含义的知识，开发人员应该记住插入一个空字节来终止复制的字符串，同时还要确保不要将空字节写入缓冲区的边界之外。举例来说：

```c
char* hello = "HELLO WORLD";
char dest[5];

// bad, writing out of bounds
strncpy(dest, hello, 5);
dest[5] = 0;

// good, within bounds
strncpy(dest, hello, 4);
dest[4] = 0;
```

This does not look good. While the usage of `strncpy` was meant for safety purposes, it has introduced a new problem for developers to be wary of. Now, developers are required to remember adding a null byte, while also being careful about the position it is added (need to minus one from the buffer size). A lapse in concentration by a developer and a code reviewer will result in an off-by-one write bug, which under the right conditions could be exploitable by an attacker to gain RCE.  
情况不妙虽然 `strncpy` 的使用是出于安全目的，但它为开发人员带来了一个新的问题。现在，开发人员需要记住添加一个空字节，同时还要注意它被添加的位置（需要从缓冲区大小中减去 1）。开发人员和代码审查人员的注意力不集中将导致一个写错误，在适当的条件下，攻击者可以利用这个错误来获得 RCE。

There is a reliable solution to this. The development team could create a custom version of `strncpy` that best suits their needs. For example, let’s call it `my_strncpy`. The custom `my_strncpy` could behave differently from `strncpy` by ensuring that a null byte is always added at the last position, that is, length minus one. The tradeoff would be that only length minus one characters are copied from the source string, but this is not a security concern. Internal documentation should then be maintained to ensure clarity of `my_strncpy`’s behaviour. Furthermore, usage of `strncpy` should be avoided since there would not be any good reason for it to be used anymore.  
对此有一个可靠的解决方案。开发团队可以创建一个最适合他们需求的自定义版本的 `strncpy` 。例如，让我们称之为 `my_strncpy` 。自定义的 `my_strncpy` 可以通过确保在最后一个位置总是添加一个空字节（即长度减 1）来实现与 `strncpy` 不同的行为。代价是只从源字符串复制长度减一个字符，但这不是安全问题。然后应保留内部文件，以确保 4# 行为的清晰度。此外，应避免使用 `strncpy` ，因为没有任何充分的理由再使用它。

By doing the above, security is decoupled from the unexpected intricacies of standard library functions, so that developers and code reviewers are protected from making mistakes caused by unintended behaviour.  
通过上述操作，安全性与标准库函数的意外复杂性脱钩，从而保护开发人员和代码审查人员免受意外行为导致的错误。

##### Caveat: `strncat` 警告： `strncat`

The problem caused by `strncat` is the opposite of `strncpy`’s. Unlike `strncpy` which does not add a null byte to the end of the copied string, `strncat` adds a null byte after the end of the copied string. For example:  
`strncat` 引起的问题与 `strncpy` 相反。与 `strncpy` 不同， `strncat` 不会在复制的字符串末尾添加空字节，而是在复制的字符串末尾添加空字节。举例来说：

```c
char* hello = "HELLO WORLD";
char dest[10];
memset(dest, '\xAA', 10);

dest[0] = 0;
strncat(dest, hello, 5);

// to print contents of `dest` in hex
for (int i = 0; i < 10; ++i) printf("%hhx ", dest[i]);
printf("\n");
```

Even though in the code above `strncat` was given 5 as the length to copy, it had also added a null byte at the 6th position, after the end of the copied string.  
即使在上面的代码中， `strncat` 被指定为 5 作为复制的长度，它也在第 6 个位置添加了一个空字节，在复制的字符串的结尾之后。

```plain
48 45 4c 4c 4f 0 aa aa aa aa
```

This is much more dangerous than the case of `strncpy`, because there is actually an out-of-bounds memory write. Consider the contrived example below:  
这比 `strncpy` 的情况危险得多，因为实际上存在越界内存写入。看看下面这个人为的例子：

```c
char* hello = "HELLO WORLD HELLO WORLD";
char dest[12];
int needs_auth = 1;
dest[0] = 0;
strncat(dest, hello, 12);
```

Here, it depends on how the compiler allocates the variables on the stack. If `needs_auth` is allocated after `dest`, then as `strncat` is given length 12, it would write 12 bytes into `dest`, and a null byte into `needs_auth` which is stored after it. As a result, `needs_auth` would contain the value 0.  
在这里，它取决于编译器如何在堆栈上分配变量。如果在 `dest` 之后分配 `needs_auth` ，则由于 `strncat` 的长度为 12，因此它将向 `dest` 写入 12 个字节，并向在其之后存储的 `needs_auth` 写入空字节。

Such an off-by-one bug to write memory out of bounds may not appear as intimidating as a conventional buffer overflow, but it is still a buffer overflow nonetheless, despite just overflowing by just one null byte. The consequences can be dire if the null byte is written into a variable that is part of critical logic. For example, authentication could be bypassed as seen in the example above; or if values related to bounds checking were overwritten, a buffer overflow may occur.  
这样一个写内存越界的错误可能不会像传统的缓冲区溢出那样令人生畏，但它仍然是一个缓冲区溢出，尽管只是溢出了一个空字节。如果将空字节写入作为关键逻辑一部分的变量，则后果可能是可怕的。例如，如上面的示例所示，可以绕过身份验证；或者如果与边界检查相关的值被覆盖，则可能发生缓冲区溢出。

##### Custom `strncat` 自定义 `strncat`

Here, we give a similar suggestion as we did for `strncpy` to protect the developers from making mistakes caused by this tricky behaviour. Again, although it is part of the developers’ responsibility to make sure the code is correct, it is not reliable to expect such intricacies to always be on their mind, as they may be very focused on implementing the feature requirements and have forgotten about the memory side effects of their code. Thus, it is very beneficial to set up a safe development environment that takes this burden off the developers’ shoulders.  
在这里，我们给予类似的建议，因为我们为 `strncpy` ，以保护开发人员从犯错误所造成的这种棘手的行为。同样，尽管确保代码正确是开发人员的责任之一，但期望这些复杂性总是在他们的脑海中是不可靠的，因为他们可能非常专注于实现功能需求，而忘记了代码的内存副作用。因此，建立一个安全的开发环境来减轻开发人员的负担是非常有益的。

Similar to `my_strncpy`, the development team could create a custom `strncat` as well, e.g. `my_strncat`, that works according to their needs. One possible implementation for `my_strncat` is to add the null byte at the last position, that is, for example if the length field is 5, the null byte will be added at the 5th position (1-indexed). This means that only length minus one characters are concatenated to the destination string. Such behaviour should then be clearly written in the team’s internal documentation to ensure there is no ambiguity. Furthermore, usage of `strncat` from the standard library should be avoided since there is no longer a good reason for it to be used.  
与 `my_strncpy` 类似，开发团队也可以创建一个自定义的 `strncat` ，例如 `my_strncat` ，它可以根据他们的需要工作。 `my_strncat` 的一个可能的实现是在最后一个位置添加空字节，也就是说，例如如果长度字段是 5，则空字节将被添加在第 5 个位置（1 - 索引）。这意味着只有长度减一个字符连接到目标字符串。这样的行为应该清楚地写在团队的内部文档中，以确保没有歧义。此外，应该避免使用标准库中的 `strncat` ，因为不再有使用它的好理由。

#### Actionable Steps 可操作的步骤

To summarize the suggestions above, we advise development and security teams to impose the following rules on their codebase:  
为了总结上述建议，我们建议开发和安全团队在他们的代码库中实施以下规则：

1.  Avoid unbounded memory-copying functions (`strcpy`, `sprintf`, `strcat`) and use bounded functions that do the same.  
    避免无限的内存复制函数（ `strcpy` ， `sprintf` ， `strcat` ），并使用有界函数做同样的事情。
2.  Create and use your own custom implementation of library functions (such as `strncpy` or `strncat`) so that you have full control and understanding of their behaviour, to avoid pitfalls caused by unexpected intricacies of the library functions. Then, avoid using the library functions.  
    创建并使用您自己的库函数的自定义实现（例如 `strncpy` 或 `strncat` ），以便您完全控制和理解它们的行为，以避免库函数的意外复杂性导致的陷阱。避免使用库函数。
3.  Avoid hardcoding sizes for allocation and memory-copying operations. For a function that takes in a buffer pointer and writes to it, ensure that the buffer’s allocation size is also taken as an argument. This removes uncertainties about the size, for both the caller and the callee.  
    避免硬编码分配和内存复制操作的大小。对于接受缓冲区指针并向其写入的函数，请确保缓冲区的分配大小也作为参数。这消除了调用方和被调用方关于大小的不确定性。

The suggestions above all follow the same principles. We want to change the problem from “are we using this function correctly” to “are we implementing this function correctly”. There will be significantly less room for error, as the former requires reviewing every single function and trying to catch incorrect usages such as the off-by-one examples given; whereas the latter removes the possibility of such pitfalls, and only requires reviewing the custom implementations to ensure that they are implemented correctly.  
上述建议都遵循相同的原则。我们想把问题从 “我们是否正确地使用了这个函数” 变成 “我们是否正确地实现了这个函数”。错误的空间将大大减少，因为前者需要审查每个函数并试图捕捉错误的用法，例如给出的 off-by-one 示例；而后者消除了此类陷阱的可能性，只需要审查自定义实现以确保它们正确实现。

In other words, instead of worrying about “what are the possible pitfalls of using this function, is there some specific scenario that I have missed”, we can now call memory-writing functions with a piece of mind and just care about “is this function implemented correctly”.  
换句话说，与其担心 “使用这个函数可能存在哪些陷阱，是否有一些我错过的特定场景”，我们现在可以轻松地调用内存写入函数，只关心 “这个函数是否正确实现”。

In routers that are higher on the price range, we have observed a widespread application of the rules above in their codebase. As a result, it was more difficult to find bugs on these devices that are typically considered as low hanging fruits on cheaper routers.  
在价格较高的路由器中，我们观察到上述规则在其代码库中的广泛应用。因此，在这些通常被认为是廉价路由器上的低挂水果的设备上找到 bug 更加困难。

#### Examples 示例

The following are examples where a buffer overflow due to improper (or the lack of) bounds checks was exploitable to gain RCE due to the lack of mitigations such as ASLR, PIE, NX or canary.  
以下是由于缺乏 ASLR、PIE、NX 或金丝雀等缓解措施而导致的缓冲区溢出（由于边界检查不正确（或缺乏边界检查））可被利用以获得 RCE 的示例。

-   [NETGEAR R6260 CVE-2021-34979](https://starlabs.sg/advisories/21/21-34979/#exploitation)
-   [Tenda AC15 CVE-2018-5767](https://p1kk.github.io/2021/03/29/iot/Tenda%20AC15%20CVE-2018-5767%20CVE-2020-10987/#:~:text=checksec%20./bin/httpd)
-   [TP-Link TL-WR940N CVE-2022-24355](https://blog.viettelcybersecurity.com/tp-link-tl-wr940n-httpd-httprpmfs-stack-based-buffer-overflow-remote-code-execution-vulnerability/)

These show that even in the recent years having modern mitigations, simple bugs are still present in routers and could be easily exploited.  
这些表明，即使在最近几年有了现代的缓解措施，路由器中仍然存在简单的错误，并且很容易被利用。

### Format String Bug 格式字符串缺陷

A format string vulnerability could be exploited to leak pointers, perform buffer overflow, or write to certain memory locations. It is a very powerful bug. However, security teams need not be too worried about this vulnerability class, as it is one that is very easy to prevent. We explain why this is the case below.  
攻击者可利用格式字符串漏洞泄漏指针、执行缓冲区溢出或写入特定内存位置。这是一个非常强大的 bug。但是，安全团队不必太担心这种漏洞，因为它是一个非常容易预防的漏洞。下面我们解释为什么会出现这种情况。

#### Prevention 预防

Format string bugs are very easy to prevent, as it can only occur when a program passes user input directly into the format string argument of `printf` or its variant functions. An example of vulnerable code is as follows:  
格式字符串错误很容易防止，因为它只会在程序将用户输入直接传递到 `printf` 的格式字符串参数或其变体函数时发生。易受攻击的代码示例如下：

```c
printf(username);
fprintf(fp, username);
```

The correct way to print a string with `printf` is by using the `%s` specifier, as shown below:  
使用 `printf` 打印字符串的正确方法是使用 `%s` 说明符，如下所示：

```c
printf("%s", username);
fprintf(fp, "%s", username);
```

This mistake is very easy to catch. Even if a programmer actually missed it, modern C compilers will also raise a warning about it under default settings. By providing the `-Werror=format-security` flag to `gcc`, the compilation will fail with these warnings treated as errors.  
这个错误很容易被抓住。即使程序员实际上错过了它，现代 C 编译器也会在默认设置下发出警告。通过为 `gcc` 提供 `-Werror=format-security` 标志，编译将失败，这些警告将被视为错误。

```bash
$ cat fs.c
#include <stdio.h>

int main(int argc, char** argv)
{
  printf(argv[1]);
  return 0;
}

$ gcc fs.c
fs.c: In function ‘main’:
fs.c:5:3: warning: format not a string literal and no format arguments [-Wformat-security]
    5 |   printf(argv[1]);
      |   ^~

$ gcc fs.c -Werror=format-security
fs.c: In function ‘main’:
fs.c:5:3: error: format not a string literal and no format arguments [-Werror=format-security]
    5 |   printf(argv[1]);
      |   ^~~~~~
cc1: some warnings being treated as errors
```

As shown above, this bug is easy to catch and fix. However, some router binaries might have been compiled decades ago, and at that time developers might not have had such awareness of format string vulnerabilities, or the compiler used might not have emitted such warnings. There is a chance that some of these vulnerable code have persisted until today.  
如上所述，这个 bug 很容易捕获和修复。但是，有些路由器二进制文件可能是在几十年前编译的，当时的开发人员可能还没有这样的意识到格式字符串漏洞，或者使用的编译器可能还没有发出这样的警告。这些易受攻击的代码中有一些可能一直持续到今天。

#### Actionable Steps 可操作的步骤

For security teams, we advise you to check your old codebases and ensure that there are no more such bugs. We recommend adding the `-Werror=format-security` flag to `gcc` in the deployment process, to catch any such bugs that persisted from the past as well as the ones that may be accidentally introduced in the future.  
对于安全团队，我们建议您检查旧的代码库，并确保不再有此类错误。我们建议在部署过程中将 `-Werror=format-security` 标志添加到 `gcc` ，以捕获过去持续存在的任何此类错误以及将来可能意外引入的错误。

There is a caveat to take careful note of. The `-Werror=format-security` flag only works on GCC 4.3.x or newer ([source](https://stackoverflow.com/a/1047003)). Please make sure that your GCC is of a sufficiently new version to support this flag. At the point of writing this, the latest version of GCC is 13.2. GCC 4.3.6, the latest version under 4.3.x, was released in 2011.  
有一个警告需要仔细注意。 `-Werror=format-security` 标志只适用于 GCC 4.3.x 或更新版本（源代码）。请确保您的 GCC 是一个足够新的版本，以支持此标志。在撰写本文时，GCC 的最新版本是 13.2。GCC 4.3.6 是 4.3.x 下的最新版本，于 2011 年发布。

#### Example 例如

[CVE-2018-14713](https://blog.securityevaluators.com/asuswrt-buffer-overflow-format-string-aslr-bypass-2bbf9736fe46) reports a format string vulnerability on the ASUS RT-AC3200 router, in which the web server might have been written back in 1999 (according to the linked article). The format string vulnerability could be used to leak pointers in memory to bypass ASLR. Then, since there is no stack canary, a buffer overflow could be exploited by using ROP to call `system` in libc to gain RCE.  
CVE-2018-14713 报告了华硕 RT-AC 3200 路由器上的格式字符串漏洞，其中 Web 服务器可能在 1999 年被写回（根据链接文章）。此漏洞可用于泄漏内存中的指针以绕过 ASLR。然后，由于没有堆栈金丝雀，可以通过使用 ROP 调用 libc 中的 `system` 来利用缓冲区溢出来获得 RCE。

Besides the admin panel, other services on a router may be vulnerable to a format string vulnerability as well. In 2013, a format string vulnerability was discovered in the Universal Plug and Play (UPnP) service of many routers, described in great detail in the article [From Zero to ZeroDay Journey: Router Hacking (WRT54GL Linksys Case)](https://web.archive.org/web/20190608235755/http://www.defensecode.com:80/whitepapers/From_Zero_To_ZeroDay_Network_Devices_Exploitation.txt). The service does not require any authentication and accepts requests from the WAN interface.  
除了管理面板，路由器上的其他服务也可能容易受到格式字符串漏洞的攻击。2013 年，在许多路由器的通用即插即用（UPnP）服务中发现了一个格式字符串漏洞，在文章 From Zero to ZeroDay Journey：Router Hacking（WRT54GL LinkSYS Case）中有详细描述。该服务不需要任何身份验证，并接受来自 WAN 接口的请求。

The scary part about this vulnerability is that it is found in the Broadcom UPnP stack, which is code that is reused by many other router vendors. At the end of the linked article, there is a long list of routers by different vendors that are affected by this vulnerability.  
关于这个漏洞的可怕部分是，它是在 Broadcom UPnP 堆栈中发现的，这是许多其他路由器供应商重用的代码。在链接文章的最后，有一长串受此漏洞影响的不同供应商的路由器列表。

## Conclusion 结论

In this article, we have discussed the attack surface of a router as well as misconfigurations and vulnerability classes that affect exposed services. We also provided suggestions in the form of best practices and secure code design to prevent attackers from abusing these services. By designing the development environment intentionally through enforcing the usage of only secure functions, the development team can be protected from making mistakes that may have dire consequences.  
在本文中，我们讨论了路由器的攻击面以及影响公开服务的错误配置和漏洞类。我们还以最佳实践和安全代码设计的形式提供了建议，以防止攻击者滥用这些服务。通过有意识地设计开发环境，强制只使用安全功能，可以保护开发团队避免犯可能导致严重后果的错误。

There are other problems not covered in this article. Firstly, we have observed that for some vendors, many of their devices share the same codebase. However, when a vulnerability is reported for a device and remediated for that device, the fixes are not applied to the other devices which share similar code and have the same vulnerability. This is likely due to the vendors’ lack of knowledge about which devices share the same code, or if they do know, it may be because of the high cost of testing such devices.  
还有一些问题没有在本文中讨论。首先，我们观察到，对于一些供应商来说，他们的许多设备共享相同的代码库。但是，当为某个设备报告漏洞并修复该设备时，修复程序不会应用于共享类似代码并具有相同漏洞的其他设备。这可能是由于供应商缺乏关于哪些设备共享相同代码的知识，或者如果他们知道，可能是因为测试这些设备的成本很高。

When vendors leave vulnerabilities unpatched on similar devices while fixing them on only one device, attackers who notice a security update published for that one device can quickly analyse the patch and write exploits for the other unpatched devices. This leaves many consumers at risk of being compromised.  
当供应商在类似设备上留下未修补的漏洞，而仅在一台设备上修复这些漏洞时，注意到为该设备发布的安全更新的攻击者可以快速分析补丁并为其他未修补的设备编写漏洞。这使得许多消费者面临被损害的风险。

Another problem comes from the usage of open source projects. It is reasonable and beneficial for developers to import open source libraries to save development time. However, these open source projects may receive vulnerability reports and apply fixes from time to time. In a way, one benefit of using such open source projects is that developers do not need to fix the bugs found in them, as they can just update that library to the latest version. However, there may be challenges in doing so, either due to worries about compatibility, or not even being aware that a library needs to be updated.  
另一个问题来自于开源项目的使用。开发人员导入开源库来保存开发时间是合理的，也是有益的。但是，这些开源项目可能会不时收到漏洞报告并应用修复程序。在某种程度上，使用这种开源项目的一个好处是开发人员不需要修复其中发现的 bug，因为他们可以将该库更新到最新版本。然而，这样做可能会遇到挑战，要么是因为担心兼容性，要么甚至不知道库需要更新。

To summarize, aside from fixing vulnerabilities within a device, there are also challenges in ensuring that patches are applied horizontally, that is, across all devices that are affected; as well as vertically, through applying updates performed on the upstream open source projects.  
总而言之，除了修复设备中的漏洞外，还存在确保补丁横向应用的挑战，即在所有受影响的设备上应用；以及垂直应用，通过在上游开源项目上执行更新。
