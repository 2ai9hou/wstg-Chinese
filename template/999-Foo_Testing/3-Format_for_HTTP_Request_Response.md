# HTTP 请求和响应的格式

|ID          |
|------------|
|WSTG-FOO-003|

## 如何包含 HTTP 请求或响应块

要在文章中使用 HTTP 请求和响应的示例，请使用[原始 HTTP 消息](https://tools.ietf.org/html/rfc2616)和 `http` Markdown 代码块语言：

```markdown
```http
在此处放置请求或响应捕获
```
```

- 尽量保持块较小。
- 使用方括号和省略号 `[...]` 来表示消息被截断，如果它有助于文章的清晰度的话。

以下部分是一个带有 HTTP 消息和格式描述的示例文章片段。

## HTTP 请求和响应示例

如果测试人员发送以下主页的 HTTP 请求：

```http
GET / HTTP/1.1
Host: localhost:8080
```

检查响应是否显示有关服务器的信息：

```http
HTTP/1.1 200
[...]
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Apache Tomcat/10.0.4
[...]
```

在此结果中，响应将服务器标识为 Tomcat 10.0.4。

## 示例说明

- HTTP 请求和响应在请求和响应之前有描述性文本向读者说明。
- GET 请求具有最少的头部以从服务器获得期望的响应。
    - 例如，没有 `User-Agent:`，因为它对于"测试用例"不是必需的。
- 文章使用方括号和省略号 `[...]` 来切除响应的不必要部分。
    - 对于此示例不必要的响应内容包括 `Content-Type:` 头和正文中 HTML 的其余部分。
