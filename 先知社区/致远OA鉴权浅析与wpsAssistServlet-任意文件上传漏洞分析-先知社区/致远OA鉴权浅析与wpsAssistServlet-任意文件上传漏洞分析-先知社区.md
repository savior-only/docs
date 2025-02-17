

# 致远 OA 鉴权浅析与 wpsAssistServlet 任意文件上传漏洞分析 - 先知社区

致远 OA 鉴权浅析与 wpsAssistServlet 任意文件上传漏洞分析

- - -

# 路由分析

核心路由在/ApacheJetspeed/webapps/seeyon/WEB-INF/web.xml 配置文件中，部分内容如下图  
[![](assets/1703210029-41ca646327e390c4a9abf2877c2bd422.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210440-d6ff4462-9cdc-1.png)  
[![](assets/1703210029-3c23de476f8486deef05b658a1a45734.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210451-dd3b9b78-9cdc-1.png)  
定义了 Servlet 组件，和过滤器。查看 web.xml 可以知道主要权限校验部分在 CTPSecurityFilter 过滤器中。可以在配置文件查看应用程序的 Servlet 组件，再去 WEN-INF 下的 lib 里找相应的 jar 包反编译得到源码。另外系统使用了 Spring 框架，我们知道 spring 中路由注解（如`@RequestMapping`）来定义或者通过 Spring 的配置文件来定义，这里使用配置文件定义。我们可以去存放配置文件的地方，这里是 ApacheJetspeed/webapps/seeyon/WEB-INF/cfgHome/spring 中，查看相应路由。  
[![](assets/1703210029-f2a4fefafc6286846de80db8ed78f294.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210504-e53a02e2-9cdc-1.png)

# 鉴权分析

权限校验在 CTPSecurityFilter 过滤器中，其中 doFilter 方法部分代码如下，其中重点方法为`authenticate`，进行用户认证，并获取结果。后续都为对认证通过后的 Url 的检查，看是否在保护名单中。  
[![](assets/1703210029-a9e2fa7ed2c900b0f63f1821e69705cd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210514-eb0d2e7e-9cdc-1.png)  
所以目前重点为对`authenticate`方法进行分析，看能否绕过相应校验。查看方法首先会检查 URI 是否存在攻击。  
[![](assets/1703210029-73275369acf8846634c50796f3b2d2df.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210528-f360d7e2-9cdc-1.png)

不是的话才会进入 else 中进行以下的 if 判断。

```bash
try {
                    if (isSpringController(uri, req)) {
                        result.setAuthenticator(controllerAuthenticator);
                        result.authenticate(req, resp);
                        if (result.getResult()) {
                            if (tokenAuthenticator.validate(uri, req)) {
                                result.setAuthenticator(tokenAuthenticator);
                                result.authenticate(req, resp);
                                return result;
                            }

                            if (isAjax(uri, req)) {
                                result.setAuthenticator(ajaxAuthenticator);
                                result.authenticate(req, resp);
                            }
                        }
                    } else if (tokenAuthenticator.validate(uri, req)) {
                        result.setAuthenticator(tokenAuthenticator);
                        result.authenticate(req, resp);
                    } else if (isRest(uri, req)) {
                        result.setAuthenticator(restAuthenticator);
                        result.authenticate(req, resp);
                    } else if (isV3xAjax(uri, req)) {
                        result.setAuthenticator(v3xAjaxAuthenticator);
                        result.authenticate(req, resp);
                    } else if (isSOAP(uri, req)) {
                        result.setAuthenticator(soapAuthenticator);
                        result.authenticate(req, resp);
                    } else if (isServlet(uri, req)) {
                        result.setAuthenticator(servletAuthenticator);
                        result.authenticate(req, resp);
                    } else if (isJSP(uri, req)) {
                        result.setAuthenticator(jspAuthenticator);
                        result.authenticate(req, resp);
                    } else if (webOfficeAuthenticator.validate(uri, req)) {
                        result.setAuthenticator(webOfficeAuthenticator);
                        result.authenticate(req, resp);
                    } else {
                        result.setAuthenticator(defaultAuthenticator);
                        result.authenticate(req, resp);
                    }

                    if (result.getResult()) {
                        getHttpSecurityFilterManager().doFrameFilter(req, resp);
                    }
                } catch (Exception var9) {
                    logger.error(var9.getLocalizedMessage(), var9);
                }
```

会根据请求的 URI，选择并执行不同的认证策略。这里我们先看后续分析的漏洞，会利用到的 isServlet 判断。查看代码，发现是调用 `ServletAuthenticator` 类的 `accept` 方法对请求进行判断。  
[![](assets/1703210029-17dc89d42fc303a4860247f8bdd7e7b2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210545-fdb7ce76-9cdc-1.png)  
查看`accept` 方法，首先获取请求的 URI，然后判断是否以特定的后缀 ".psml" 结尾，是的话立即返回 true。不是的话判断 uri 是否在集合 servlets 中。  
[![](assets/1703210029-ad2a78f4ae8c148d4dd37e1564b82347.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210559-05b5b75a-9cdd-1.png)  
其中 servlets 为如下静态成员变量  
[![](assets/1703210029-8fcae24747f2714e3e9b7b71416cb802.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210610-0c700d3e-9cdd-1.png)  
这里我们可以发现 web.xml 中的 servlet 比 servlets 列表中的多。也就是并不是所有定义的 servlet 都会进入 ServletAuthenticator。所以这也导致一些 web.xml 中配置的 servlet 会进入到 defaultAuthenticator，而默认的认证器 defaultAuthenticator，根据以下代码可知。并未做任何校验所有传入的 URI 都被视为有效。  
[![](assets/1703210029-61c4887f0caa92e1f7f61dfdb7c23007.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210626-1634182e-9cdd-1.png)

# Wps 任意文件上传

这里我们可以发现这个 servlet，进入 CTPSecurityFilter 过滤器，且不在 ServletAuthenticator 的 servlets 列表中，也就是会进入默认的认证器 defaultAuthenticator。那么就是没有鉴权的操作。  
[![](assets/1703210029-916580d07a87ac7fcfff19e3265f1828.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217211258-ff6721f8-9cdd-1.png)  
进入 WPSOfficeServlet 查看代码，这里会从请求中获取名为 `flag` 的参数，这个参数表示后续要执行的操作类型。  
[![](assets/1703210029-29e59800fb9668ef9ae99dad0f874a50.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217211359-24148fae-9cde-1.png)  
这里可以发现当，flag 为 save 时会执行 saveFile 方法，应该是上传保存文件的操作，这里跟进方法。  
[![](assets/1703210029-c888b79620fe166aa70d19c44f420a26.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210741-42a8c918-9cdd-1.png)  
可以发现这里，从请求头中获取，fileId 参数作为要保存的文件的标识，然后调用 canGetLock(Long fileId) 方法检查 locksMap 中是否已经存在对应 fileId 的锁定标记。所以请求中必须要包含 fileId 参数，切 locksMap 中存在。然后才会调用 oaSaveFile 方法进行保存。  
[![](assets/1703210029-2c4e73e0497469d6b9a57aead0c4ced8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210757-4c4adbdc-9cdd-1.png)  
这里 oaSaveFile 方法会将请求中的 realFileType 参数拼接进入文件路径中，所以这里可以通过../穿越目录。便可以上传 jsp 文件至可执行目录。  
[![](assets/1703210029-065a95bd3945075d75900454c7b3443f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210811-5488be0e-9cdd-1.png)  
我们便可以构造任意 jsp 文件上传至指定目录并执行了。  
[![](assets/1703210029-3f732ce29eb2d17a656217081c9d88e2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231217210829-5f1afb8e-9cdd-1.png)

参考：\[[https://mp.weixin.qq.com/s/TjKLMesbRCry7Tk8k8kjYA](https://mp.weixin.qq.com/s/TjKLMesbRCry7Tk8k8kjYA)\]
