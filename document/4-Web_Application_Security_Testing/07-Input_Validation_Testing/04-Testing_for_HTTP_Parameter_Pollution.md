# 测试HTTP参数污染

|ID          |
|------------|
|WSTG-INPV-04|

## 概述

HTTP参数污染测试应用程序在收到具有相同名称的多个HTTP参数时的响应；例如，如果参数`username`在GET或POST参数中包含两次。

提供具有相同名称的多个HTTP参数可能导致应用程序以意外方式解释值。通过利用这些效果，攻击者可能能够绕过输入验证、触发应用程序错误或修改内部变量值。由于HTTP参数污染（简称*HPP*）影响所有Web技术的构建块，因此存在服务器端和客户端攻击。

当前HTTP标准没有关于如何解释具有相同名称的多个输入参数的指导。例如，[RFC 3986](https://www.ietf.org/rfc/rfc3986.txt)简单地将*查询字符串*定义为一系列字段值对，[RFC 2396](https://www.ietf.org/rfc/rfc2396.txt)定义了反向和未保留查询字符串字符类。在没有标准的情况下，Web应用程序组件以各种方式处理此边缘情况（有关详细信息，请参阅下表）。

仅此一项并不一定表示存在漏洞。但是，如果开发人员不知道该问题，重复参数的存在可能导致应用程序产生异常行为，攻击者可能利用此行为。如同安全领域常见情况，意外行为通常是可能导致HTTP参数污染攻击的弱点的常见来源。为了更好地介绍此类漏洞以及HPP攻击的结果，分析过去发现的一些真实示例很有趣。

### 输入验证和过滤器绕过

2009年，在HTTP参数污染首次研究发布后，该技术引起安全社区的关注，成为绕过Web应用程序防火墙的可能方式。

其中一个缺陷，影响*ModSecurity SQL注入核心规则*，代表了应用程序和过滤器之间阻抗不匹配的完美示例。ModSecurity过滤器将正确地对以下字符串应用拒绝列表：`select 1,2,3 from table`，从而阻止此示例URL由Web服务器处理：`/index.aspx?page=select 1,2,3 from table`。但是，通过利用多个HTTP参数的连接，攻击者可能导致应用程序服务器在ModSecurity过滤器已接受输入后将字符串连接起来。例如，URL `/index.aspx?page=select 1&page=2,3 from table`不会触发ModSecurity过滤器，但应用程序层会将输入连接回完整的恶意字符串。

另一个HPP漏洞影响*Apple Cups*，这是许多Unix系统使用的知名打印系统。利用HPP，攻击者可以轻松使用以下URL触发跨站脚本漏洞：`https://127.0.0.1:631/admin/?kerberos=onmouseover=alert(1)&kerberos`。可以通过添加具有有效字符串（例如空字符串）的额外`kerberos`参数来绕过应用程序验证检查点。由于验证检查点仅考虑第二次出现，因此第一个`kerberos`参数在用于生成动态HTML内容之前未正确清理。成功利用将导致JavaScript代码在托管站点的上下文中执行。

### 身份验证绕过

在* Blogger*（一种流行的博客平台）发现了更关键的HPP漏洞。该错误允许恶意用户通过使用以下HTTP请求（`https://www.blogger.com/add-authors.do`）夺取受害者博客的所有权：

```html
POST /add-authors.do HTTP/1.1
[...]

security_token=attackertoken&blogID=attackerblogidvalue&blogID=victimblogidvalue&authorsList=goldshlager19test%40gmail.com(attacker email)&ok=Invite
```

该缺陷存在于Web应用程序使用的身份验证机制中，因为安全检查在第一个`blogID`参数上执行，而实际操作使用第二次出现。

### 应用程序服务器的预期行为

下表说明了不同Web技术在存在相同HTTP参数的多次出现时的行为。

给定URL和查询字符串：`https://example.com/?color=red&color=blue`

  | Web应用程序服务器后端 | 解析结果 | 示例 |
  |--------------------------------|----------------|--------|
  | ASP.NET / IIS | 所有出现连接与逗号 |  color=red,blue |
  | ASP / IIS     | 所有出现连接与逗号 | color=red,blue |
  | .NET Core 3.1 / Kestrel | 所有出现连接与逗号 | color=red,blue |
  | .NET 5 / Kestrel | 所有出现连接与逗号 | color=red,blue |
  | PHP / Apache  | 仅最后一次出现 | color=blue |
  | PHP / Zeus | 仅最后一次出现 | color=blue |
  | JSP, Servlet / Apache Tomcat | 仅第一次出现 | color=red |
  | JSP, Servlet / Oracle Application Server 10g | 仅第一次出现 | color=red |
  | JSP, Servlet / Jetty  | 仅第一次出现 | color=red |
  | IBM Lotus Domino | 仅最后一次出现 | color=blue |
  | IBM HTTP Server | 仅第一次出现 | color=red |
  | Node.js / express | 仅第一次出现 | color=red |
  | mod_perl, libapreq2 / Apache | 仅第一次出现 | color=red |
  | Perl CGI / Apache | 仅第一次出现 | color=red |
  | mod_wsgi (Python) / Apache | 仅第一次出现 | color=red |
  | Python / Zope | 列表数据类型中的所有出现 | color=['red','blue'] |

（来源：Appsec EU 2009 Carettoni & Paola）

## 测试目标

- 识别后端和使用的解析方法。
- 评估注入点并尝试使用HPP绕过输入过滤器。

## 如何测试

幸运的是，由于HTTP参数的分配通常由Web应用程序服务器处理，而不是应用程序代码本身，因此测试参数污染的响应应该在所有页面和操作中是标准的。但是，由于需要深入的业务逻辑知识，测试HPP需要手动测试。自动工具只能部分帮助审计员，因为它们往往会产生太多误报。此外，HPP可能表现在客户端和服务器端组件中。

### 服务器端HPP

要测试HPP漏洞，识别任何允许用户输入的表单或操作。HTTP GET请求中的查询字符串参数很容易在浏览器导航栏中调整。如果表单操作通过POST提交数据，测试人员需要使用拦截代理在POST数据发送到服务器时进行篡改。识别要测试的特定输入参数后，可以通过拦截请求编辑GET或POST数据，或在响应页面加载后更改查询字符串。要测试HPP漏洞，只需将同名参数附加到GET或POST数据，但分配不同的值。

例如：如果测试查询字符串中的`search_string`参数，请求URL将包含该参数名称和值：

```text
https://example.com/?search_string=kittens
```

特定参数可能隐藏在其他几个参数中，但方法相同；保留其他参数并附加重复参数：

```text
https://example.com/?mode=guest&search_string=kittens&num_results=100
```

使用不同的值附加同名参数：

```text
https://example.com/?mode=guest&search_string=kittens&num_results=100&search_string=puppies
```

对于基于JSON的端点，payload将如下所示：

```http
POST /search HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "search_string": "kittens",
  "search_string": "puppies"
}
```

并提交新请求。

分析响应页面以确定解析了哪些值。在上面的示例中，搜索结果可能显示`kittens`、`puppies`、两者的某种组合（`kittens,puppies`或`kittens~puppies`或`['kittens','puppies']`）、可能给出空结果或错误页面。

这种行为（是否使用具有相同名称的第一个、最后一个或组合输入参数）很可能在整个应用程序中一致。无论此默认行为是否显示潜在漏洞，都取决于特定应用程序的特定输入验证和过滤。一般来说：如果现有输入验证和其他安全机制对单个输入足够，并且如果服务器仅分配第一个或最后一个污染参数，则参数污染不会显示漏洞。如果重复参数被连接、不同的Web应用程序组件使用不同的出现或测试产生错误，则更有可能使用参数污染来触发安全漏洞。

更深入的分析需要对每个HTTP参数进行三次HTTP请求：

1. 提交包含标准参数名称和值的HTTP请求，并记录HTTP响应。例如`page?par1=val1`
2. 将参数值替换为篡改值，提交并记录HTTP响应。例如`page?par1=HPP_TEST1`
3. 发送组合步骤（1）和（2）的新请求。再次保存HTTP响应。例如`page?par1=val1&par1=HPP_TEST1`
4. 比较前几步获得的所有响应。如果（3）的响应不同于（1），并且（3）的响应也不同于（2），则存在可能被滥用来触发HPP漏洞的阻抗不匹配。

从参数污染弱点构建完整利用超出了本文的范围。请参阅参考资料中的示例和详细信息。

### 客户端HPP

与服务器端HPP类似，手动测试是审计Web应用程序以检测影响客户端组件的参数污染漏洞的唯一可靠技术。在服务器端变体中，攻击者利用易受攻击的Web应用程序访问受保护数据或执行不允许或不应执行的操作，而客户端攻击旨在颠覆客户端组件和技术。

要测试HPP客户端漏洞，识别任何允许用户输入并向用户显示该输入结果的表单或操作。搜索页面是理想的，但登录框可能不起作用（因为它可能不会将无效用户名显示回给用户）。

与服务器端HPP类似，使用`%26HPP_TEST`污染每个HTTP参数，并查找用户提供的payload的*URL解码*出现：

- `&HPP_TEST`
- `&amp;HPP_TEST`
- 等等。

特别要注意在`data`、`src`、`href`属性或表单操作中包含HPP向量的响应。同样，此默认行为是否显示潜在漏洞取决于特定的输入验证、过滤和应用程序业务逻辑。此外，重要的是要注意此漏洞也可能影响XMLHttpRequest（XHR）中使用的查询字符串参数、运行时属性创建和其他插件技术（例如Adobe Flash的flashvars变量）。

## 工具

- [ZAP被动/主动扫描器](https://www.zaproxy.org)

## 参考资料

### 白皮书

- [HTTP参数污染 - Luca Carettoni, Stefano di Paola](https://www.acunetix.com/websitesecurity/HTTP-Parameter-Pollution-WhitePaper.pdf)
- [客户端HTTP参数污染示例（Yahoo!经典邮件缺陷） - Stefano di Paola](https://blog.mindedsecurity.com/2009/05/client-side-http-parameter-pollution.html)
- [如何检测HTTP参数污染攻击 - Chrysostomos Daniel](https://www.acunetix.com/blog/whitepaper-http-parameter-pollution/)
- [CAPEC-460：HTTP参数污染（HPP） - Evgeny Lebanidze](https://capec.mitre.org/data/definitions/460.html)
- [Web应用程序HTTP参数污染漏洞的自动发现 - Marco Balduzzi, Carmen Torrano Gimenez, Davide Balzarotti, Engin Kirda](https://s3.eurecom.fr/docs/ndss11_hpp.pdf)
