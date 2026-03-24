# 测试JavaScript执行

|ID          |
|------------|
|WSTG-CLNT-02|

## 概述

JavaScript注入漏洞是跨站脚本（XSS）的一种子类型，涉及注入任意JavaScript代码的能力，这些代码由应用程序在受害者浏览器中执行。此漏洞可能有许多后果，例如泄露用户的会话Cookie，这些Cookie可用于模拟受害者，或者更一般情况下，它允许攻击者修改受害者看到的页面内容或应用程序的行为。

当应用程序缺乏适当的用户提供的输入和输出验证时，可能发生JavaScript注入漏洞。由于JavaScript用于动态填充网页，因此在内容处理阶段发生此注入，并因此影响受害者。

测试此漏洞时，请注意某些字符在不同浏览器中的处理方式不同。请参阅[基于DOM的XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)。

这是不执行任何变量`rr`验证的脚本示例。该变量通过查询字符串接收用户提供的输入，此外不应用任何形式的编码：

```js
var rr = location.search.substring(1);
if(rr) {
    window.location=decodeURIComponent(rr);
}
```

这意味着攻击者可以简单通过提交以下查询字符串来注入JavaScript代码：`www.victim.com/?javascript:alert(1)`。

## 测试目标

- 识别接收器和可能的JavaScript注入点。

## 如何测试

考虑以下内容：[DOM XSS练习](http://www.domxss.com/domxss/01_Basics/04_eval.html)

该页面包含以下脚本：

```html
<script>
function loadObj(){
    var cc=eval('('+aMess+')');
    document.getElementById('mess').textContent=cc.message;
}

if(window.location.hash.indexOf('message')==-1) {
    var aMess='({"message":"Hello User!"})';
} else {
    var aMess=location.hash.substr(window.location.hash.indexOf('message=')+8)
}
</script>
```

上述代码包含一个可由攻击者控制的源`location.hash`，该源可以直接注入到`message`值中的JavaScript代码中，以获取对用户浏览器的控制。
