# 测试HTML注入

|ID          |
|------------|
|WSTG-CLNT-03|

## 概述

HTML注入是一种注入漏洞，当用户能够控制输入点并能够将任意HTML代码注入到易受攻击的网页时会发生。这种漏洞可能有许多后果，例如泄露用户的会话Cookie，这些Cookie可用于模拟受害者，或者更一般情况下，它允许攻击者修改受害者看到的页面内容。

当用户输入未正确清理且输出未编码时，会发生此漏洞。注入允许攻击者向受害者发送恶意HTML页面。目标浏览器将无法区分（信任）页面的合法部分和恶意部分，因此将在受害者上下文中解析和执行整个页面。

有许多方法和属性可用于呈现HTML内容。如果向这些方法提供不受信任的输入，则存在HTML注入漏洞的高风险。例如，恶意HTML代码可以通过`innerHTML` JavaScript方法注入，该方法通常用于呈现用户插入的HTML代码。如果字符串未正确清理，该方法可以启用HTML注入。`document.write()`是可用于此目的的JavaScript函数。

以下示例显示了一段易受攻击的代码，该代码允许使用未验证的输入在页面上下文中创建动态HTML：

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.getElementById("Welcome").innerHTML=" Hello, "+user;
```

以下示例显示使用`document.write()`函数的易受攻击代码：

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.write("<h1>Hello, " + user +"</h1>");
```

在这两个示例中，可以使用以下输入来利用此漏洞：

```text
https://vulnerable.site/page.html?user=<img%20src='aaa'%20onerror=alert(1)>
```

此输入将向页面添加一个图像标签，该标签将执行恶意用户在HTML上下文中插入的任意JavaScript代码。

## 测试目标

- 识别HTML注入点并评估注入内容的严重性。

## 如何测试

考虑以下DOM XSS练习<https://www.domxss.com/domxss/01_Basics/06_jquery_old_html.html>

HTML代码包含以下脚本：

```html
<script src="../js/jquery-1.7.1.js"></script>
<script>
function setMessage(){
    var t=location.hash.slice(1);
    $("div[id="+t+"]").text("The DOM is now loaded and can be manipulated.");
}
$(document).ready(setMessage  );
$(window).bind("hashchange",setMessage)
</script>
<body>
    <script src="../js/embed.js"></script>
    <span><a href="#message" > Show Here</a><div id="message">Showing Message1</div></span>
    <span><a href="#message1" > Show Here</a><div id="message1">Showing Message2</div>
    <span><a href="#message2" > Show Here</a><div id="message2">Showing Message3</div>
</body>
```

可以注入HTML代码。
