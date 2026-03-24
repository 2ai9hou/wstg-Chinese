# 测试跨站请求伪造

|ID          |
|------------|
|WSTG-SESS-05|

## 概述

跨站请求伪造（[CSRF](https://owasp.org/www-community/attacks/csrf)）是一种攻击，迫使最终用户在当前认证的Web应用中执行非预期的操作。通过一点社会工程帮助（如通过电子邮件或聊天发送链接），攻击者可能迫使Web应用的用户执行攻击者选择的操作。成功的CSRF漏洞利用可以危及最终用户的数据和操作（当目标是普通用户时）。如果目标最终用户是管理员账户，CSRF攻击可能危及整个Web应用。

CSRF依赖于：

1. 浏览器处理会话相关信息（如Cookie和HTTP认证信息）的行为。
2. 攻击者对有效Web应用URL、请求或功能的了解。
3. 应用仅依赖浏览器已知的信息进行会话管理。
4. 存在导致立即访问HTTP[S]资源的HTML标签；例如图像标签`img`。

第1、2和3点对于漏洞的存在至关重要，而第4点促进了实际利用，但不是严格必需的。

1. 浏览器自动发送用于识别用户会话的信息。假设*site*是托管Web应用的站点，用户*victim*刚刚在*site*上认证。作为回应，*site*向*victim*发送一个Cookie，用于识别由*victim*发送的请求属于*victim*的已认证会话。一旦浏览器收到*site*设置的Cookie，它将自动随后续指向*site*的任何请求一起发送。
2. 如果应用不使用URL中的会话相关信息，则可以识别应用URL、其参数和合法值。这可以通过代码分析或访问应用并记下HTML或JavaScript中嵌入的表单和URL来实现。
3. "浏览器已知"指的是Cookie或HTTP认证信息（如基本认证，而非表单认证）等信息，这些信息由浏览器存储，随后在面向需要该认证的应用区域的每个请求中出现。接下来讨论的漏洞适用于完全依赖此类信息来识别用户会话的应用。

为简单起见，考虑GET可访问的URL（尽管讨论也适用于POST请求）。如果*victim*已经认证了自己，提交另一个请求会导致Cookie自动随其发送。下图说明了用户访问`www.example.com`上应用的情况。

![会话劫持](images/Session_riding.GIF)\
*图4.6.5-1：会话劫持*

GET请求可以通过几种不同的方式由用户发送：

- 使用Web应用
- 直接在浏览器中输入URL
- 遵循指向URL的外部链接

这些调用对应用来说是无法区分的。特别是，第三种可能非常危险。有许多技术和漏洞可以伪装链接的真实属性。链接可以嵌入电子邮件消息中，出现在用户被诱骗到的恶意站点上，或出现在托管在第三方（如另一个站点或HTML电子邮件）的内容中，并指向应用的资源。如果用户点击链接，由于他们已经由*site*上的Web应用认证，浏览器将向Web应用发出GET请求，并附带认证信息（会话ID Cookie）。这将导致在用户非预期的Web应用上执行有效操作；例如，在Web银行应用上进行资金转移。

通过使用上述第4点中指定的`img`等标签，甚至不需要用户遵循特定链接。假设攻击者向用户发送一封电子邮件，诱使他们访问引用包含以下（过度简化）HTML的页面的URL。

```html
<html>
    <body>
...
<img src="https://www.company.example/action" width="0" height="0">
...
    </body>
</html>
```

当浏览器显示此页面时，它将尝试显示指定的来自`https://www.company.example`的零尺寸（因此不可见）图像。这会导致请求自动发送到托管在*site*上的Web应用。图像URL引用的是否是正确的图像并不重要，因为其存在将触发`src`字段中指定的`action`请求。这是在浏览器未禁用图像下载的情况下发生的。大多数浏览器不禁用图像下载，因为这会使大多数Web应用无法使用。

这里的问题是以下原因的后果：

- 页面上的HTML标签导致自动HTTP请求执行（`img`就是其中之一）。
- 浏览器没有办法知道`img`引用的资源不是合法的图像。
- 图像加载无论所谓图像源的位置如何都会发生，即表单和图像本身不需要位于同一主机上，甚至不需要位于同一域上。

HTML内容可以引用应用中的组件，而浏览器会自动向应用组成有效请求，这一事实允许这种攻击。除非使攻击者无法与应用功能交互成为不可能，否则无法禁止此行为。

在集成邮件/浏览器环境中，简单显示包含图像引用的电子邮件也会导致执行带有关联浏览器Cookie的对Web应用的请求。电子邮件可能引用看似有效的图像URL，例如：

```html
<img src="https://[attacker]/picture.gif" width="0" height="0">
```

在这个例子中，`[attacker]`是攻击者控制的站点。通过利用重定向机制，恶意站点可以使用`https://[attacker]/picture.gif`将受害者引导至`https://[thirdparty]/action`并触发`action`。

Cookie不是这种漏洞的唯一例子。仅由浏览器提供的会话信息的Web应用也容易受到攻击。这包括仅依赖HTTP认证机制的应用，因为认证信息由浏览器知道并随每个请求自动发送。这不包括基于表单的认证，它只发生一次并生成某种形式的会话相关信息，通常是Cookie。

让我们假设受害者登录了防火墙Web管理控制台。要登录，用户必须进行身份认证，会话信息存储在Cookie中。

让我们假设防火墙Web管理控制台有一个功能，允许认证用户删除由数字ID指定的规则，如果用户指定`*`，则删除配置中的所有规则（实际上这是一个危险的功能，但会使示例更有趣）。删除页面如下所示。让我们假设该表单为简单起见发出GET请求。要删除第一条规则：

```text
https://[target]/fwmgt/delete?rule=1
```

要删除所有规则：

```text
https://[target]/fwmgt/delete?rule=*
```

这个例子是故意天真的，但以简化的方式展示了CSRF的危险。

![会话劫持防火墙管理](images/Session_Riding_Firewall_Management.gif)\
*图4.6.5-2：会话劫持防火墙管理*

使用上图中的表单，输入值`*`并点击删除按钮将提交以下GET请求：

```text
https://www.company.example/fwmgt/delete?rule=*
```

这将删除所有防火墙规则。

![会话劫持防火墙管理2](images/Session_Riding_Firewall_Management_2.gif)\
*图4.6.5-3：会话劫持防火墙管理2*

用户也可以通过手动提交URL来实现相同的结果：

```text
https://[target]/fwmgt/delete?rule=*
```

或者通过遵循直接或通过重定向指向上述URL的链接。或者通过访问带有嵌入指向相同URL的`img`标签的HTML页面。

在所有这些情况下，如果用户当前登录到防火墙管理应用，请求将成功，并将修改防火墙的配置。可以想象针对敏感应用的攻击，自动进行拍卖出价、资金转移、订单、更改关键软件组件的配置等。

有趣的是，这些漏洞可以在防火墙后面进行；也就是说，链接被攻击的目标只需要被受害者访问，而不需要被攻击者直接访问。例如，它可以是任何内部网Web服务器；例如，之前提到的防火墙管理场景，不太可能暴露于互联网。

自我脆弱的应用，即同时用作攻击矢量和目标的应用（如Web邮件应用），使情况更糟。由于用户在阅读电子邮件时已登录，此类易受攻击的应用可以允许攻击者执行删除消息或发送看似来自受害者的消息等操作。

## 测试目标

- 确定是否可能代表用户发起非用户主动的请求。

## 如何测试

审查应用以确定其会话管理是否有漏洞。如果会话管理仅依赖客户端值（浏览器可用的信息），则应用存在漏洞。"客户端值"指的是Cookie和HTTP认证凭证（基本认证和其他形式的HTTP认证；非基于表单的认证，它是应用级认证）。

可通过HTTP GET请求访问的资源容易受到攻击，虽然POST请求可以通过JavaScript自动化，但也容易受到攻击；因此，单独使用POST不足以纠正CSRF漏洞的发生。

如果是POST，可以使用以下示例。

1. 创建一个类似于下面显示的HTML页面
2. 将HTML托管在恶意或第三方站点上
3. 将页面链接发送给受害者并诱导他们点击

```html
<html>
<body onload='document.CSRF.submit()'>

<form action='https://targetWebsite/Authenticate.jsp' method='POST' name='CSRF'>
    <input type='hidden' name='name' value='Hacked'>
    <input type='hidden' name='password' value='Hacked'>
</form>

</body>
</html>
```

对于使用JSON进行浏览器到服务器通信的Web应用，可能会出现一个问题，即JSON格式没有查询参数，而自我提交表单必须有查询参数。为了绕过这种情况，我们可以使用带有JSON负载的自我提交表单，包括隐藏输入来利用CSRF。我们必须将编码类型（`enctype`）更改为`text/plain`以确保负载按原样传递。攻击代码将如下所示：

```html
<html>
 <body>
  <script>history.pushState('', '', '/')</script>
   <form action='https://victimsite.com' method='POST' enctype='text/plain'>
     <input type='hidden' name='{"name":"hacked","password":"hacked","padding":"'value='something"}' />
     <input type='submit' value='Submit request' />
   </form>
 </body>
</html>
```

POST请求将如下所示：

```http
POST / HTTP/1.1
Host: victimsite.com
Content-Type: text/plain

{"name":"hacked","password":"hacked","padding":"=something"}
```

当此数据作为POST请求发送时，服务器将愉快地接受名称和密码字段，并忽略名称为padding的字段，因为它不需要它。

## 修复方案

- 有关预防措施，请参阅[OWASP CSRF预防速查表](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

## 工具

- [ZAP](https://www.zaproxy.org/)
- [CSRF Tester](https://wiki.owasp.org/index.php/Category:OWASP_CSRFTester_Project)
- [Pinata-csrf-tool](https://code.google.com/archive/p/pinata-csrf-tool/)

## 参考资料

- [Peter W: "Cross-Site Request Forgeries"](https://web.archive.org/web/20160303230910/http://www.tux.org/~peterw/csrf.txt)
- [Thomas Schreiber: "Session Riding"](https://web.archive.org/web/20160304001446/https://www.securenet.de/papers/Session_Riding.pdf)
- [Oldest known post](https://web.archive.org/web/20000622042229/https://www.zope.org/Members/jim/ZopeSecurity/ClientSideTrojan)
- [Cross-site Request Forgery FAQ](https://www.cgisecurity.com/csrf-faq.html)
- [A Most-Neglected Fact About Cross Site Request Forgery (CSRF)](https://yehg.net/lab/pr0js/view.php/A_Most-Neglected_Fact_About_CSRF.pdf)
- [Multi-POST CSRF](https://www.lanmaster53.com/2013/07/17/multi-post-csrf/)
- [SANS Pen Test Webcast: Complete Application pwnage via Multi POST XSRF](https://www.youtube.com/watch?v=EOs5PZiiwug)
