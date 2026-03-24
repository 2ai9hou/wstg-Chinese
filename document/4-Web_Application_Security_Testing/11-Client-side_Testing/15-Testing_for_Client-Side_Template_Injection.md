# 测试客户端模板注入

|ID          |
|------------|
|WSTG-CLNT-15|

## 概述

客户端模板注入（CSTI），也称为基于DOM的模板注入，发生在使用客户端框架（如Angular、Vue.js或Alpine.js）的应用程序动态将用户输入嵌入到网页DOM中时。如果此输入被嵌入到模板表达式中或被框架的模板引擎解释，攻击者可以注入恶意指令。

与[服务器端模板注入（SSTI）](../07-Input_Validation_Testing/18-Testing_for_Server-side_Template_Injection.md)（其中模板在服务器上呈现）不同，CSTI完全在用户浏览器中发生。当框架扫描DOM中的动态内容时，它可能执行注入的模板表达式。这通常导致跨站脚本（XSS），但注入和利用方法与标准XSS不同，因为有效载荷必须遵循模板引擎的特定语法（例如，`{% raw %}`{{ 7*7 }}{% endraw %}`）。

这种漏洞在单页应用程序（SPAs）中特别常见，开发人员可能依赖客户端渲染而没有严格的上下文分离。

## 测试目标

- 识别应用程序使用的客户端框架及其版本。
- 检测用户输入被反映到DOM中并由模板引擎处理的注入点。
- 评估注入是否允许通过模板语法执行任意JavaScript（XSS）。

## 如何测试

### 黑盒测试

#### 框架识别

第一步是识别是否使用客户端框架。测试人员应查找HTML源代码中的特定属性或特定HTTP响应头。

- **Angular：** 查找`ng-app`、`ng-model`或`ng-bind`等属性。
- **Vue.js：** 查找以`v-`开头的属性，如`v-if`、`v-for`，或在控制台中查找Vue全局对象。
- **Alpine.js：** 查找`x-data`、`x-html`。

#### 注入检测

为了检测CSTI，测试人员应注入对模板引擎有句法意义的字符。许多框架最常见的插值语法是双花括号`{% raw %}`{{ }}{% endraw %}`。

简单的算术运算是标准探测。如果应用程序计算数学，则是易受攻击的。

**通用探测：**

{% raw %}

```txt
Inject the string: {{ 7*7 }}
```

{% endraw %}

- 如果应用程序呈现`49`，则存在CSTI。
- 如果应用程序呈现`{% raw %}`{{ 7*7 }}{% endraw %}`，则可能不易受攻击或存在严格的上下文转义。

#### Angular

Angular作用于DOM。如果攻击者可以将包含Angular表达式的字符串注入到DOM中（在Angular引导或编译之前），则该表达式将被执行。

**检测有效载荷：**

{% raw %}

- `{{ 7*7 }}`
- `{{ constructor.constructor('alert(1)')() }}`

{% endraw %}

在较旧版本的Angular（1.x）中，模板引擎在沙箱中工作。利用需要突破此沙箱。有效载荷的复杂性很大程度上取决于特定版本。

**示例有效载荷（Angular 1.5.x沙箱绕过）：**

{% raw %}

```javascript
{{x={'y':''.constructor.prototype};x['y'].charAt=[].join;$eval('x=alert(1)');}}
```

{% endraw %}

#### Vue.js

如果开发人员将用户输入与`v-html`指令一起使用，或者将Vue实例挂载到已包含用户控制HTML的DOM元素上，Vue.js也容易受到攻击。

**检测有效载荷：**

{% raw %}

- `{{ 7*7 }}`

{% endraw %}

**示例有效载荷（Vue.js 2.x）：**

{% raw %}

```javascript
{{_v.constructor('alert(1)')()}}
```

{% endraw %}

#### Alpine.js

Alpine.js严重依赖DOM属性。如果攻击者可以控制属性名称或注入到指令中，他们可以执行代码。

**示例有效载荷：**

```html
<div x-data="" x-html="'<img src=x onerror=alert(1)>'"></div>
```

### 灰盒测试

#### 代码审查

在灰盒场景中，测试人员验证用户输入在前端代码中的处理方式。关键是识别"接收器"（解释HTML或模板语法的位置）与"污染源"（用户输入）的组合使用位置。

**Angular接收器：**
搜索`$compile`或`ng-bind-html`的使用。
如果使用`ng-bind-html`，检查`$sce`（严格上下文转义）是否正确配置，或者`$sce.trustAsHtml()`是否用于不受信任的数据。

```javascript
// Angular中的易受攻击示例
$scope.htmlSnippet = $sce.trustAsHtml(userInput);
```

**Vue.js接收器：**
搜索`v-html`指令。在用户提供的内容上使用`v-html`是Vue中CSTI/XSS的主要原因。

```html
<div v-html="userProvidedComment"></div>
```

**React接收器：**
虽然React通常在模板注入方面更安全，因为它不扫描DOM模板（JSX被编译），但不当使用`dangerouslySetInnerHTML`允许基于DOM的XSS，这是此处讨论的风险配置文件的React等价物。

{% raw %}

```javascript
// React中的易受攻击示例
<div dangerouslySetInnerHTML={{__html: userContent}} />
```

{% endraw %}

#### 配置审查

特别检查内容安全策略（CSP）头。强大的CSP可以通过限制可以从中加载脚本的位置或防止内联脚本（包括模板引擎生成的脚本）执行来缓解CSTI的影响。

查找CSP中的`unsafe-eval`。许多模板引擎（特别是较旧的）需要`unsafe-eval`来动态编译模板。如果存在`unsafe-eval`，CSTI攻击要容易利用得多。

## 修复

- **避免将用户输入呈现为HTML：** 尽可能使用仅将输入视为文本的安全指令。
    - Angular：使用`ng-bind`而不是`ng-bind-html`。
    - Vue：使用`v-text`或花括号`{% raw %}`{{ }}{% endraw %}`（自动转义HTML）而不是`v-html`。
    - React：避免`dangerouslySetInnerHTML`。
- **清理：** 如果需要HTML渲染，使用专用清理库（如DOMPurify）在将数据传递到框架之前剥离危险标签和属性。
- **内容安全策略（CSP）：** 实施禁用`unsafe-eval`并限制脚本来源的严格CSP。
- **使用离线编译：** 对于Vue或React等框架，首选使用构建步骤（webpack、vite）提前编译模板，而不是使用解析DOM内容的运行时编译器。

## 工具

- [DOMPurify](https://github.com/cure53/DOMPurify)：一个仅DOM、超快速、超宽容的XSS清理器，用于HTML、MathML和SVG。

## 参考资料

### 白皮书和文章

- [PortSwigger：客户端模板注入](https://portswigger.net/research/server-side-template-injection)
- [Gareth Heyes：Angular沙箱绕过](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes)
- [Vue.js安全指南](https://vuejs.org/guide/best-practices/security.html)
- [Angular安全指南](https://angular.io/guide/security)
- [{% raw %}`{{alert('CSTI')}}{% endraw %}`：客户端模板注入的大规模检测](https://ieeexplore.ieee.org/document/11352459)
