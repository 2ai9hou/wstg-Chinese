# 测试服务器端模板注入

|ID          |
|------------|
|WSTG-INPV-18|

## 概述

Web应用程序通常使用服务器端模板技术（Jinja2、Twig、FreeMaker等）来生成动态HTML响应。服务器端模板注入漏洞（SSTI）当用户输入以不安全的方式嵌入模板并导致服务器上的远程代码执行时发生。任何支持高级用户提供的标记的功能都可能容易受到SSTI影响，包括维基页面、评论、营销应用程序、CMS系统等。一些模板引擎采用各种机制（如沙箱、允许列表等）来防止SSTI。

### 示例 - Twig

以下示例摘自[极端易受攻击的Web应用程序](https://github.com/s4n7h0/xvwa)项目。

```php
public function getFilter($name)
{
        [snip]
        foreach ($this->filterCallbacks as $callback) {
        if (false !== $filter = call_user_func($callback, $name)) {
            return $filter;
        }
    }
    return false;
}
```

在getFilter函数中，`call_user_func($callback, $name)`容易受到SSTI：`name`参数从HTTP GET请求获取并由服务器执行：

![SSTI XVWA示例](images/SSTI_XVWA.jpeg)\
*图4.7.18-1：SSTI XVWA示例*

### 示例 - Flask/Jinja2

以下示例使用Flask和Jinja2模板引擎。`page`函数从HTTP GET请求接受`name`参数，并使用`name`变量内容渲染HTML响应：

```python
@app.route("/page")
def page():
    name = request.values.get('name')
    output = Jinja2.from_string('Hello ' + name + '!').render()
    return output
```

此代码片段容易受到XSS但也容易受到SSTI。使用以下作为`name`参数中的有效载荷：

```bash
$ curl -g 'https://www.target.com/page?name={{7*7}}'
Hello 49!
```

## 测试目标

- 检测模板注入漏洞点。
- 识别模板引擎。
- 构建漏洞利用。

## 如何测试

SSTI漏洞存在于文本或代码上下文中。在明文上下文中，允许用户使用直接HTML代码的自由格式"文本"。在代码上下文中，用户输入也可以放在模板语句中（例如，在变量名中）

### 识别模板注入漏洞

测试明文上下文中的SSTI的第一步是构造各种模板引擎使用的常见模板表达式作为有效载荷，并监控服务器响应以识别服务器执行了哪个模板表达式。

常见模板表达式示例：

```text
a{{bar}}b
a{{7*7}}
{var} ${var} {{var}} <%var%> [% var %]
```

在此步骤中，建议使用[模板表达式测试字符串/有效载荷列表](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)。

在代码上下文中测试SSTI略有不同。首先，测试人员构造导致空白或错误服务器响应的请求。在下面的示例中，HTTP GET参数插入到模板语句中的变量`personal_greeting`中：

```text
personal_greeting=username
Hello user01
```

使用以下有效载荷，服务器响应为空白"Hello"：

```text
personal_greeting=username<tag>
Hello
```

下一步是使用以下有效载荷跳出模板语句并在其后注入HTML标签

```text
personal_greeting=username}}<tag>
Hello user01 <tag>
```

### 识别模板引擎

根据上一步的信息，测试人员需要通过提供各种模板表达式来确定使用哪种模板引擎来识别它。测试人员根据服务器响应推断使用的模板引擎。此手动方法在[这篇](https://portswigger.net/blog/server-side-template-injection#Identify) PortSwigger文章中有更详细的讨论。用于自动识别SSTI漏洞和模板引擎的各种工具包括[Tplmap](https://github.com/epinna/tplmap)或[反斜杠驱动的扫描仪Burp Suite扩展](https://github.com/PortSwigger/backslash-powered-scanner)。

### 构建RCE漏洞利用

此步骤的主要目标是识别通过研究模板文档和研究来进一步控制服务器以获得RCE漏洞利用。关键感兴趣区域包括：

- **面向模板作者** 涵盖基本语法的部分。
- **安全注意事项** 部分。
- 内置方法、函数、过滤器和变量的列表。
- 扩展/插件列表。

测试人员还可以关注`self`对象，识别哪些其他对象、方法和属性可以暴露。如果`self`对象不可用且文档未显示技术细节，建议对变量名进行暴力破解。识别对象后，下一步是循环该对象以识别可以通过模板引擎访问的所有方法、属性和属性。这可能导致其他类型的安全发现，包括权限提升、信息泄露（关于应用程序密码、API密钥配置和环境变量等）。

## 工具

- [Tplmap](https://github.com/epinna/tplmap)
- [反斜杠驱动的扫描仪Burp Suite扩展](https://github.com/PortSwigger/backslash-powered-scanner)
- [模板表达式测试字符串/有效载荷列表](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)

## 参考资料

- [James Kettle：服务器端模板注入：现代Web应用的RCE（白皮书）](https://portswigger.net/kb/papers/serversidetemplateinjection.pdf)
- [服务器端模板注入](https://portswigger.net/blog/server-side-template-injection)
- [探索Flask/Jinja2中的SSTI](https://www.lanmaster53.com/2016/03/exploring-ssti-flask-jinja2/)
- [服务器端模板注入：从检测到远程shell](https://www.okiok.com/server-side-template-injection-from-detection-to-remote-shell/)
- [极端易受攻击的Web应用程序](https://github.com/s4n7h0/xvwa)
- [在Thymeleaf中利用SSTI](https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/)
