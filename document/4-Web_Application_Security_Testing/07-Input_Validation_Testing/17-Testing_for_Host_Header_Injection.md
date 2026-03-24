# 测试主机头注入

|ID          |
|------------|
|WSTG-INPV-17|

## 概述

Web服务器通常在同一IP地址上托管多个Web应用程序，通过虚拟主机引用每个应用程序。在传入的HTTP请求中，Web服务器通常根据Host头中提供的值将请求分派到目标虚拟主机。如果没有正确验证头值，攻击者可以提供无效输入，导致Web服务器：

- 将请求分派到列表上的第一个虚拟主机。
- 执行到攻击者控制域的重定向。
- 执行Web缓存中毒。
- 操作密码重置功能。
- 允许访问本不应从外部访问的虚拟主机。

## 测试目标

- 评估主机头是否在应用程序中动态解析。
- 绕过依赖该头的安全控制。

## 如何测试

初始测试非常简单，只需向Host头字段提供另一个域名（即`attacker.com`）。攻击是否有效取决于Web服务器处理该头值的方式。当Web服务器处理输入以将请求发送到攻击者控制的域（而非位于Web服务器上的内部虚拟主机）时，攻击是有效的。

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

在最简单的情况下，这可能导致302重定向到提供的域。

```http
HTTP/1.1 302 Found
[...]
Location: https://www.attacker.com/login.php

```

或者，Web服务器可能将请求分派到列表上的第一个虚拟主机。

### X-Forwarded-Host头绕过

如果通过Host头注入的无效输入来缓解主机头注入，您可以向`X-Forwarded-Host`头提供该值。

```http
GET / HTTP/1.1
Host: www.example.com
X-Forwarded-Host: www.attacker.com
[...]
```

可能产生如下客户端输出：

```html
[...]
<link src="https://www.attacker.com/link" />
[...]
```

再一次，这取决于Web服务器如何处理该头值。

### Web缓存中毒

使用此技术，攻击者可以操纵Web缓存以向任何请求它的人提供中毒内容。这取决于能够毒化由应用程序本身、CDN或其他下游提供商运行的缓存代理。然后，受害者在请求易受攻击的应用程序时将无法控制收到恶意内容。

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

当受害者访问易受攻击的应用程序时，将从Web缓存提供以下内容。

```html
[...]
<link src="https://www.attacker.com/link" />
[...]
```

### 密码重置 poisoning

密码重置功能通常在创建使用生成令牌的重置链接时包含Host头值。如果应用程序处理攻击者控制的域来创建密码重置链接，受害者可能点击电子邮件中的链接，允许攻击者获取重置令牌，从而重置受害者的密码。

下面的示例显示使用`$_SERVER['HTTP_HOST']`的值在PHP中生成的密码重置链接，其基于HTTP Host头的内容设置：

```php
$reset_url = "https://" . $_SERVER['HTTP_HOST'] . "/reset.php?token=" .$token;
send_reset_email($email,$rset_url);
```

通过使用篡改的Host头发出密码重定向页面请求，我们可以修改URL指向的位置：

```http
POST /request_password_reset.php HTTP/1.1
Host: www.attacker.com
[...]

email=user@example.org
```

然后，指定域（`www.attacker.com`）将用于重置链接，该链接通过电子邮件发送给用户。当用户点击此链接时，攻击者可以窃取令牌并危害其账户。

```text
... Email snippet ...

Click on the following link to reset your password:

https://www.attacker.com/reset.php?token=12345

... Email snippet ...
```

### 访问私有虚拟主机

在某些情况下，服务器可能具有不期望从外部访问的虚拟主机。这最常见于[分DNS](https://en.wikipedia.org/wiki/Split-horizon_DNS)设置（内部和外部DNS服务器为同一域返回不同记录）。

例如，组织可能在内部网络的单一Web服务器上托管其公共网站（`www.example.org`）及其内部Intranet（在`intranet.example.org`上，但该记录仅存在于内部DNS服务器上）。虽然从网络外部直接浏览到`intranet.example.org`是不可能的（因为该域将无法解析），但可以通过使用以下`Host`头从外部访问Intranet：

```http
Host: intranet.example.org
```

这也可以通过向主机文件添加`intranet.example.org`条目（使用`www.example.org`的公共IP地址）或在测试工具中覆盖DNS解析来实现。

## 参考资料

- [什么是主机头攻击？](https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)
- [主机头攻击](https://www.briskinfosec.com/blogs/blogsdetail/Host-Header-Attack)
- [HTTP主机头攻击](https://portswigger.net/web-security/host-header)
