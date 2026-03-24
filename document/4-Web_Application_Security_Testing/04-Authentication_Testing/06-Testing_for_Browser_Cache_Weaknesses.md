# 测试浏览器缓存弱点

|ID          |
|------------|
|WSTG-ATHN-06|

## 概述

在这个阶段，测试人员检查应用程序是否正确指示浏览器不要保留敏感数据。

浏览器可以存储信息用于缓存和历史记录。缓存用于提高性能，这样先前显示的信息就不需要再次下载。历史机制用于用户便利，这样用户可以看到他们在获取资源时看到的确切内容。如果向用户显示了敏感信息（如他们的地址、信用卡详情、社会安全号码或用户名），则这些信息可能会被存储用于缓存或历史记录的目的，因此可以通过检查浏览器缓存或简单地按下浏览器的**后退**按钮来检索。

## 测试目标

- 审查应用程序是否在客户端存储敏感信息。
- 审查是否可以在未授权的情况下访问。

## 如何测试

### 浏览器历史

从技术上讲，**后退**按钮是历史记录而不是缓存（请参阅 [HTTP 缓存：历史列表](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.13)）。缓存和历史是两个不同的实体。然而，它们具有相同的弱点：显示先前显示的敏感信息。

第一个也是最简单的测试是将敏感信息输入应用程序，然后注销。然后测试人员点击浏览器的**后退**按钮，检查是否可以在未认证的情况下访问先前显示的敏感信息。

如果按下**后退**按钮，测试人员可以访问以前的页面但不能访问新页面，那么这不是认证问题，而是浏览器历史问题。如果这些页面包含敏感数据，则意味着应用程序没有禁止浏览器存储它。

认证不一定需要涉及测试。例如，当用户输入他们的电子邮件地址以订阅新闻通讯时，如果处理不当，这些信息可能是可检索的。

**后退**按钮可以被阻止显示敏感数据。这可以通过以下方式完成：

- 通过 HTTPS 传送页面。
- 设置 `Cache-Control: no-store`

`Cache-Control: no-store` 指示浏览器不要将响应存储在任何缓存或历史记录存储中，这有助于防止用户在注销后导航回来时访问敏感数据。

### 浏览器缓存

在这里，测试人员检查应用程序是否不会将任何敏感数据泄露到浏览器缓存中。为此，他们可以使用代理（如 ZAP）并搜索属于该会话的服务器响应，检查每个包含敏感信息的页面，服务器是否指示浏览器不要缓存任何数据。可以通过以下指令在 HTTP 响应头中发出此类指令：

- `Cache-Control: no-store`

当使用 `Cache-Control: no-store` 时，对于现代浏览器来说，通常不需要 `must-revalidate`、`max-age` 或 `Expires` 头等额外指令。

```http
HTTP/1.1
Cache-Control: no-store
```

例如，如果测试人员正在测试一个电子商务应用程序，他们应该查找所有包含信用卡号或其他财务信息的页面，并检查所有这些页面是否强制执行 `no-store` 指令。如果他们发现包含关键信息但未能指示浏览器不要缓存其内容的页面，则意味着敏感信息将存储在磁盘上，他们可以通过简单地在浏览器缓存中查找页面来确认这一点。

这些信息的确切存储位置取决于客户端操作系统和所使用的浏览器。以下是一些示例：

- Mozilla Firefox：
    - Unix/Linux：`~/.cache/mozilla/firefox/`
    - Windows：`C:\Users\<user_name>\AppData\Local\Mozilla\Firefox\Profiles\<profile-id>\Cache2\`
- Internet Explorer：
    - `C:\Users\<user_name>\AppData\Local\Microsoft\Windows\INetCache\`
- Chrome：
    - Windows：`C:\Users\<user_name>\AppData\Local\Google\Chrome\User Data\Default\Cache`
    - Unix/Linux：`~/.cache/google-chrome`

#### 审查缓存的信息

Firefox 提供了查看缓存信息的功能，这对测试人员可能有益。当然，行业也产生了各种扩展，以及您可能更喜欢或需要的 Chrome、Internet Explorer 或 Edge 的外部应用程序。

缓存详情也可以通过大多数现代浏览器的开发者工具获得，如 [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector#Cache_Storage)、[Chrome](https://developers.google.com/web/tools/chrome-devtools/storage/cache) 和 Edge。使用 Firefox，还可以使用 URL `about:cache` 检查缓存详情。

#### 检查移动浏览器的处理

移动浏览器的缓存指令处理可能完全不同。因此，测试人员应该使用干净的缓存启动新的浏览会话，并利用 Chrome 的[设备模式](https://developers.google.com/web/tools/chrome-devtools/device-mode)或 Firefox 的[响应式设计模式](https://developer.mozilla.org/en-US/docs/Tools/Responsive_Design_Mode)等功能来重新测试或单独测试上述概念。

此外，ZAP 和 Burp Suite 等个人代理允许测试人员指定蜘蛛/爬虫应发送的 `User-Agent`。这可以设置为匹配移动浏览器 `User-Agent` 字符串，并用于查看被测应用程序发送了哪些缓存指令。

### 灰盒测试

测试方法与黑盒情况相同，因为在这两种情况下，测试人员都可以完全访问服务器响应头和 HTML 代码。但是，通过灰盒测试，测试人员可能拥有允许他们测试仅对已认证用户可访问的敏感页面的账户凭据。

## 工具

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)

## 参考资料

### 白皮书

- [MDN – Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
- [Euthanize Pragma no-cache](https://www.veggiespam.com/euthanize-pragma-no-cache/)
- [HTTP 中的缓存](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)
