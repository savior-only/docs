

# 高效的 Web 渗透测试：Mitmdump 和 Burp Suite 整合详解 - 先知社区

高效的 Web 渗透测试：Mitmdump 和 Burp Suite 整合详解

- - -

# mitmdump

最近好几次众测遇到前端 js 加密，除了需要进行前端加密算法分析之外，还需要 mitmdump 配合 burp 进行自动化解密，因此这里就遇到了 mitmdump 的使用，之前遇到的加密站是朋友帮忙写的，我本想着在此基础上修修改改但是在修改的过程中遇到了很多问题，本质上是对上下游代理的转发不是很熟悉，于是自己在本地进行手动调试做了详细的记录。

## 上下游代理

在此之前我们先了解一下代理，平时我们正常渗透一般是在浏览器挂代理，将流量转发给 burp，在转发给服务器。同时服务器响应先返回给 burp，burp 在返回给客户端。

[![](assets/1704158228-675579fc00a5a0eb4f01951638c1ddeb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213045-cd1657e2-a7e0-1.png)

那么我们结合 mitmdump 的时候具体的请求如下：

[![](assets/1704158228-e9c8764aacbb9c5c662df717b91fcfff.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213902-f5a184ba-a7e1-1.png)

### 下游代理

下游代理主要是接收客户端的加密请求密文，进行解密将解密结果发送到 burp，burp 将解密的数据呈现给我们。同时将 burp 传递的明文加密然后返回给客户端。

### 上游代理

上游代理主要是处理 burp 传递的请求进行加密返回给服务端。同时接收服务端的加密响应密文，进行解密将解密结果发送到 burp，burp 将解密结果呈现给我们。

### burp

burp 的主要作用其实就是将解密结果呈现给我们，在一个就是可以使用 repeter 重发。

## mitmproxy

上下游代理均需要使用函数去处理请求和响应。

```bash
from mitmproxy import flowfilter,ctx
from mitmproxy.http import HTTPFlow
from mitmproxy import flowfilter
from mitmproxy.http import HTTPFlow

class Mimit():
    def request(self,flow):

    def response(self,flow):

addons = [Mimit(),]
```

上游代理：

-   request 函数：处理 burp 发送的请求，编写函数进行加密并将结果发送给服务端。因为上游的 request 请求是处理 burp 中的数据的，在 burp 中无法看到处理之后的请求。这里通过控制台演示一下

[![](assets/1704158228-ed4da5a018cb7f3290322d8d8024dce8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213120-e224054e-a7e0-1.png)

-   response 函数：处理服务端返回的响应，编写函数进行解密将结果发送给 burp。

[![](assets/1704158228-085c43f989b6c115889971f78a8a3e04.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213131-e88cf44a-a7e0-1.png)

下游代理：

-   request 函数：处理客户端发送的请求，编写函数进行解密并将结果返回给 burp。

[![](assets/1704158228-791cf50bb32a5488cc9759b36b10c3d9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213147-f2632b74-a7e0-1.png)

-   response 函数：处理 burp 接收到的明文响应，编写函数进行加密并将结果返回给客户端。  
    [![](assets/1704158228-a702af49cb7a173807e214c7104c2f6b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213157-f87ec220-a7e0-1.png)

为了方便理解，这里单独对上、下游代理进行调试，梳理一下处理流程。这里我使用 GPT 简单写了一个客户端和服务端。

主要逻辑就是客户端对数据 base64 加密，然后服务端解密继续 base64 加密并将结果返回。

前端代码

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Base64 编码示例</title>
</head>
<body>

    <form id="myForm">
        <label for="userInput">输入文本：</label>
        <input type="text" id="userInput" name="userInput" required>
        <button type="button" onclick="encodeAndSend()">提交</button>
    </form>

    <!-- 添加用于显示服务端返回的数据的元素 -->
    <div id="responseDataContainer"></div>

    <script>
        function encodeAndSend() {
            var userInput = document.getElementById("userInput").value;
            var encodedData = btoa(userInput);

            var jsonData = {
                encodedData: encodedData
            };

            fetch('1.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(jsonData)
            })
            .then(response => response.text()) // 修改部分：使用 response.text() 获取字符串
            .then(data => {
                // 修改部分：将服务端返回的字符串显示在页面上
                var responseDataContainer = document.getElementById("responseDataContainer");
                responseDataContainer.innerHTML = '服务端返回的数据： ' + data;
            })
            .catch(error => {
                console.error('Error:', error);
            });
        }
    </script>

</body>
</html>
```

后端代码

```bash
<?php
header('Content-Type: application/json');

// 获取 POST 参数
$postData = json_decode(file_get_contents("php://input"), true);

// 检查是否接收到有效数据
if (isset($postData['encodedData'])) {
    // 获取 Base64 编码的数据
    $encodedData = $postData['encodedData'];

    // 判断是否以等号("=")结尾
     if (substr($encodedData, -1) === '=') {
            // 如果以等号结尾，解码并再次进行加密传递给客户端
            $decodedData = base64_decode($encodedData);
            $decodedData = base64_encode($decodedData);
    } else {
        // 如果不是 Base64 编码，直接对原始数据进行 Base64 编码
        $decodedData = base64_encode($encodedData);
    }

    // 返回继续加密后的数据
    $response = array('reencodedData' => $decodedData);
    echo json_encode($response);
} else {
    // 返回错误信息
    $errorResponse = array('error' => 'Invalid data received');
    echo json_encode($errorResponse);
}
?>
```

正常请求，如图：可以看到正常的请求就是加密处理的，这里我们就需要使用 mitmproxy 对请求和响应加解密方便我们进行渗透测试。

[![](assets/1704158228-90f8f77b8b34a3f4e8b319be4ae87f66.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213211-00ebb1ac-a7e1-1.png)

## 上游代理调试

下游处理函数

```bash
from mitmproxy import flowfilter,ctx
from mitmproxy.http import HTTPFlow
from mitmproxy import flowfilter
from mitmproxy.http import HTTPFlow
import base64

class Mimit():
    def request(self,flow):
        if flow.request.host=="192.168.0.101":
            req = json.loads(flow.request.get_text())
            ctx.log.info("浏览器请求数据 => "+req['encodedData'])
            data = base64.b64decode(str(req['encodedData']).encode()).decode()
            req['encodedData'] = data
            ctx.log.info("下游代理解密请求数据 => " + req['encodedData'])
            flow.request.set_text((json.dumps(req)))

    def response(self,flow):
        if flow.request.host=="192.168.0.101":
            rep = json.loads(flow.response.get_text())
            ctx.log.info("响应数据 => "+rep['decodedData'])
            data = base64.b64encode(str(rep['decodedData']).encode()).decode()
            ctx.log.info("响应数据 => " + data)
            rep['decodedData'] = data
            flow.response.set_text(json.dumps(rep))

addons = [Mimit(),]
```

命令行启动下游代理并将上游代理设置为 burp。

```bash
mitmdump -p 7070 -s dec.py  --mode upstream:http://127.0.0.1:8080 --ssl-insecure
```

在浏览器中设置下游代理。

[![](assets/1704158228-c912578e68448e602e32247456957d05.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213225-08d16df8-a7e1-1.png)

此时的流程为

[![](assets/1704158228-c9cfae62e82a95000a0df25bfdaaff12.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213234-0e1f6f76-a7e1-1.png)

配置好之后发起请求，此时我们查看命令行和 burp 中的数据

[![](assets/1704158228-7596b0dfb077faa94fe75b5076eedfbc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213244-14621eba-a7e1-1.png)

可以看到，没有挂上游代理不会对响应数据进行解密，只是下游代理将客户端的请求解密在 burp 中呈现，因此请求包是明文，返回包是密文。

下游代理会加密 burp 传递的参数，也就是服务端传递的 bWl0bXByb3h5，最终得到结果 YldsMGJYQnliM2g1。因为这里没有上游代理，返回包没有被解密，所以会二次 base64 加密嘛。

[![](assets/1704158228-a578b0b0ce9f609ea96c1921103e3b22.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213302-1f41b02a-a7e1-1.png)

## 上游代理调试

上游处理函数

```bash
from mitmproxy import flowfilter,ctx
from mitmproxy.http import HTTPFlow
from mitmproxy import flowfilter
from mitmproxy.http import HTTPFlow
import base64

class Mimit():
    def request(self,flow):
        if flow.request.host=="192.168.0.101":
            req = json.loads(flow.request.get_text())
            ctx.log.info("浏览器请求数据 => "+req['encodedData'])
            data = base64.b64decode(str(req['encodedData']).encode()).decode()
            req['encodedData'] = data
            ctx.log.info("下游代理解密请求数据 => " + req['encodedData'])
            flow.request.set_text((json.dumps(req)))

    def response(self,flow):
        if flow.request.host=="192.168.0.101":
            rep = json.loads(flow.response.get_text())
            ctx.log.info("响应数据 => "+rep['decodedData'])
            data = base64.b64encode(str(rep['decodedData']).encode()).decode()
            ctx.log.info("响应数据 => " + data)
            rep['decodedData'] = data
            flow.response.set_text(json.dumps(rep))
```

命令行启动上游代理。

```bash
mitmdump -p 9091 -s enc.py --ssl-insecure
```

burp 挂上游代理

[![](assets/1704158228-4373ebe274e7f51ff8e44c0891f0a374.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213314-264722c4-a7e1-1.png)

此时流程为

[![](assets/1704158228-d4b8373571e0792620cb652ee3e507ee.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231214820-424f3086-a7e3-1.png)

配置好之后发起请求，此时我们查看命令行和 burp 中的数据。注意因为我们没有下游代理不会对客户端发送的请求解密，然而上游代理会进行加密，因此进行了二次 base64 加密。此时为了方便岩石，服务端也需要二次解密。所以修改一下代码。

```bash
// 判断是否以等号 ("=") 结尾
     if (substr($encodedData, -1) === '=') {
            // 如果以等号结尾，解码并再次进行加密传递给客户端
            $decodedData = base64_decode($encodedData);
            $decodedData = base64_decode($decodedData);
            $decodedData = base64_encode($decodedData);
    } else {
        // 如果不是 Base64 编码，直接对原始数据进行 Base64 编码
        $decodedData = base64_encode($encodedData);
    }
```

可以看到上游代理对服务端返回的数据进行解密，呈现给客户端。

[![](assets/1704158228-751286f407582901f839c5fe30388f24.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213337-33ece6de-a7e1-1.png)

此时并没有下游代理对数据进行加密，因此客户端接收的是明文。

[![](assets/1704158228-933d98ceeb1b2d83c56c902ac40b0264.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213348-3ab71142-a7e1-1.png)

## 上下游调试

最后我们上、下游代理同时挂起进行调试。

下游代理

```bash
mitmdump -p 7070 -s dec.py  --mode upstream:http://127.0.0.1:8080 --ssl-insecure
```

[![](assets/1704158228-7420e270f23d580a0bc4c2dba4145c13.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213359-40e1887c-a7e1-1.png)

上游代理

```bash
mitmdump -p 9091 -s enc.py --ssl-insecure
```

[![](assets/1704158228-a19c9b50458b257f9c28ead3062c04aa.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213409-47036810-a7e1-1.png)

最终 burp 呈现数据如下

[![](assets/1704158228-053f5ad3a6d71ac034ddad6a0b16203d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213420-4d385394-a7e1-1.png)

客户端也可以正常接收请求

[![](assets/1704158228-dce272b93bc9af2e7a08d71bb1b320e6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213429-53106626-a7e1-1.png)

## Debug

平时在测试的时候，编写脚本可能需要 debug，在上游代理可以使用以下脚本进行 debug

```bash
import sys
import os
from mitmproxy.tools.main import mitmdump

sys.path.append(os.path.dirname(os.path.abspath(__file__)))
print(os.path.dirname(os.path.abspath(__file__)))

# 使用 mitmdump 启动并指定端口
mitmdump(['-s', 'enc.py', '-p', str(9091)])
```

此时在 enc.py 中下断点就行。

[![](assets/1704158228-5311b230ab558a95428f0bdd6ab4c85c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213443-5b7ca176-a7e1-1.png)

下游代理同理

```bash
import sys
import os
from mitmproxy.tools.main import mitmdump

sys.path.append(os.path.dirname(os.path.abspath(__file__)))
print(os.path.dirname(os.path.abspath(__file__)))


# 启动 mitmdump
mitmdump(['-s', 'dec.py','-p', str(7071), '--mode', "upstream:http://127.0.0.1:8080"])
```

在 dec.py 下断点即可

[![](assets/1704158228-6c527d15eae7c287fc9d35197fe1a894.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231231213454-6210fbc2-a7e1-1.png)
