# 测试客户端URL重定向

|ID          |
|------------|
|WSTG-CLNT-04|

## 概述

本节描述如何检查客户端URL重定向，也称为开放重定向。它是一种输入验证缺陷，当应用程序接受用户控制的输入指定指向外部URL的链接时存在，该链接可能是恶意的。这类漏洞可用于完成钓鱼攻击或将受害者重定向到感染页面。

当应用程序接受包含URL值且不清理的不受信任输入时，会发生此漏洞。此URL值可能导致Web应用程序将用户重定向到另一个页面，例如攻击者控制的恶意页面。

此漏洞可能使攻击者成功启动钓鱼诈骗并窃取用户凭据。由于重定向源自真实应用程序，钓鱼尝试可能具有更值得信赖的外观。

这是钓鱼攻击URL的示例。

```text
https://www.target.site?#redirect=www.fake-target.site
```

访问此URL的受害者将自动重定向到`fake-target.site`，攻击者可以在那里放置类似于预期站点的假页面，以窃取受害者的凭据。

开放重定向也可用于制作URL，以绕过应用程序的访问控制检查并将攻击者转发到他们通常无法访问的特权功能。

## 测试目标

- 识别处理URL或路径的注入点。
- 评估系统可以重定向到的位置。

## 如何测试

当测试人员手动检查此类漏洞时，他们首先识别是否在客户端代码中实现了客户端重定向。这些重定向可能使用`window.location`对象实现，以JavaScript为例。这可用于通过简单地将字符串赋值给它来将浏览器引导到另一个页面，如下所示：

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location='https://'+decodeURIComponent(redir);
}
```

在此示例中，脚本不执行任何验证变量`redir`，该变量通过查询字符串接收用户提供的输入。由于未应用任何形式的编码，此未验证输入被传递到`windows.location`对象，创建URL重定向漏洞。

这意味着攻击者可以简单通过提交以下查询字符串将受害者重定向到恶意站点：

```text
https://www.victim.site/?#www.malicious.site
```

通过轻微修改，上述示例片段可能容易受到JavaScript注入。

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location=decodeURIComponent(redir);
}
```

可以通过提交以下查询字符串来利用：

```text
https://www.victim.site/?#javascript:alert(document.cookie)
```

测试此漏洞时，请注意某些字符在不同浏览器中的处理方式不同。请参阅[基于DOM的XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)。
