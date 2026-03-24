# 测试其他 HTTP 安全头配置错误

|ID         |
|------------|
|WSTG-CONF-14|

## 概述

安全头在保护 Web 应用程序免受各种攻击（包括跨站脚本（XSS）、点击劫持和数据注入攻击）方面起着至关重要的作用。这些头指示浏览器如何处理与网站通信相关的安全方面，降低已知攻击向量的暴露。然而，配置错误可能导致漏洞，削弱预期的安全保护或使其无效。本节概述了常见的安全头配置错误、其风险以及如何正确测试它们。

## 测试目标

- 识别配置不当的安全头。
- 评估配置错误的安全头的影响。
- 验证所需安全头的正确实现。

## 常见安全头配置错误

- **值为空的安全头：** 存在但缺少值的安全头可能被浏览器忽略，使其无效。
- **值或名称无效的安全头（拼写错误）：** 错误的头名称或拼写导致头无法被识别或执行。
- **过于宽松的安全头：** 配置过于广泛的安全头（如使用通配符 `*` 或过于宽松的指令）可能泄露信息或允许访问超出预期范围的资源。
- **重复的安全头：** 同一头的多次出现与冲突值可能导致不可预测的浏览器行为，可能完全禁用安全措施。
- **遗留或已弃用的头：** 包含现代浏览器不再支持的过时头（如 HPKP）或指令（如 `ALLOW-FROM` in X-Frame-Options）可能造成不必要的风险。
- **安全头放置无效：** 某些头仅在特定条件下有效。例如，HSTS 等头必须通过 HTTPS 传递；如果通过 HTTP 发送，它们将无效。
- **META 标签处理错误：** 在安全策略（如内容安全策略（CSP））通过 HTTP 头和 META 标签（使用 `http-equiv`）两者都强制执行的情况下，META 标签值可能覆盖或与 HTTP 头中定义的安全逻辑冲突。这可能导致不安全的策略无意中占上风，削弱整体安全态势。
- **逐跳头注入：** 当中间件错误地处理 `Connection` 头时发生，允许攻击者列出并"剥离"敏感的内部安全头（如 `X-Forwarded-For`），然后再将请求到达后端。

## 配置错误的安全头的风险

- **有效性降低：** 配置错误的头可能无法提供预期保护，使应用程序容易受到 XSS、点击劫持或 CORS 相关利用等攻击。
- **安全措施破坏：** 重复的头或冲突的指令可能导致浏览器完全忽略 HTTP 安全头，从而禁用预期的保护。
- **引入新的攻击向量：** 使用遗留或已弃用的头可能带来风险而不是缓解，如果现代浏览器不再支持预期的安全措施。

## 如何测试

### 获取并审查 HTTP 安全头

要检查应用程序使用的安全头，请采用以下方法：

- **拦截代理：** 使用 **Burp Suite** 等工具分析服务器响应。
- **命令行工具：** 执行 cURL 命令以检索 HTTP 响应头：`curl -I https://example.com`
    - 有时 Web 应用程序会重定向到新页面，要跟随重定向，请使用以下命令：`curl -L -I https://example.com`
    - 某些防火墙可能阻止 cURL 的默认 User-Agent，某些 TLS/SSL 错误也会阻止其返回正确信息，在这种情况下，您可以尝试使用以下命令：
    `curl -I -L -k --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.81 Safari/537.36" https://example.com`
- **浏览器开发人员工具：** 打开开发人员工具（F12），导航到 **Network** 选项卡，选择一个请求，并查看 **Headers** 部分。

### 检查过于宽松的安全头

- **识别有风险的头：** 查看可能允许过度访问的头，例如：
- **评估指令：** 验证是否执行了严格指令。例如，过于宽松的设置可能表现为：

    ```http
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Credentials: true
    X-Permitted-Cross-Domain-Policies: all
    Referrer-Policy: unsafe-url
    ```

    安全配置应如下：

    ```http
    Access-Control-Allow-Origin: {theallowedoriginurl}
    X-Permitted-Cross-Domain-Policies: none
    Referrer-Policy: no-referrer
    ```

- **交叉引用文档：** 使用 [Mozilla Developer Network: Security Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) 等资源审查安全和不安全指令。

### 检查重复、已弃用/过时的头

- **重复头：** 确保同一头没有用冲突值定义多次。
- **过时的头：** 识别并删除已弃用的头（如 HPKP）和过时指令（如 `ALLOW-FROM` in X-Frame-Options）。请参阅 [Mozilla Developer Network: X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) 等来源以获取当前标准。

### 确认安全头的正确放置

- **协议特定要求：** 验证旨在用于安全上下文的头（如 HSTS）仅在适当条件下（即通过 HTTPS）传递。
- **条件传递：** 某些头可能仅在特定情况下有效。验证这些条件是否满足，以便头按预期工作。

### 评估 META 标签处理

- **双重执行检查：** 当安全策略（如 CSP）通过 HTTP 头和使用 `http-equiv` 的 META 标签两者应用时，确认 HTTP 头（通常被视为更权威）不会被 META 标签无意中覆盖。
- **审查浏览器行为：** 在各种浏览器中测试应用程序，看看是否存在由于冲突指令而导致的差异。在可能的情况下，避免使用双重定义以防止意外的安全漏洞。

### 测试头剥离（逐跳注入）

攻击者可以通过在 `Connection` 头中列出敏感的内部安全头来进行利用。按照标准，代理在将请求转发到后端之前会剥离这些头。这可能导致：

- 绕过基于 IP 的访问控制列表（ACL）。
- 绕过在边缘执行的身份验证/身份检查。
- 禁用由中间头强制执行的安全功能。

#### 识别内部/敏感头（侦察）

要执行此测试，首先需要识别内部基础设施使用了哪些头。您可以通过以下方式识别它们：

- **触发错误页面：** 发送格式错误的请求以触发错误页面（404、500），这些页面可能会在响应中泄露内部头。
- **反射端点：** 搜索显示后端收到的所有头的调试或"Echo"页面（如 `/phpinfo`、`/debug`、`/env`）。
- **头猜测：** 常见目标包括 `X-Forwarded-For`、`X-Real-IP`、`X-Forwarded-Proto` 和 `X-Authenticated-User`。

#### 执行注入

尝试通过将目标头添加为 `Connection` 头的值来"剥离"它。

##### 场景 A：绕过基于 IP 的限制

```http
GET /admin HTTP/1.1
Host: example.com
X-Forwarded-For: 203.0.113.10
Connection: close, X-Forwarded-For
```

##### 场景 B：剥离身份验证上下文

```http
GET /api/user/profile HTTP/1.1
Host: example.com
X-Authenticated-User: victim_user
Connection: close, X-Authenticated-User
```

#### 分析响应

- **有漏洞：** 应用程序行为改变（例如，授予访问权限，或反射的 IP 消失）。
- **安全：** 应用程序行为保持不变，或者代理返回 `400 Bad Request`。

## 修复

- **正确的头配置：** 确保头被正确实现，具有正确的值且没有拼写错误。
- **执行严格指令：** 配置具有最安全设置的头，同时仍允许所需功能。例如，除非绝对必要，否则避免在 CORS 策略中使用 `*`。
- **删除已弃用的头：** 用现代等价物替换遗留安全头，并删除不再支持的任何头。
- **避免冲突定义：** 防止重复头定义，并确保 META 标签不与安全策略的 HTTP 头冲突。
- **限制 Connection 头：** 配置代理忽略 `Connection` 头中与敏感内部头匹配的客户提供的值。
- **零信任：** 避免完全依赖逐跳头进行关键安全决策。

## 工具

- [Mozilla Observatory](https://observatory.mozilla.org/)
- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net/burp)
- 浏览器开发人员工具（Chrome、Firefox、Edge）

## 参考资料

- [OWASP 安全头项目](https://owasp.org/www-project-secure-headers/)
- [Mozilla Developer Network：安全头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [RFC 6797 - HTTP 严格传输安全（HSTS）](https://datatracker.ietf.org/doc/html/rfc6797)
- [Google Web 安全指南](https://web.dev/security-headers/)
- [HPKP 不再存在](https://scotthelme.co.uk/hpkp-is-no-more/)
- [RFC 9110 - HTTP 语义学：Connection 头](https://datatracker.ietf.org/doc/html/rfc9110#section-7.6.1)
- [滥用 HTTP 逐跳请求头](https://nathandavison.com/blog/abusing-http-hop-by-hop-request-headers)
