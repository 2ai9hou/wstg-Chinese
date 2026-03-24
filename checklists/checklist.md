# 测试检查清单

以下是在评估期间需要测试的项目列表：

注意：`状态` 列可以设置为类似"通过"、"失败"、"不适用"的值。

| 测试 ID | 测试名称 | 状态 | 备注 |
|-------------------|----------------------------------------------------------------------------|--------|-------|
| **WSTG-INFO** | **信息收集** | | |
| WSTG-INFO-01 | 进行搜索引擎侦察以发现信息泄露 | | |
| WSTG-INFO-02 | 识别 Web 服务器指纹 | | |
| WSTG-INFO-03 | 审查 Web 服务器元文件以发现信息泄露 | | |
| WSTG-INFO-04 | 枚举 Web 服务器上的应用 | | |
| WSTG-INFO-05 | 审查网页内容以发现信息泄露 | | |
| WSTG-INFO-06 | 识别应用入口点 | | |
| WSTG-INFO-07 | 映射应用执行路径 | | |
| WSTG-INFO-08 | 识别 Web 应用框架指纹 | | |
| WSTG-INFO-09 | 识别 Web 应用指纹 | | |
| WSTG-INFO-10 | 映射应用架构 | | |
| **WSTG-CONF** | **配置与部署管理测试** | | |
| WSTG-CONF-01 | 测试网络基础设施配置 | | |
| WSTG-CONF-02 | 测试应用平台配置 | | |
| WSTG-CONF-03 | 测试敏感信息的文件扩展名处理 | | |
| WSTG-CONF-04 | 审查旧备份和未引用文件以发现敏感信息 | | |
| WSTG-CONF-05 | 枚举基础设施和应用管理接口 | | |
| WSTG-CONF-06 | 测试 HTTP 方法 | | |
| WSTG-CONF-07 | 测试 HTTP 严格传输安全 | | |
| WSTG-CONF-08 | 测试 RIA 跨域策略 | | |
| WSTG-CONF-09 | 测试文件权限 | | |
| WSTG-CONF-10 | 测试子域名接管 | | |
| WSTG-CONF-11 | 测试云存储 | | |
| WSTG-CONF-12 | 测试内容安全策略 | | |
| WSTG-CONF-13 | 测试路径混淆 | | |
| WSTG-CONF-14 | 测试其他 HTTP 安全头配置错误 | | |
| **WSTG-IDNT** | **身份管理测试** | | |
| WSTG-IDNT-01 | 测试角色定义 | | |
| WSTG-IDNT-02 | 测试用户注册流程 | | |
| WSTG-IDNT-03 | 测试账户配置流程 | | |
| WSTG-IDNT-04 | 测试账户枚举和可猜测用户账户 | | |
| WSTG-IDNT-05 | 测试弱或未强制的用户名策略 | | |
| **WSTG-ATHN** | **认证测试** | | |
| WSTG-ATHN-01 | 测试通过加密通道传输的凭据 | | |
| WSTG-ATHN-02 | 测试默认凭据 | | |
| WSTG-ATHN-03 | 测试弱锁定机制 | | |
| WSTG-ATHN-04 | 测试绕过认证架构 | | |
| WSTG-ATHN-05 | 测试易受的记住密码功能 | | |
| WSTG-ATHN-06 | 测试浏览器缓存弱点 | | |
| WSTG-ATHN-07 | 测试弱密码策略 | | |
| WSTG-ATHN-08 | 测试弱安全问题答案 | | |
| WSTG-ATHN-09 | 测试弱密码更改或重置功能 | | |
| WSTG-ATHN-10 | 测试替代通道中较弱的认证 | | |
| WSTG-ATHN-11 | 测试多因素认证（MFA） | | |
| **WSTG-ATHZ** | **授权测试** | | |
| WSTG-ATHZ-01 | 测试目录遍历文件包含 | | |
| WSTG-ATHZ-02 | 测试绕过授权架构 | | |
| WSTG-ATHZ-03 | 测试权限提升 | | |
| WSTG-ATHZ-04 | 测试不安全的直接对象引用 | | |
| WSTG-ATHZ-05 | 测试 OAuth 弱点 | | |
| **WSTG-SESS** | **会话管理测试** | | |
| WSTG-SESS-01 | 测试会话管理架构 | | |
| WSTG-SESS-02 | 测试 Cookie 属性 | | |
| WSTG-SESS-03 | 测试会话固定 | | |
| WSTG-SESS-04 | 测试暴露的会话变量 | | |
| WSTG-SESS-05 | 测试跨站请求伪造 | | |
| WSTG-SESS-06 | 测试注销功能 | | |
| WSTG-SESS-07 | 测试会话超时 | | |
| WSTG-SESS-08 | 测试会话拼图 | | |
| WSTG-SESS-09 | 测试会话劫持 | | |
| WSTG-SESS-10 | 测试 JSON Web 令牌 | | |
| WSTG-SESS-11 | 测试并发会话 | | |
| **WSTG-INPV** | **输入验证测试** | | |
| WSTG-INPV-01 | 测试反射型跨站脚本 | | |
| WSTG-INPV-02 | 测试存储型跨站脚本 | | |
| WSTG-INPV-03 | 测试 HTTP 动词篡改 | | |
| WSTG-INPV-04 | 测试 HTTP 参数污染 | | |
| WSTG-INPV-05 | 测试 SQL 注入 | | |
| WSTG-INPV-06 | 测试 LDAP 注入 | | |
| WSTG-INPV-07 | 测试 XML 注入 | | |
| WSTG-INPV-08 | 测试 SSI 注入 | | |
| WSTG-INPV-09 | 测试 XPath 注入 | | |
| WSTG-INPV-10 | 测试 IMAP SMTP 注入 | | |
| WSTG-INPV-11 | 测试代码注入 | | |
| WSTG-INPV-12 | 测试命令注入 | | |
| WSTG-INPV-13 | 测试格式化字符串注入 | | |
| WSTG-INPV-14 | 测试潜伏型漏洞 | | |
| WSTG-INPV-15 | 测试 HTTP 响应拆分 | | |
| WSTG-INPV-16 | 测试 HTTP 请求走私 | | |
| WSTG-INPV-17 | 测试 Host 头注入 | | |
| WSTG-INPV-18 | 测试服务端模板注入 | | |
| WSTG-INPV-19 | 测试服务端请求伪造 | | |
| WSTG-INPV-20 | 测试批量赋值 | | |
| **WSTG-ERRH** | **错误处理** | | |
| WSTG-ERRH-01 | 测试不正确的错误处理 | | |
| WSTG-ERRH-02 | 测试堆栈跟踪 | | |
| **WSTG-CRYP** | **密码学** | | |
| WSTG-CRYP-01 | 测试弱传输层安全 | | |
| WSTG-CRYP-02 | 测试填充预言机 | | |
| WSTG-CRYP-03 | 测试通过未加密通道发送的敏感信息 | | |
| WSTG-CRYP-04 | 测试弱密码原语 | | |
| **WSTG-BUSLOGIC** | **业务逻辑测试** | | |
| WSTG-BUSL-01 | 测试业务逻辑数据验证 | | |
| WSTG-BUSL-02 | 测试伪造请求的能力 | | |
| WSTG-BUSL-03 | 测试完整性检查 | | |
| WSTG-BUSL-04 | 测试流程时序 | | |
| WSTG-BUSL-05 | 测试函数使用次数限制 | | |
| WSTG-BUSL-06 | 测试工作流绕过 | | |
| WSTG-BUSL-07 | 测试应用滥用防御 | | |
| WSTG-BUSL-08 | 测试意外文件类型上传 | | |
| WSTG-BUSL-09 | 测试恶意文件上传 | | |
| WSTG-BUSL-10 | 测试支付功能 | | |
| **WSTG-CLIENT** | **客户端测试** | | |
| WSTG-CLNT-01 | 测试基于 DOM 的跨站脚本 | | |
| WSTG-CLNT-02 | 测试 JavaScript 执行 | | |
| WSTG-CLNT-03 | 测试 HTML 注入 | | |
| WSTG-CLNT-04 | 测试客户端 URL 重定向 | | |
| WSTG-CLNT-05 | 测试 CSS 注入 | | |
| WSTG-CLNT-06 | 测试客户端资源操作 | | |
| WSTG-CLNT-07 | 测试跨域资源共享 | | |
| WSTG-CLNT-08 | 测试跨站 Flash 攻击 | | |
| WSTG-CLNT-09 | 测试点击劫持 | | |
| WSTG-CLNT-10 | 测试 WebSockets | | |
| WSTG-CLNT-11 | 测试 Web 消息传递 | | |
| WSTG-CLNT-12 | 测试浏览器存储 | | |
| WSTG-CLNT-13 | 测试跨站脚本包含 | | |
| WSTG-CLNT-14 | 测试反向标签劫持 | | |
| **WSTG-APIT** | **API 测试** | | |
| WSTG-APIT-01 | API 侦察 | | |
| WSTG-APIT-02 | API 对象级别授权损坏 | | |
| WSTG-APIT-99 | 测试 GraphQL | | |
