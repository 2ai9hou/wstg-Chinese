# 测试SSI注入

|ID          |
|------------|
|WSTG-INPV-08|

## 概述

Web服务器通常为开发人员提供在静态HTML页面中添加小型动态代码片段的功能，而无需处理完整的服务器端或客户端语言。此功能由[服务器端包含（SSI）](https://owasp.org/www-community/attacks/Server-Side_Includes_%28SSI%29_Injection)提供。

服务器端包含是Web服务器在将页面提供给用户之前解析的指令。它们代表编写CGI程序或嵌入服务器端脚本语言代码的替代方案，当只需要执行非常简单的任务时。常见SSI实现提供指令（命令）以包含外部文件、设置和打印Web服务器CGI环境变量或执行外部CGI脚本或系统命令。

SSI可能导致远程命令执行（RCE），但大多数Web服务器默认禁用`exec`指令。

此漏洞与经典脚本语言注入漏洞非常相似。一个缓解措施是Web服务器需要配置为允许SSI。另一方面，SSI注入漏洞通常更容易利用，因为SSI指令易于理解，同时又非常强大，例如，它们可以输出文件内容并执行系统命令。

## 测试目标

- 识别SSI注入点。
- 评估注入的严重程度。

## 如何测试

要测试可利用的SSI，请将SSI指令作为用户输入注入。如果启用SSI且未正确实施用户输入验证，服务器将执行该指令。这与经典脚本语言注入漏洞非常相似，因为当用户输入未正确验证和清理时会发生。

首先确定Web服务器是否支持SSI指令。答案通常是肯定的，因为SSI支持非常常见。要确定是否支持SSI指令，请确定目标运行的Web服务器类型（请参阅[指纹识别Web服务器](../01-Information_Gathering/02-Fingerprint_Web_Server.md)）。如果您可以访问代码，请通过在Web服务器配置文件中搜索特定关键字来确定是否使用了SSI指令。

验证SSI指令是否启用的另一种方法是检查带有`.shtml`扩展名的页面，这些页面与SSI指令相关联。使用`.shtml`扩展名不是强制的，因此未找到任何`.shtml`文件并不一定意味着目标不易受SSI注入攻击。

下一步是确定所有可能的用户输入向量，并测试SSI注入是否可利用。

首先找到允许用户输入的所有页面。可能的输入向量可能还包括标题和Cookie。确定输入如何存储和使用，即如果输入作为错误消息或页面元素返回，以及是否以某种方式对其进行了修改。访问源代码可以帮助您更轻松地确定输入向量的位置以及如何处理输入。

一旦有了潜在注入点的列表，您可以确定是否可以正确验证输入。确保可以注入SSI指令中使用的字符，例如`<!#=/."->`和`[a-zA-Z0-9]`。

以下示例返回变量的值。[参考资料](#参考资料)部分提供了有用的链接，帮助您更好地评估特定系统。

```html
<!--#echo var="VAR" -->
```

当使用`include`指令时，如果提供的文件是CGI脚本，此指令将包含CGI脚本的输出。此指令也可用于包含文件内容或列出目录中的文件：

```html
<!--#include virtual="FILENAME" -->
```

要返回系统命令的输出：

```html
<!--#exec cmd="OS_COMMAND" -->
```

如果应用程序存在漏洞，则注入指令，当下次提供页面时，指令将被服务器解释。

如果Web应用程序使用该数据构建动态生成的页面，SSI指令也可以注入HTTP头：

```text
GET / HTTP/1.1
Host: www.example.com
Referer: <!--#exec cmd="/bin/ps ax"-->
User-Agent: <!--#include virtual="/proc/version"-->
```

## 工具

- [Web代理Burp Suite](https://portswigger.net/burp/communitydownload)
- [ZAP](https://www.zaproxy.org/)
- [字符串搜索器：grep](https://www.gnu.org/software/grep)

## 参考资料

- [Nginx SSI模块](https://nginx.org/en/docs/http/ngx_http_ssi_module.html)
- [Apache：模块mod_include](https://httpd.apache.org/docs/current/mod/mod_include.html)
- [IIS：服务器端包含指令](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525185%28v=vs.90%29)
- [Apache教程：服务器端包含简介](https://httpd.apache.org/docs/current/howto/ssi.html)
- [Apache：服务器配置安全提示](https://httpd.apache.org/docs/current/misc/security_tips.html#ssi)
- [SSI注入而不是JavaScript恶意软件](https://jeremiahgrossman.blogspot.com/2006/08/ssi-injection-instead-of-javascript.html)
- [IIS：服务器端包含（SSI）语法说明](https://blogs.iis.net/robert_mcmurray/archive/2010/12/28/iis-notes-on-server-side-includes-ssi-syntax-kb-203064-revisited.aspx)
- [基于头部的利用](https://www.cgisecurity.com/papers/header-based-exploitation.txt)
