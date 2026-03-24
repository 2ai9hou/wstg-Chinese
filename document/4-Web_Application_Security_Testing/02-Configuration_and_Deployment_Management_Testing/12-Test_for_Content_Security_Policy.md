# 测试内容安全策略

|ID          |
|------------|
|WSTG-CONF-12|

## 概述

内容安全策略（CSP）是一种通过 `Content-Security-Policy` 响应头或等效的 `<meta>` 元素强制执行的声明式白名单策略。它允许开发人员限制可从其中加载资源（如 JavaScript、CSS、图像、文件等）的来源。CSP 是一种有效的深度防御技术，可以降低跨站脚本（XSS）和点击劫持等漏洞的风险。

内容安全策略支持指令，允许对策略流程进行细粒度控制。（请参阅[参考资料](#references)以了解更多详情。）

## 测试目标

- 审查 Content-Security-Policy 头或 meta 元素以识别配置错误。

## 如何测试

测试内容安全策略（CSP）弱点需要的不只是验证头的存在。测试人员应评估策略是否有效地减少了攻击面并被正确执行。

### 识别并确认 CSP 执行

- 检查 HTTP 响应中是否存在 `Content-Security-Policy` 头。
- 检查是否存在 `Content-Security-Policy-Report-Only`。如果仅存在 Report-Only，则策略未被执行。
- 验证 CSP 是通过 HTTP 头还是 `<meta>` 标签传递的（HTTP 头是首选）。
- 注意，通过 `<meta>` 标签传递的 CSP 不支持某些指令，如 `frame-ancestors`、`report-uri`、`report-to` 或 `sandbox`。
- 确认策略在一致的敏感端点上应用。

### 审查高风险指令

检查策略中是否存在不安全或过于宽松的指令：

- `unsafe-inline` 允许内联脚本或样式，并显著削弱 XSS 保护。
- `unsafe-eval` 允许动态代码执行（`eval()`），增加绕过风险。
- `unsafe-hashes` 如果哈希是可预测的或作用域不当，可能允许内联执行。
- 通配符（`*`）源可能允许从任何来源加载资源。
    - 应仔细评估部分通配符，如 `https://*` 或 `*.cdn.com`。
    - 确定白名单域是否托管 JSONP 端点或用户控制的内容。
- 缺少或滥用 `frame-ancestors` 可能使应用程序面临点击劫持风险。
- 缺少 `object-src`、`base-uri` 或限制性不强的 `default-src` 指令可能削弱策略有效性。
- 审查 `require-trusted-types-for` 和 `trusted-types` 的使用情况。在高风险应用程序中，缺少可信类型可能使基于 DOM 的注入接收端暴露。如果定义了 `trusted-types` 策略，确保它们不是过于宽松。
- 检查是否存在重复指令或冲突的策略定义，这可能导致非预期的执行行为。

### 验证 Nonce 和 strict-dynamic 使用

如果策略使用 nonce：

- 确认 nonce 在密码学上是随机的。
- 验证 nonce 是为每个响应重新生成的，而不是被重用的。
- 确保传统内联脚本模式不会被无意信任。

如果使用 `strict-dynamic`：

- 理解信任从基于 nonce 或哈希的脚本传播。
- 确认没有不安全的信任链允许攻击者控制的脚本加载。

### 评估 CSP 报告机制

如果配置了 `report-uri` 或 `report-to`：

- 验证报告端点可达且功能正常。
- 确定报告中是否暴露了敏感信息。
- 确认报告不会创建新的注入或拒绝服务向量。

### 尝试受控绕过技术

在适当和授权的情况下，尝试通过测试受控 payload 来验证执行：

- 内联脚本注入尝试。
- 基于数据 URL 的 payload。
- 从白名单域进行的 JSONP 回调操作。
- 使用可信脚本源的基于 DOM 的小工具链。

注入的 JavaScript 成功执行表明 CSP 配置错误或执行无效。

### 评估策略强度

关键业务应用程序应旨在实施严格的策略。强大的 CSP 通常：

- 避免使用 `unsafe-inline` 和 `unsafe-eval`。
- 使用基于 nonce 或哈希的脚本控制。
- 限制对象嵌入（`object-src 'none'`）。
- 限制 base 标签操作（`base-uri 'none'`）。
- 使用 `frame-ancestors` 限制框架。

### 常见 CSP 绕过模式

即使存在 CSP，配置错误或设计弱点也可能允许绕过。测试人员应考虑以下常见模式：

#### JSONP 和可信第三方端点

如果 CSP 白名单了第三方域（如 CDN），确定这些域是否暴露了 JSONP 端点或用户控制的内容。攻击者可能利用回调注入来执行任意 JavaScript，同时仍遵守策略。

#### 通配符和广泛源策略

依赖通配符（`*`、`https://*`、`*.example.com`）的策略显著扩大了信任边界。子域名接管或受损的第三方服务可能允许在被允许的范围内进行绕过。

#### Nonce 重用或可预测的 Nonce

如果 nonce 在请求之间被重用或可预测地生成，攻击者可能重用或猜测它们来执行注入的脚本。每个响应应生成一个新的、密码学上强的 nonce。

#### 过度依赖 strict-dynamic

虽然 `strict-dynamic` 改进了基于 nonce 的策略，但信任从可信脚本传播。如果可信脚本加载了攻击者控制的资源，保护可能会被削弱。

#### 缺少深度防御指令

缺少以下指令可能使应用程序暴露于对象注入、base 标签操作或点击劫持，即使脚本执行受到部分限制：

- `object-src 'none'`
- `base-uri 'none'`
- `frame-ancestors`

#### CSP 仅在 Report-Only 模式下

应用程序有时会无限期地在 `Report-Only` 模式下部署 CSP。在这种情况下，违规被记录但不被阻止，提供不了真正的保护。

## 修复

组织应实施强大的、范围适当的内容安全策略，以有意义地减少攻击面，而不是简单地满足合规要求。

### 一般建议

- 避免使用 `unsafe-inline` 和 `unsafe-eval`。
- 优先使用基于 nonce 或哈希的脚本控制。
- 使用限制性的 `default-src` 指令。
- 明确界定：
    - `object-src 'none'`
    - `base-uri 'none'`
    - `frame-ancestors`
- 最小化白名单第三方域的数量。
- 在引入新的前端依赖项时定期审查 CSP 策略。

### 部署最佳实践

- 暂时部署 `Content-Security-Policy-Report-Only` 进行测试，然后在验证后执行 `Content-Security-Policy`。
- 监控违规报告以发现合法破坏和潜在攻击尝试。
- 优先使用 `report-to`（CSP Level 3），同时在需要时维护 `report-uri` 以实现向后兼容。
- 确保 nonce 在密码学上是随机的，并为每个 HTTP 响应重新生成。
- 在重大前端重构或框架更改期间审查策略。

### 严格策略指导

严格的策略提供针对存储型、反射型和某些基于 DOM 的 XSS 攻击的保护，应该是高风险或关键业务应用程序的目标。

基于 nonce 的方法，示例策略包括：

中等严格策略：

```HTTP
script-src 'nonce-r4nd0m' 'strict-dynamic';
object-src 'none';
base-uri 'none';
```

锁定严格策略：

```HTTP
script-src 'nonce-r4nd0m';
object-src 'none';
base-uri 'none';
```

注意：`'nonce-r4nd0m'` 显示为示例占位符。在实践中，nonce 必须是密码学上强的、base64 编码的值，为每个 HTTP 响应唯一生成。

团队应仔细调整严格策略，确保与应用程序架构的兼容性，同时保持安全目标。

## 工具

- [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [CSP Auditor - Burp Suite 扩展](https://portswigger.net/bappstore/35237408a06043e9945a11016fcbac18)
- [CSP Generator Chrome](https://chrome.google.com/webstore/detail/content-security-policy-c/ahlnecfloencbkpfnpljbojmjkfgnmdc) / [Firefox](https://addons.mozilla.org/en-US/firefox/addon/csp-generator/)
- [CSP Validator](https://cspvalidator.netlify.app/)
- [OWASP ZAP](https://www.zaproxy.org/) – 包括 CSP 配置错误的自动化和被动分析。
- [CSPBypass](https://cspbypass.com/) – 旨在帮助安全测试人员分析和尝试针对限制性 CSP 实现的绕过技术的工具。

## 参考资料

- [OWASP 内容安全策略速查表](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Mozilla Developer Network：内容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [CSP Level 3 W3C](https://www.w3.org/TR/CSP3/)
- [使用 Google 的 CSP](https://csp.withgoogle.com/docs/index.html)
- [Content-Security-Policy](https://content-security-policy.com/)
- [CSP 在强化和缓解之间成功的混乱](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation)
- [unsafe-hashes 源列表关键字](https://content-security-policy.com/unsafe-hashes/)
