# 测试基于DOM的跨站脚本

|ID          |
|------------|
|WSTG-CLNT-01|

## 概述

[基于DOM的跨站脚本](https://owasp.org/www-community/attacks/DOM_Based_XSS)是[XSS](https://owasp.org/www-community/attacks/xss/)漏洞的事实名称，这些漏洞是页面活动浏览器端内容（通常是JavaScript）通过[源](https://github.com/wisec/domxsswiki/wiki/sources)获取用户输入并将其用于[接收器](https://github.com/wisec/domxsswiki/wiki/Sinks)，导致注入代码执行的结果。本文档仅讨论导致XSS的JavaScript漏洞。

DOM或[文档对象模型](https://en.wikipedia.org/wiki/Document_Object_Model)是用于在浏览器中表示文档的结构格式。DOM使动态脚本（如JavaScript）能够引用文档的组件，如表单字段或会话cookie。DOM也被浏览器用于安全——例如，限制不同域的脚本获取其他域的会话Cookie。当活动内容（如JavaScript函数）被特别制作的请求修改时（如DOM元素可被攻击者控制），可能发生基于DOM的XSS漏洞。

并非所有XSS漏洞都需要攻击者控制从服务器返回的内容，而是可以滥用不良JavaScript编码实践来实现相同结果。后果与典型XSS缺陷相同，只是交付方式不同。

与其他类型的跨站脚本漏洞（[反射型和存储型](https://owasp.org/www-community/attacks/xss/)，其中未清理的参数由服务器传递然后返回给用户并在用户浏览器上下文中执行）相比，基于DOM的XSS漏洞通过使用文档对象模型（DOM）元素和攻击者精心制作的代码来控制代码流程。

由于其性质，基于DOM的XSS漏洞可以在许多实例中执行，而服务器无法确定实际执行了什么。这可能使许多通用XSS过滤和检测技术对这种攻击无效。

这个假设示例使用以下客户端代码：

```html
<script>
document.write("Site is at: " + document.location.href + ".");
</script>
```

攻击者可能附加`#<script>alert('xss')</script>`到受影响的页面URL，当执行时，将显示警报框。在此示例中，附加的代码不会发送到服务器，因为`#`字符后的所有内容都被浏览器视为片段而不是查询的一部分。在本例中，代码立即执行，页面显示"xss"警报。与更常见的跨站脚本类型（[反射型和存储型](https://owasp.org/www-community/attacks/xss/)，其中代码被发送到服务器然后返回浏览器）不同，这在用户浏览器中直接执行，无需联系服务器。

基于DOM的XSS漏洞的后果与更知名形式的XSS一样广泛，包括Cookie检索、进一步的恶意脚本注入等，因此应与相同严重性对待。

## 测试目标

- 识别DOM接收器。
- 构建适用于每种接收器类型的有效载荷。

## 如何测试

JavaScript应用程序与其他类型的应用程序显著不同，因为它们通常由服务器动态生成。要了解正在执行的代码，需要爬网被测试的网站以确定所有正在执行的JavaScript实例以及接受用户输入的位置。许多网站依赖大型函数库，这些函数库通常跨越数万行代码，并非内部开发。在这些情况下，自上而下测试通常成为唯一可行的选择，因为许多底层函数从未被使用，分析它们以确定哪些是接收器将花费比可用时间更多的时间。如果未识别输入或其缺乏，自上而下测试也是如此。

用户输入有两种主要形式：

- 以不允许直接XSS的方式由服务器写入页面的输入，以及
- 从客户端JavaScript对象获取的输入。

这里是服务器可能插入JavaScript的两个示例：

```js
var data = "<escaped data from the server>";
var result = someFunction("<escaped data from the server>");
```

这里是客户端JavaScript对象输入的两个示例：

```js
var data = window.location;
var result = someFunction(window.referrer);
```

虽然对JavaScript代码如何检索它们几乎没有区别，但重要的是要注意，当通过服务器接收输入时，服务器可以对数据应用任何排列。另一方面，JavaScript对象执行的排列是相当好理解和记录的。如果上述示例中的`someFunction`是接收器，那么在前一种情况下，可利用性取决于服务器的过滤，而在后一种情况下，取决于浏览器对`window.referrer`对象执行的编码。Stefano Di Paulo写了一篇关于浏览器在询问各种[使用文档和位置属性的URL元素时返回什么的出色文章](https://github.com/wisec/domxsswiki/wiki/location,-documentURI-and-URL-sources)。

此外，JavaScript经常在`<script>`块之外执行，这已被证据表明过去导致XSS过滤器绕过的许多向量。在爬网应用程序时，注意在事件处理程序和具有expression属性的CSS块等地方使用脚本。还请注意，任何场外CSS或脚本对象都需要评估正在执行的代码。

自动测试仅在识别和验证基于DOM的XSS方面取得非常有限的成功，因为它通常通过发送特定有效载荷并尝试在服务器响应中观察它来识别XSS。这对下面提供的简单示例很有效，其中消息参数被反射回用户：

```html
<script>
var pos=document.URL.indexOf("message=")+5;
document.write(document.URL.substring(pos,document.URL.length));
</script>
```

但是，在以下人为示例中可能无法检测到：

```html
<script>
var navAgt = navigator.userAgent;

if (navAgt.indexOf("MSIE")!=-1) {
        document.write("You are using IE as a browser and visiting site: " + document.location.href + ".");
}
else
{
    document.write("You are an unknown browser.");
}
</script>
```

因此，除非测试工具能够对客户端代码执行额外分析，否则自动测试不会检测可能容易受到基于DOM的XSS影响的区域。

因此，应进行手动测试，并且可以通过检查代码中对可能对攻击者有用的参数的引用区域来完成。此类区域的示例包括动态写入页面的代码位置以及其他修改DOM甚至直接执行脚本的位置。

## 修复

有关防止基于DOM的XSS的措施，请参阅[基于DOM的XSS预防速查表](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)。

## 参考资料

- [DomXSSWiki](https://github.com/wisec/domxsswiki/wiki/)
