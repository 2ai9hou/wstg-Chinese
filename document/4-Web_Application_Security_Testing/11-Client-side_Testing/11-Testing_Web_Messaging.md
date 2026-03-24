# 测试Web消息传递

|ID          |
|------------|
|WSTG-CLNT-11|

## 概述

Web消息传递（也称为[跨文档消息传递](https://html.spec.whatwg.org/multipage/web-messaging.html#web-messaging)）允许应用程序在不同域上以安全方式进行通信。在Web消息传递引入之前，由于同源策略，跨不同来源（iframe、选项卡和窗口之间）的通信受到限制，由浏览器强制执行。开发人员使用了多个hack来完成这些任务，其中大部分本质上是不安全的。

浏览器中的此限制是为了防止恶意网站从其他iframe、选项卡等读取机密数据；但是，在某些合法情况下，两个可信网站需要彼此交换数据。为了满足这一需求，跨文档消息传递在[WHATWG HTML5](https://html.spec.whatwg.org/multipage/)草案规范中引入，并在所有主流浏览器中实现。它能够在跨iframe、选项卡和窗口的多个来源之间进行安全通信。

消息API引入了[`postMessage()`方法](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)，通过它可以发送跨源的纯文本消息。它由两个参数组成：消息和域。

当使用`*`作为域时存在一些安全风险，我们将在下面讨论。为了接收消息，接收网站需要添加一个新的事件处理程序，它具有以下属性：

- Data，传入消息的内容；
- Origin，发送方文档的来源；和
- Source，源窗口。

这是正在使用的消息API的示例。发送消息：

```js
iframe1.contentWindow.postMessage("Hello world","https://www.example.com");
```

接收消息：

```js
window.addEventListener("message", handler, true);
function handler(event) {
    if(event.origin === 'chat.example.com') {
        /* process message (event.data) */
    } else {
        /* ignore messages from untrusted domains */
    }
}
```

### Origin安全

origin由方案、主机名和端口组成。它唯一标识发送或接收消息的域，不包括URL的路径或片段部分。例如，`https://example.com`将被视为与`http://example.com`不同，因为前者的方案是`https`，而后者是`http`。这同样适用于在同一域但不同端口上运行的Web服务器。

## 测试目标

- 评估消息来源的安全性。
- 验证它是否使用安全方法并验证其输入。

## 如何测试

### 检查Origin安全

测试人员应检查应用程序代码是否过滤和处理来自可信域的消息。在发送域中，还要确保明确指定了接收域，并且`postMessage()`的第二个参数未用作`*`。这种做法可能引入安全问题，如果通过重定向或origin以其他方式更改，网站可能会将数据发送到未知主机，从而将机密数据泄露给恶意服务器。

如果网站未能添加安全控制来限制允许向网站发送消息的域或来源，则很可能引入安全风险。测试人员应检查消息事件监听器的代码，并从`addEventListener`方法获取回调函数以进行进一步分析。域必须在数据操作之前始终验证。

### 检查输入验证

虽然网站在理论上仅接受来自可信域的消息，但数据仍必须被视为外部来源、不受信任的数据，并使用适当的安全控制进行处理。测试人员应分析代码并查找不安全方法，特别是数据通过`eval()`进行评估或通过`innerHTML`属性插入DOM的位置，这可能创建基于DOM的XSS漏洞。

### 静态代码分析

应分析JavaScript代码以确定Web消息传递的实现方式。特别是，测试人员应对以下内容感兴趣：网站如何限制来自不受信任域的消息，以及即使对于可信域，数据是如何处理的。

在此示例中，需要访问owasp.org域内的每个子域（www、chat、forums、...）。代码试图接受带有`.owasp.org`的任何域：

```js
window.addEventListener("message", callback, true);

function callback(e) {
    if(e.origin.indexOf(".owasp.org")!=-1) {
        /* process message (e.data) */
    }
}
```

意图是允许子域，例如：

- `www.owasp.org`
- `chat.owasp.org`
- `forums.owasp.org`

不幸的是，这引入了漏洞。因为像`www.owasp.org.attacker.com`这样的域将匹配。

以下是没有origin检查的代码示例。这是非常不安全的，因为它将接受来自任何域的输入：

```js
window.addEventListener("message", callback, true);

function callback(e) {
        /* process message (e.data) */
}
```

以下是可能导致XSS攻击的输入验证漏洞示例：

```js
window.addEventListener("message", callback, true);

function callback(e) {
        if(e.origin === "trusted.domain.com") {
            element.innerHTML= e.data;
        }
}
```

更安全的方法是使用`innerText`属性而不是`innerHTML`。

有关Web消息传递的更多OWASP资源，请参阅[OWASP HTML5安全速查表](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)
