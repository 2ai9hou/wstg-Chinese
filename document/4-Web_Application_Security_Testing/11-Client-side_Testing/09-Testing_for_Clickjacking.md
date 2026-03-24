# 测试点击劫持

|ID          |
|------------|
|WSTG-CLNT-09|

## 概述

点击劫持是UI覆盖的子集，是一种恶意技术，Web用户被欺骗与（大多数情况下通过点击）而非他们认为正在交互的内容进行交互。这种攻击单独或与其他攻击结合，可能在用户与看似无害的网页交互时发送未授权命令或泄露机密信息。点击劫持一词由Jeremiah Grossman和Robert Hansen于2008年创造。

点击劫持攻击使用看似无害的HTML和JavaScript功能，迫使受害者在不知道的情况下执行意外操作，如点击无意中显示的攻击者控制的页面上的不可见按钮。这是一个影响各种浏览器和平台的客户端安全问题。

为了执行此攻击，攻击者创建一个看似无害的网页，使用CSS代码隐藏的iframe加载目标应用程序。一旦完成，攻击者可能通过社交工程等方式诱导受害者与网页交互。像其他攻击一样，常见前提是受害者已针对攻击者的目标应用程序进行身份验证。

![点击劫持插图](images/Clickjacking_description.png)\
*图4.11.9-1：点击劫持iframe插图*

受害者浏览攻击者的网页，意图与可见用户界面交互，但实际上正在对隐藏的网页执行操作。使用隐藏页面，攻击者可以欺骗用户执行他们从未打算执行的操作，通过在网页中定位隐藏元素。

![隐藏iframe插图](images/Masked_iframe.png)\
*图4.11.9-2：隐藏iframe插图*

此方法的力量在于受害者的操作源自隐藏但真实的目标网页。因此，开发人员为保护网页免受CSRF攻击而部署的一些反CSRF保护可能被绕过。

## 测试目标

- 评估应用程序对点击劫持攻击的脆弱性。

## 如何测试

如上所述，这种类型的攻击通常被设计为允许攻击者在目标站点上诱导用户操作，即使正在使用反CSRF令牌。

### 使用HTML iframe标签加载目标网页到HTML解释器的测试

未受frame busting保护的站点容易受到点击劫持攻击。如果`https://www.target.site`网页成功加载到帧中，则该站点容易受到点击劫持。创建此测试网页的HTML代码示例如下：

```html
<html>
    <head>
        <title>Clickjack test web page</title>
    </head>
    <body>
        <iframe src="https://www.target.site" width="400" height="400"></iframe>
    </body>
</html>
```

### 针对禁用JavaScript的应用程序测试

由于这些类型的客户端保护依赖于JavaScript frame busting代码，如果受害者禁用JavaScript或者攻击者能够禁用JavaScript代码，网页将没有任何针对点击劫持的保护机制。

有几种可用于帧的技术去激活方法。更深入的技术可在[点击劫持防御速查表](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html)上找到。

### Sandbox属性

使用HTML5，一个名为"sandbox"的新属性可用。它对加载到iframe中的内容启用一组限制。

示例：

```html
<iframe src="https://example.org" sandbox></iframe>
```

### 在兼容性和可访问性模式下测试应用程序

网页的移动版本通常比桌面版本更小更快，而且必须比主应用程序更简单。移动变体通常保护较少。但是，攻击者可以伪造Web浏览器给出的真实来源，非移动受害者可能能够访问为移动用户制作的应用程序。此场景可能允许攻击者利用网页的移动版本。
还应在可访问性模式下测试应用程序的点击劫持，因为站点分帧可能受影响。

### 服务器端保护：使用内容安全策略的frame-ancestors指令

HTTP Content-Security-Policy（CSP）响应头允许Web页面管理员控制用户代理允许为给定网页加载的资源。HTTP CSP中的`frame-ancestors`指令指定可以使用`<frame>`、`<iframe>`、`<object>`、`<embed>`或`<applet>`标签嵌入网页的可接受父级。

#### 测试内容安全策略响应头

- 使用浏览器，打开开发者工具并访问目标网页。导航到Network选项卡。
- 查找加载网页的请求。它应该与网页具有相同的域——通常是Network选项卡上的第一项。
- 点击文件后，会出现更多信息。查找200 OK响应代码。
- 向下滚动到Response Header部分。Content-Security-Policy部分指示采用的保护级别。

或者，查看网页源代码以在meta标签中找到Content-Security-Policy。WSTG在[Test for Content Security Policy](../02-Configuration_and_Deployment_Management_Testing/12-Test_for_Content_Security_Policy.md)上有详细信息。

##### 代理

Web代理以添加和剥离头而闻名。在Web代理剥离`X-FRAME-OPTIONS`头的情况下，站点会丢失其分帧保护。

##### 应用程序的移动版本

在这种情况下，因为`X-FRAME-OPTIONS` HTTP头必须在应用程序的每个页面上实现，开发人员可能没有保护移动版本的每个页面。

### 修复

- 有关防止点击劫持的措施，请参阅[点击劫持防御速查表](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html)。
- 有关点击劫持的互动实验，请访问[Port Swigger网页](https://portswigger.net/web-security/clickjacking)
- 有关ClickJacking的其他资源，请访问[OWASP社区](https://owasp.org/www-community/attacks/Clickjacking)

## 参考资料

- [OWASP点击劫持](https://owasp.org/www-community/attacks/Clickjacking)
- [维基百科点击劫持](https://en.wikipedia.org/wiki/Clickjacking)
- [Gustav Rydstedt, Elie Bursztein, Dan Boneh, and Collin Jackson: "破坏Frame Busting：点击劫持漏洞对流行网站的研究"](https://seclab.stanford.edu/websec/framebusting/framebust.pdf)
