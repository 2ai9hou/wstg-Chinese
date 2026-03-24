# 测试认证架构绕过

|ID          |
|------------|
|WSTG-ATHN-04|

## 概述

在计算机安全中，认证是尝试验证通信发送者数字身份的过程。这样的过程的一个常见例子是登录过程。测试认证架构意味着理解认证过程如何工作，并利用这些信息来规避认证机制。

虽然大多数应用程序需要认证才能访问私有信息或执行任务，但并非每种认证方法都能提供足够的安全保障。对安全威胁的疏忽、无知或简单低估通常会导致认证方案被简化为：通过跳过登录页面直接调用内部页面（这些页面本应在完成认证后才能访问）来绕过认证。

此外，通过篡改请求并欺骗应用程序使用户相信已经完成认证，通常也可以绕过认证措施。这可以通过修改给定的 URL 参数、操作表单或伪造会话来实现。

与认证架构相关的问题可以在软件开发生命周期（SDLC）的不同阶段发现，如设计阶段、开发阶段和部署阶段：

- 设计阶段的错误可能包括：未正确划定需要保护的应用程序区域，未选择强加密协议来保护凭据传输，以及更多其他错误。
- 开发阶段的错误可能包括：输入验证功能实现不正确，或未遵循特定语言的安全最佳实践。
- 应用程序部署阶段可能由于缺乏所需技术技能或缺乏良好文档而在应用程序设置（安装和配置活动）期间出现问题。

## 测试目标

- 确保所有需要认证的服务都应用了认证。

## 如何测试

有几种方法可以绕过 Web 应用程序使用的认证架构：

- 参数修改
- 会话 ID 预测
- SQL 注入

### 参数修改

与认证设计相关的另一个问题是，应用程序基于固定值参数验证成功登录。用户可以修改这些参数来获得对受保护区域的访问，而无需提供有效凭据。在以下示例中，"authenticated"参数被更改为"yes"值，这允许用户获得访问权限。在这个例子中，参数在 URL 中，但也可以使用代理修改参数，特别是当参数作为 POST 请求中的表单元素发送时，或存储在 cookie 中时。

```html
https://www.site.com/page.asp?authenticated=no

raven@blackbox /home $nc www.site.com 80
GET /page.asp?authenticated=yes HTTP/1.0

HTTP/1.1 200 OK
Date: Sat, 11 Nov 2006 10:22:44 GMT
Server: Apache
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>You Are Authenticated</H1>
</BODY></HTML>
```

![参数修改请求](images/Basm-parammod.jpg)\
*图 4.4.4-1：参数修改请求*

### 会话 ID 预测

许多 Web 应用程序使用会话标识符（会话 ID）来管理认证。因此，如果会话 ID 生成是可预测的，恶意用户可能能够找到有效的会话 ID 并获得对应用程序的未授权访问，冒充先前已认证的用户。

在下图中，cookie 中的值线性增加，因此攻击者可能很容易猜到有效的会话 ID。

![Cookie 值随时间变化](images/Basm-sessid.jpg)\
*图 4.4.4-2：Cookie 值随时间变化*

在下图中，cookie 中的值仅部分更改，因此可以将暴力破解攻击限制在下面显示的定义字段中。

![部分更改的 Cookie 值](images/Basm-sessid2.jpg)\
*图 4.4.4-3：部分更改的 Cookie 值*

### SQL 注入（HTML 表单认证）

SQL 注入是一种广为人知的攻击技术。本节不打算详细描述此技术，因为本指南中有多个部分解释注入技术，超出了本节的范围。

![SQL 注入](images/Basm-sqlinj.jpg)\
*图 4.4.4-4：SQL 注入*

下图显示，通过简单的 SQL 注入攻击，有时可以绕过认证表单。

![简单 SQL 注入攻击](images/Basm-sqlinj2.gif)\
*图 4.4.4-5：简单 SQL 注入攻击*

### PHP 松散比较

如果攻击者能够通过利用先前发现的漏洞（例如目录遍历）或从 Web 仓库（开源应用程序）获取应用程序源代码，则可能对认证过程的实现进行精细攻击。

在以下示例中（PHPBB 2.0.12 - 认证绕过漏洞），在第 2 行，`unserialize()` 函数解析用户提供的 cookie 并在 `$sessiondata` 数组中设置值。在第 7 行，存储在后端数据库中的用户 MD5 密码哈希（`$auto_login_key`）与用户提供的（`$sessiondata['autologinid']`）进行比较。

```php
1. if (isset($HTTP_COOKIE_VARS[$cookiename . '_sid'])) {
2.     $sessiondata = isset($HTTP_COOKIE_VARS[$cookiename . '_data']) ? unserialize(stripslashes($HTTP_COOKIE_VARS[$cookiename . '_data'])) : array();
3.     $sessionmethod = SESSION_METHOD_COOKIE;
4. }
5. $auto_login_key = $userdata['user_password'];
6. // We have to login automagically
7. if( $sessiondata['autologinid'] == $auto_login_key )
8. {
9.     // autologinid matches password
10.     $login = 1;
11.     $enable_autologin = 1;
12. }

```

在 PHP 中，字符串值和 `true` 布尔值之间的比较总是 `true`（因为字符串包含值），因此通过向 `unserialize()` 函数提供以下字符串，可以绕过认证控制并以管理员身份登录，管理员的 `userid` 为 2：

```php
a:2:{s:11:"autologinid";b:1;s:6:"userid";s:1:"2";}  // original value: a:2:{s:11:"autologinid";s:32:"8b8e9715d12e4ca12c4c3eb4865aaf6a";s:6:"userid";s:4:"1337";}
```

让我们分解一下我们在这个字符串中所做的工作：

1. `autologinid` 现在是一个设置为 `true` 的布尔值：这可以通过将密码哈希的 MD5 值（`s:32:"8b8e9715d12e4ca12c4c3eb4865aaf6a"`）替换为 `b:1` 看出。
2. `userid` 现在设置为管理员 ID：这可以从字符串的最后一部分看出，我们将普通用户 ID（`s:4:"1337"`）替换为 `s:1:"2"`。

## 工具

- [WebGoat](https://owasp.org/www-project-webgoat/)
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)

## 参考资料

- [Niels Teusink：phpBB 2.0.12 认证绕过](http://blog.teusink.net/2008/12/classic-bug-phpbb-2012-authentication.html)
- [David Endler："会话 ID 暴力破解利用与预测"](https://www.cgisecurity.com/lib/SessionIDs.pdf)
