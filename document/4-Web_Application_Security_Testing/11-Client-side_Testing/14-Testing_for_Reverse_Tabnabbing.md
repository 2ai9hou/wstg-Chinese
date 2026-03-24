# 测试反向标签劫持

|ID          |
|------------|
|WSTG-CLNT-14|

## 概述

[反向标签劫持](https://owasp.org/www-community/attacks/Reverse_Tabnabbing)是一种可用于将用户重定向到钓鱼页面的攻击。这通常由于`<a>`标签的`target`属性设置为`_blank`而导致链接在新选项卡中打开而成为可能。当属性`rel='noopener noreferrer'`未在同一`<a>`标签中使用时，新打开的页面可以影响原始页面并将其重定向到攻击者控制的域。

由于用户在原始域上新选项卡打开时，他们不太可能注意到页面已更改，特别是如果钓鱼页面与原始域相似。用户在任何时候在攻击者控制的域上输入的凭据都将落入攻击者手中。

通过`window.open` JavaScript函数打开的链接也容易受到此攻击。

_注意：这是一个不影响[现代浏览器](https://caniuse.com/mdn-html_elements_a_implicit_noopener)的遗留问题。旧版流行浏览器（例如Google Chrome 88之前的版本）以及Internet Explorer容易受到此攻击。_

### 示例

想象一个允许用户在个人资料中插入URL的Web应用程序。如果应用程序容易受到反向标签劫持，恶意用户将能够提供指向具有以下代码的页面的链接：

```html
<html>
 <body>
  <script>
    window.opener.location = "https://example.org";
  </script>
<b>Error loading...</b>
 </body>
</html>
```

点击链接将打开新选项卡，而原始选项卡将重定向到"example.org"。假设"example.org"看起来类似于易受攻击的Web应用程序，用户不太可能注意到更改，更可能在页面上输入敏感信息。

## 如何测试

- 检查应用程序的HTML源代码，查看带有`target="_blank"`的链接是否在`rel`属性中使用`noopener`和`noreferrer`关键字。如果没有，应用程序可能容易受到反向标签劫持。如果链接指向已被攻击者破坏的第三方站点，或者是用户控制的，则此类链接可被利用。
- 检查攻击者可以插入链接的区域，即控制`<a>`标签的`href`参数的位置。尝试插入指向具有上述示例源代码的页面的链接，并查看原始域是否重定向。如果其他浏览器不起作用，此测试可以在IE中进行。

## 修复

建议确保所有链接的`rel` HTML属性都设置了`noreferrer`和`noopener`关键字。

## 参考资料

- [Tabnabbing - HTML5速查表](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#tabnabbing)
- [目标="_blank"漏洞示例](https://dev.to/ben/the-targetblank-vulnerability-by-example)
- [关于rel=noopener](https://mathiasbynens.github.io/rel-noopener/)
- [目标="_blank" — 最被低估的漏洞](https://medium.com/@jitbit/target-blank-the-most-underestimated-vulnerability-ever-96e328301f4c)
- [反向标签劫持漏洞影响IBM Business Automation Workflow和IBM Business Process Manager](https://www.ibm.com/support/pages/security-bulletin-reverse-tabnabbing-vulnerability-affects-ibm-business-automation-workflow-and-ibm-business-process-manager-bpm-cve-2020-4490-0)
