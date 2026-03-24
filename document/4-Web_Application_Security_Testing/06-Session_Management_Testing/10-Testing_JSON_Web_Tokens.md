# 测试JSON Web令牌

|ID          |
|------------|
|WSTG-SESS-10|

## 概述

JSON Web令牌（JWT）是加密签名的JSON令牌，旨在在系统之间共享声明。它们经常用作认证或会话令牌，特别是在REST API中。

JWT是漏洞的常见来源，无论是在应用中的实现方式上，还是在底层库中。由于它们用于认证，漏洞很容易导致应用的完全危害。

## 测试目标

- 确定JWT是否暴露敏感信息。
- 确定JWT是否可以被篡改或修改。

## 如何测试

### 概述

JWT由三个组件组成：

- 头部
- 负载（或正文）
- 签名

每个组件都是base64编码的，它们用句点（`.`）分隔。请注意，JWT中使用的base64编码会去掉等号（`=`），因此您可能需要添加回来以解码各个部分。

### 分析内容

#### 头部

头部定义令牌类型（通常为`JWT`）和用于签名的算法。以下是解码头部的示例：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

有三种主要类型的算法用于计算签名：

| 算法 | 描述 |
|-----------|-------------|
| HSxxx | 使用密钥和SHA-xxx的HMAC。 |
| RSxxx和PSxxx | 使用RSA的公钥签名。 |
| ESxxx | 使用ECDSA的公钥签名。 |

还有广泛的其他算法范围，可能用于加密令牌（JWE），尽管这些不太常见。

#### 负载

JWT的负载包含实际数据。以下是示例负载：

```json
{
  "username": "administrator",
  "is_admin": true,
  "iat": 1516239022,
  "exp": 1516242622
}
```

负载通常不加密，因此请查看它以确定是否包含任何敏感的或不适当的数据。

此JWT包含用户的用户名和管理状态，以及两个标准声明（`iat`和`exp`）。这些声明在[RFC 5719](https://tools.ietf.org/html/rfc7519#section-4.1)中定义，以下表格给出了简要总结：

| 声明 | 全名 | 描述 |
|-------|-----------|-------------|
| `iss` | Issuer | 颁发令牌的一方的身份。 |
| `iat` | Issued At | 令牌颁发时间的Unix时间戳。 |
| `nbf` | Not Before | 令牌可以使用的最早日期的Unix时间戳。 |
| `exp` | Expires | 令牌过期的时间的Unix时间戳。 |

#### 签名

签名是使用JWT头部中定义的算法计算的，然后进行base64编码并附加到令牌。修改JWT的任何部分都会导致签名无效，并导致令牌被服务器拒绝。

### 审查使用

除了密码学安全本身之外，JWT还需要以安全的方式存储和发送。这应包括检查：

- 它始终通过加密（HTTPS）连接[发送](../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md)。
- 如果存储在Cookie中，则应[使用适当的属性标记](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)。

还应基于`iat`、`nbf`和`exp`声明审查JWT的有效性，以确定：

- JWT在应用中有合理的生命周期。
- 应用拒绝过期的令牌。

### 签名验证

在使用JWT时遇到的最严重漏洞之一是应用未能验证签名是否正确。这通常发生在开发人员使用诸如Node.js的`jwt.decode()`函数之类的东西时，该函数只是解码JWT的主体，而不是`jwt.verify()`，后者在解码JWT之前验证签名。

通过在不改带头部或签名中的任何内容的情况下修改JWT的主体，提交请求以查看应用是否接受它，可以轻松测试这一点。

#### None算法

除了公钥和基于HMAC的算法外，JWT规范还定义了一个名为`none`的签名算法。顾名思义，这意味着JWT没有签名，允许修改。

可以通过将JWT头部中的签名算法（`alg`）修改为`none`来测试，如下例所示：

```json
{
        "alg": "none",
        "typ": "JWT"
}
```

然后，头部和负载使用base64重新编码，并删除签名（留下尾随句点）。使用上面的头部和[负载](#payload)部分中列出的负载，这将产生以下JWT：

```txt
eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0K.eyJ1c2VybmFtZSI6ImFkbWluaW5pc3RyYXRvciIsImlzX2FkbWluIjp0cnVlLCJpYXQiOjE1MTYyMzkwMjIsImV4cCI6MTUxNjI0MjYyMn0.
```

一些实现通过明确阻止使用`none`算法来尝试避免这种情况。如果以不区分大小写的方式完成，则可能通过指定诸如`NoNe`之类的算法来绕过。

#### ECDSA "心理签名"

在Java版本15到18中发现了一个漏洞，它们在某些情况下没有正确验证ECDSA签名（[CVE-2022-21449](https://neilmadden.blog/2022/04/19/psychic-signatures-in-java/)，称为"心理签名"）。如果使用这些易受攻击的版本之一来使用`ES256`算法解析JWT，则可以通过篡改主体然后用以下值替换签名来完全绕过签名验证：

```txt
MAYCAQACAQA
```

生成的JWT看起来像这样：

```txt
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6InRydWUifQ.MAYCAQACAQA
```

### 弱HMAC密钥

如果JWT使用基于HMAC的算法（如HS256）签名，则签名的安全性完全依赖于HMAC中使用的密钥的强度。

如果应用使用现成的或开源软件，第一步应该是调查代码，看看是否有使用的默认HMAC签名密钥。

如果没有默认的，则可能猜测或暴力破解密钥。最简单的方法是使用[crackjwt.py](https://github.com/Sjord/jwtcrack)脚本，它只需要JWT和字典文件。

一个更强大的选项是将JWT转换为可用于[John the Ripper](https://github.com/openwall/john)的格式，使用[jwt2john.py](https://github.com/Sjord/jwtcrack/blob/master/jwt2john.py)脚本。然后John可以用来对密钥进行更高级的攻击。

如果JWT很大，它可能超过John支持的最大大小。可以通过增加`/src/hmacSHA256_fmt_plug.c`中（或对于其他HMAC格式的等效文件）的`SALT_LIMBS`变量值并重新编译John来解决这个问题，如以下[GitHub问题](https://github.com/openwall/john/issues/1904)中所讨论的。

如果可以获得此密钥，则可以创建和签名任意JWT，这通常会导致应用的完全危害。

### HMAC与公钥混淆

如果应用使用带有公钥签名的JWT，但不检查算法是否正确，这可能会在签名类型混淆攻击中被利用。为了使其成功，需要满足以下条件：

1. 应用必须期望JWT使用基于公钥的算法（即`RSxxx`或`ESxxx`）签名。
2. 应用不得检查JWT实际使用哪种算法进行签名。
3. 用于验证JWT的公钥必须可供攻击者使用。

如果所有这些条件都成立，则攻击者可以使用公钥使用基于HMAC的算法（如`HS256`）对JWT进行签名。例如，[Node.js jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)库对公钥和基于HMAC的令牌使用相同的函数，如下所示：

```javascript
// Verify a JWT signed using RS256
jwt.verify(token, publicKey);

// Verify a JWT signed using HS256
jwt.verify(token, secretKey);
```

这意味着如果JWT使用`publicKey`作为`HS256`算法的密钥进行签名，则签名将被视为有效。

为了利用此问题，必须获取公钥。最常见的情况是应用将同一密钥用于签名JWT和作为TLS证书的一部分。在这种情况下，可以使用以下命令从服务器下载密钥：

```sh
openssl s_client -connect example.org:443 | openssl x509 -pubkey -noout
```

或者，密钥可能以公共文件的形式在站点的常见位置提供，例如`/.well-known/jwks.json`。

为了测试这个，修改JWT的内容，然后使用先前获取的公钥使用`HS256`算法对JWT进行签名。当在没有访问源代码或实现细节的情况下进行测试时，这通常很困难，因为密钥的格式必须与服务器使用的格式完全相同，因此诸如空格或CRLF编码等问题可能导致密钥不匹配。

### 攻击者提供的公钥

[JSON Web Signature (JWS)标准](https://tools.ietf.org/html/rfc7515)（定义JWT使用的头部和签名）允许将用于签名令牌的密钥嵌入头部。如果用于验证令牌的库支持此功能，并且不将密钥与已批准密钥列表进行检查，则允许攻击者使用他们提供的任意密钥对JWT进行签名。

有各种脚本可用于执行此操作，例如[jwk-node-jose.py](https://github.com/zi0Black/POC-CVE-2018-0114)或[jwt_tool](https://github.com/ticarpi/jwt_tool)。

### 密钥ID（kid）操作

`kid`头参数通常用于从文件系统或数据库检索验证签名所需的密钥。它可能容易受到多种注入攻击。

#### 目录遍历

如果应用使用`kid`参数从文件系统读取密钥文件，攻击者可能会指定一个已知的空文件路径，例如`../../../../dev/null`（在Linux上）或`nul`（在Windows上）。

例如，攻击者可以修改头部以指向一个空文件：

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "../../../../../dev/null"
}
```

由于`/dev/null`的内容为空，攻击者可以使用**空字符串**作为密钥对恶意令牌进行签名。如果服务器易受攻击，它将读取空文件，使用空字符串验证签名，并接受伪造的令牌。

#### 命令/SQL注入

如果`kid`未经过滤地传递到数据库查询或系统命令以检索密钥，则可能容易受到SQL注入或命令注入的影响。

例如，攻击者可以将SQL有效负载注入`kid`参数，以控制数据库返回的密钥：

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "invalid-key' UNION SELECT 'attacker-controlled-key'--"
}
```

这允许攻击者强制应用使用已知密钥（例如"attacker-controlled-key"）进行验证，使他们能够伪造有效令牌。

## 相关测试用例

- [测试通过未加密通道发送的敏感信息](../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md)。
- [测试Cookie属性](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)。
- [测试浏览器存储](../11-Client-side_Testing/12-Testing_Browser_Storage.md)。

## 修复方案

- 使用安全且最新的库来处理JWT。
- 确保签名有效，并且使用预期的算法。
- 使用强HMAC密钥或唯一私钥进行签名。
- 确保负载中没有敏感信息暴露。
- 确保JWT被安全存储和传输。
- 参阅[OWASP JSON Web令牌Java速查表](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)。

## 工具

- [John the Ripper](https://github.com/openwall/john)
- [jwt2john](https://github.com/Sjord/jwtcrack)
- [jwt-cracker](https://github.com/brendan-rius/c-jwt-cracker)
- [JSON Web Tokens Burp Extension](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6)
- [ZAP JWT Add-on](https://github.com/SasanLabs/owasp-zap-jwt-addon)

## 参考资料

- [RFC 7515 JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)
- [RFC 7519 JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
- [OWASP JSON Web Token Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
