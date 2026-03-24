# API对象级别授权损坏

|ID          |
|------------|
|WSTG-APIT-02|

## 概述

当API不正确地对客户端访问的每个对象实施授权检查时，会发生对象级别授权损坏（BOLA）。攻击者可以操纵API请求中的对象标识符（如ID、GUID或令牌）来访问或修改他们未被授权访问的资源。由于API直接访问底层对象以及API在现代应用程序中的普遍性，此漏洞在API中很关键。

利用BOLA可能导致未授权访问敏感数据、用户模拟、水平权限提升（访问其他用户的资源）和垂直权限提升（获得未授权的管理员级访问）。

## 测试目标

- 此测试的目的是识别API是否实施正确的**对象级授权**检查，确保用户只能访问和操作他们被授权交互的对象。

## 如何测试

### 了解API端点和对象引用

审查API文档（如OpenAPI规范）、流量或使用拦截代理（如**Burp Suite**、**ZAP**）识别接受感兴趣的对象标识符的端点。这些可能是**ID**、**UUID**或其他引用的形式。

示例：

- `GET /api/users/{user_id}`
- `GET /api/orders/{order_id}`
- `POST /graphql`\
        `query: {user(id: "123") }`

通过上一步获得的知识，收集可用于后续对象标识符操作的第三方对象标识符（如用户ID、订单ID等）。

另外，生成潜在对象标识符列表以进行暴力破解。例如，如果API从认证用户检索采购订单，生成各种采购订单ID用于测试。

### 在API请求中操纵对象标识符

目标是确定用户是否可以通过更改URL或请求体中的对象标识符（如用户ID、订单ID）来访问或修改他们不拥有的对象。

示例：将`GET /api/users/123/profile`（其中123是当前用户ID）的请求修改为`GET /api/users/124/profile`（其中124是另一个用户的ID）。

根据应用程序上下文，使用两个不同账户执行测试。使用账户A创建仅属于该账户的资源（如采购订单），并使用账户B尝试访问账户A的资源（如采购订单）。

### 使用不同HTTP方法测试对象级访问

测试各种**HTTP方法**的BOLA漏洞：

- **GET**：尝试通过操纵请求中的对象ID访问未授权对象。
- **POST/PUT/PATCH**：尝试创建或修改属于其他用户的对象。
- **DELETE**：尝试删除属于另一个用户的对象。

### 在GraphQL API中测试BOLA

对于**GraphQL API**，在查询参数中发送带有修改对象ID的查询（请参阅[测试GraphQL](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/12-API_Testing/01-Testing_GraphQL)）：

示例：`query { user(id: "124") { name, email } }`.

### 测试批量对象访问

测试API是否允许未授权**批量访问**对象。这可能发生在返回对象列表的端点中。

示例：`GET /api/users`返回所有用户的数据，而不是仅返回认证用户的数据。

## BOLA指标

- **成功利用**：如果修改请求中的对象ID返回属于其他用户的数据或允许对他们不拥有的对象执行操作，则API容易受到BOLA影响。
- **错误响应**：通常，安全API对于未授权对象访问会返回`403 Forbidden`或`401 Unauthorized`。对另一个用户的对象返回`200 OK`表示BOLA。
- **不一致响应**：如果某些端点实施授权而其他端点不实施，则表示安全控制不完整或不一致。

## 修复

- **对象所有权检查**：确保对每个API请求执行对象级授权检查。始终验证发出请求的用户有权访问所请求的对象。
- **基于角色的访问控制（RBAC）：** 实施定义哪些角色可以访问或修改特定对象的RBAC策略。
- **最小权限原则：** 应用最小权限原则以确保用户只能访问其角色所需的最小对象集。
- **使用UUID或非顺序ID：** 优先使用不可预测、非顺序的对象标识符（如**UUID**而非简单整数）使枚举和暴力攻击更加困难。

## 工具

- **ZAP**：自动扫描器或手动代理工具可帮助测试API请求中的对象引用。
- **Burp Suite**：使用**Repeater**或**Intruder**工具操纵对象ID并发送多个请求以测试访问控制。
- **Postman**：发送带有更改对象ID的请求并观察响应。
- **模糊测试工具：** 使用模糊器暴力破解对象ID并检查未授权访问。

## 参考资料

- [OWASP API安全Top 10：BOLA](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP测试指南：测试不安全直接对象引用（IDOR）](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [OWASP测试指南：测试GraphQL](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/12-API_Testing/01-Testing_GraphQL)
