# 测试过度数据暴露

|ID          |
|------------|
|WSTG-APIT-03|

## 概述

过度数据暴露发生在API在响应中返回的信息超过客户端需要显示或功能所需的信息时。这通常发生在开发人员通过将整个后端对象（如数据库模型或内部数据结构）直接序列化到API响应中来实现API端点，依赖客户端代码在向用户呈现之前过滤敏感字段。

问题在于API响应可直接被任何可以发出请求的人访问，无论用户界面选择显示什么。检查原始API流量的攻击者可以观察所有返回字段，包括UI故意隐藏的字段。这可能暴露密码、认证令牌、内部标识符、个人身份信息（PII）、财务数据、基础设施细节或内部业务逻辑。

此漏洞映射到[OWASP API安全Top 10 API3:2023损坏的对象属性级别授权](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)。

## 测试目标

- 识别API响应中包含的数据超过客户端应用程序显示或需要的数据的情况。
- 检测敏感字段在API响应中的返回，如密码哈希、认证令牌、内部对象ID、PII或基础设施细节。
- 确定API是否依赖客户端过滤而不是服务器端字段选择来控制数据暴露。

## 如何测试

### 将API响应与显示的数据进行比较

最直接的测试是将用户界面显示的内容与API实际返回的内容进行比较。

1. 在浏览器中打开应用程序，导航到显示用户或对象数据的页面（如个人资料页面、订单摘要或仪表板）。
2. 打开浏览器的开发者工具（Network选项卡）或配置拦截代理如Burp Suite或ZAP。
3. 识别填充页面内容的API请求。
4. 将UI中显示的字段与API响应中的完整字段集进行比较。

例如，用户资料页面可能只显示姓名和电子邮件，但底层API响应可能返回：

```json
{
  "id": 12045,
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "password_hash": "$2b$12$LJ3m4ys5qGn...",
  "role": "admin",
  "internal_account_id": "ACC-2024-88421",
  "ssn": "123-45-6789",
  "api_key": "REDACTED_EXAMPLE_KEY_12345",
  "created_by": "system_migration_v2",
  "last_login_ip": "10.0.3.47"
}
```

在这种情况下，API返回UI从不显示的密码哈希、社会安全号码、API密钥、内部标识符和内部IP地址。

### 检查不同API端点的响应

不要将测试限制在单个端点。检查多种端点类型的响应：

- **用户和账户端点：** `GET /api/users/{id}`、`GET /api/me`、`GET /api/account`
- **列表和搜索端点：** `GET /api/users`、`GET /api/orders?status=completed`（列表端点经常为列表中的每个项目返回相同的完整对象，放大暴露）
- **相关对象端点：** `GET /api/orders/{id}`可能嵌入完整用户对象、支付详情或订单响应中的收货地址
- **错误响应：** 失败的请求可能返回包含堆栈跟踪、内部文件路径、数据库查询详情或配置值的详细错误消息
- **管理或内部端点：** 检查旨在内部使用的端点是否可从外部访问并返回提升的数据

### 使用不同用户权限级别测试

同一端点可能根据请求用户的角色返回不同数量的数据。测试是否：

- 低权限用户请求相同资源时收到与管理员相同的字段。
- 未认证请求返回应需要认证的数据。
- 请求其他用户的数据（IDOR）返回与请求自己数据相同的丰富对象。

例如，`GET /api/users/124`可能无论请求者是用户124、另一用户还是未认证，都返回包含敏感字段的完整对象。

### 分析嵌套和嵌入对象

现代API经常在响应中嵌入相关对象。单个API调用可能返回深入嵌套的结构，暴露多个后端系统的数据。

示例：订单端点返回嵌入的客户对象、支付对象和发货对象：

```json
{
  "order_id": "ORD-9923",
  "items": [...],
  "customer": {
    "id": 5001,
    "name": "Bob Smith",
    "email": "bob@example.com",
    "phone": "+1-555-0199",
    "date_of_birth": "1985-03-14",
    "loyalty_tier": "gold"
  },
  "payment": {
    "method": "card",
    "card_last_four": "4242",
    "card_brand": "Visa",
    "card_fingerprint": "fp_abc123xyz",
    "billing_address": {...}
  },
  "internal_notes": "Customer flagged for manual review - suspected fraud"
}
```

应审查每个嵌套对象中超出消费客户端需要的字段。

### 检查GraphQL响应中的敏感数据

GraphQL API呈现这种问题的特定形式。虽然GraphQL允许客户端选择他们想要的字段，但架构可能暴露前端默认查询不请求但攻击者可以添加到自定义查询的敏感字段。

1. 如果启用自省，检索架构并审查每种类型上的可用字段（请参阅[测试GraphQL](99-Testing_GraphQL.md)）：

    ```graphql
    {
      __type(name: "User") {
        fields {
          name
          type { name }
        }
      }
    }
    ```

2. 查找如`passwordHash`、`apiKey`、`ssn`、`internalId`、`role`或`isAdmin`等字段，这些是可查询但前端未使用的。
3. 尝试直接查询这些字段：

    ```graphql
    {
      user(id: "123") {
        name
        email
        passwordHash
        apiKey
        role
      }
    }
    ```

### 检查JavaScript源映射和客户端捆绑包

前端JavaScript捆绑包和源映射可能无意中透露预期的API响应结构，包括UI故意隐藏的字段。如果源映射可公开访问（如`main.js.map`），它们可能暴露：

- 前端期望从API响应获取的完整字段列表
- 内部API端点路径
- 认证和授权流程逻辑
- 硬编码API密钥、令牌或配置值

测试源映射可用性：

```http
GET /static/js/main.js.map
GET /assets/app.js.map
```

如果源映射可访问并包含`sourcesContent`，可以重构和分析原始源代码以获取预期的API响应结构和隐藏字段。

### 检查错误响应和调试输出

API可能通过错误响应泄露敏感信息，特别是在开发或调试模式运行时：

- **堆栈跟踪**揭示内部文件路径、框架版本和代码结构
- **数据库错误消息**暴露表名、列名或查询语法
- **详细验证错误**列出所有预期字段，包括公共API文档中未记录的
- **调试头**如`X-Debug-Token`、`X-Powered-By`或包含内部信息的自定义头

通过发送格式错误的输入、无效认证或请求不存在的资源来触发错误响应，并检查包括头在内的完整响应。

## 修复

- **实施服务器端响应过滤。** 切勿在API响应中返回原始数据库对象或完整模型序列化。明确为每个端点和上下文定义仅包含客户端需要的字段的响应模式。
- **使用数据传输对象（DTO）或序列化允许列表。** 将内部对象映射到仅包含允许字段的专用响应对象。大多数Web框架都提供此机制（如Django REST Framework中的序列化器、Spring中的JsonView或Rails中的ActiveModel::Serializers）。
- **应用字段级访问控制。** 某些字段（如电子邮件或电话号码）可能适合一个消费者但不适合另一个。实施逻辑以根据请求用户的角色及其与请求资源的关系包含或排除字段。
- **避免暴露内部标识符。** 使用外部面向的标识符（如UUID或不透明令牌）而不是顺序内部数据库ID，这些会透露记录计数并启用枚举。
- **限制GraphQL架构。** 从GraphQL架构中完全删除敏感字段，而不是依赖解析器级检查。如果架构中必须存在内部使用的字段，应用授权指令以防止未授权访问。
- **在生产中禁用源映射。** 不要将JavaScript源映射部署到生产环境。配置构建管道以从生产工件中排除`.map`文件。
- **清理错误响应。** 在生产中返回通用错误消息。在服务器端记录详细错误信息，但不要在面向客户端的响应中包含堆栈跟踪、内部路径或查询详情。

## 工具

- **Burp Suite：** 使用Comparer工具对比显示的UI内容与原始API响应。Repeater和Intruder工具可用于系统地测试具有不同参数和认证上下文的端点。
- **ZAP（Zed Attack Proxy）：** 拦截和检查API响应，使用活动扫描规则识别信息披露，并使用模糊功能触发详细错误响应。
- **Postman：** 使用不同认证令牌发送API请求并比较响应体以识别角色相关数据暴露差异。
- **浏览器开发者工具：** Network选项卡提供对API响应的即时可见性，无需代理设置。用于快速比较UI呈现的数据与原始响应内容。
- **jq：** 命令行JSON处理器，用于提取和比较API响应之间的字段：`curl -s https://api.example.com/users/me | jq 'keys'`列出所有返回字段。

## 参考资料

- [OWASP API安全Top 10：API3:2023损坏的对象属性级别授权](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- [OWASP ASVS V5：API和Web服务](https://github.com/OWASP/ASVS/blob/v5.0.0/5.0/en/0x13-V4-API-and-Web-Service.md)
- [测试不安全直接对象引用（IDOR）](../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)
