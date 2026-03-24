# 测试弱密码学原语

|ID          |
|------------|
|WSTG-CRYP-04|

## 概述

加密算法的错误使用可能导致敏感数据暴露、密钥泄露、认证破坏、不安全会话和欺骗攻击。有一些已知较弱的加密算法或哈希算法，不建议使用，如MD5和RC4。

除了选择正确的安全加密或哈希算法外，参数的正确使用也对安全级别产生影响。例如，ECB（电子密码本）模式通常不应使用。

## 测试目标

- 为识别弱加密或哈希使用和实现提供指导。

## 如何测试

### 基本安全检查清单

- 当使用AES128或AES256时，IV（初始化向量）必须是随机且不可预测的。参阅[FIPS 140-2，密码模块的安全要求](https://csrc.nist.gov/publications/detail/fips/140/2/final)，第4.9.1节。随机数生成器测试。例如，在Java中，`java.util.Random`被认为是弱的随机数生成器。应该使用`java.security.SecureRandom`而不是`java.util.Random`。
- 对于非对称加密，使用椭圆曲线密码学（ECC），首选`Curve25519`等安全曲线。
    - 如果不能使用ECC，则使用至少2048位密钥的RSA加密。
- 当在签名中使用RSA时，推荐使用PSS填充。
- 不应使用弱哈希/加密算法，如MD5、RC4、DES、Blowfish、SHA1。1024位RSA或DSA、160位ECDSA（椭圆曲线）、80/112位2TDEA（双密钥三重DES）
- 最小密钥长度要求：

```text
密钥交换：Diffie-Hellman密钥交换，最小2048位
消息完整性：HMAC-SHA2
消息哈希：SHA2 256位
非对称加密：RSA 2048位
对称密钥算法：AES 128位
密码哈希：PBKDF2、Scrypt、Bcrypt
ECDH、ECDSA：256位
```

- 不应使用SSH、CBC模式。
- 当使用对称加密算法时，不应使用ECB（电子密码本）模式。
- 当使用PBKDF2哈希密码时，建议迭代参数超过10000。[NIST](https://pages.nist.gov/800-63-3/sp800-63b.html#sec5)也建议至少10000次哈希函数迭代。此外，禁止将MD5哈希函数与PBKDF2一起使用，如PBKDF2WithHmacMD5。

### 源代码审查

- 搜索以下关键字以识别弱算法的使用：`MD4, MD5, RC4, RC2, DES, Blowfish, SHA-1, ECB`

- 对于Java实现，以下API与加密相关。检查加密实现的参数。例如，

```java
SecretKeyFactory(SecretKeyFactorySpi keyFacSpi, Provider provider, String algorithm)
SecretKeySpec(byte[] key, int offset, int len, String algorithm)
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
```

- 对于RSA加密，建议使用以下填充模式。

```text
RSA/ECB/OAEPWithSHA-1AndMGF1Padding (2048)
RSA/ECB/OAEPWithSHA-256AndMGF1Padding (2048)
```

- 搜索`ECB`，不允许在填充中使用。
- 检查是否使用了不同的IV（初始向量）。

```java
// 为每次加密使用不同的IV值
byte[] newIv = ...;
s = new GCMParameterSpec(s.getTLen(), newIv);
cipher.init(..., s);
...
```

- 搜索`IvParameterSpec`，检查IV值是否以不同方式生成和随机化。

```java
 IvParameterSpec iv = new IvParameterSpec(randBytes);
 SecretKeySpec skey = new SecretKeySpec(key.getBytes(), "AES");
 Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
 cipher.init(Cipher.ENCRYPT_MODE, skey, iv);
```

- 在Java中，搜索MessageDigest以检查是否使用了弱哈希算法（MD5或CRC）。例如：

`MessageDigest md5 = MessageDigest.getInstance("MD5");`

- 对于签名，不应使用SHA1和MD5。例如：

`Signature sig = Signature.getInstance("SHA1withRSA");`

- 搜索`PBKDF2`。要生成密码的哈希值，建议使用`PBKDF2`。检查生成`PBKDF2`哈希值的参数。

迭代次数应超过**10000**，**salt**值应生成为**随机值**。

```java
private static byte[] pbkdf2(char[] password, byte[] salt, int iterations, int bytes)
    throws NoSuchAlgorithmException, InvalidKeySpecException
  {
     PBEKeySpec spec = new PBEKeySpec(password, salt, iterations, bytes * 8);
     SecretKeyFactory skf = SecretKeyFactory.getInstance(PBKDF2_ALGORITHM);
     return skf.generateSecret(spec).getEncoded();
  }
```

- 硬编码的敏感信息：

```text
用户相关关键字：name, root, su, sudo, admin, superuser, login, username, uid
密钥相关关键字：public key, AK, SK, secret key, private key, passwd, password, pwd, share key, shared key, cryto, base64
其他常见敏感关键字：sysadmin, root, privilege, pass, key, code, master, admin, uname, session, token, Oauth, privatekey, shared secret
```

## 工具

- Nessus、Nmap（脚本）或OpenVAS等漏洞扫描器可以扫描SNMP、TLS、SSH、SMTP等协议是否使用或接受弱加密。
- 使用静态代码分析工具进行源代码审查，如klocwork、Fortify、Coverity、CheckMark，用于以下情况。

```text
CWE-261：密码的弱密码学
CWE-323：加密中重用Nonce、密钥对
CWE-326：加密强度不足
CWE-327：使用破碎或风险密码算法
CWE-328：可逆单向哈希
CWE-329： CBC模式不使用随机IV
CWE-330：使用不充分的随机值
CWE-347：密码签名验证不当
CWE-354：完整性检查值验证不当
CWE-547：使用硬编码的安全相关常量
CWE-780：使用不带OAEP的RSA算法
```

## 参考资料

- [NIST FIPS标准](https://csrc.nist.gov/publications/fips)
- [维基百科：初始化向量](https://en.wikipedia.org/wiki/Initialization_vector)
- [安全编码 - 生成强随机数](https://www.securecoding.cert.org/confluence/display/java/MSC02-J.+Generate+strong+random+numbers)
- [最优非对称加密填充](https://en.wikipedia.org/wiki/Optimal_asymmetric_encryption_padding)
- [密码存储速查表](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [密码存储速查表](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [安全编码 - 不使用不安全或弱密码算法](https://www.securecoding.cert.org/confluence/display/java/MSC61-J.+Do+not+use+insecure+or+weak+cryptographic+algorithms)
- [不安全随机性](https://owasp.org/www-community/vulnerabilities/Insecure_Randomness)
- [熵不足](https://owasp.org/www-community/vulnerabilities/Insufficient_Entropy)
- [会话ID长度不足](https://owasp.org/www-community/vulnerabilities/Insufficient_Session-ID_Length)
- [使用破碎或风险的密码算法](https://owasp.org/www-community/vulnerabilities/Using_a_broken_or_risky_cryptographic_algorithm)
- [Javax.crypto.cipher API](https://docs.oracle.com/javase/8/docs/api/javax/crypto/Cipher.html)
- ISO 18033-1:2015 – 加密算法
- ISO 18033-2:2015 – 非对称密码
- ISO 18033-3:2015 – 分组密码
