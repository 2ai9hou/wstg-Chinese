# 测试场景模板

本区域提供了一个[示例模板](999-Foo_Testing/1-Testing_for_a_Cat_in_a_Box.md)及其[模板说明](999-Foo_Testing/2-Template_Explanation.md)，用于开发指南内容。（它们基于虚假的章节 `999`。）

## 格式示例

本文件夹还展示了如何格式化特定类型的内容：

- [HTTP 请求和响应](999-Foo_Testing/3-Format_for_HTTP_Request_Response.md)

## 合并或退役的测试

当测试被合并到另一个测试中时，保留原始文件以保持 ID 连续性。

在文件末尾，包含以下参考式元数据标记：

`[merged]: # (WSTG-XXXX-XX)`

将 `WSTG-XXXX-XX` 替换为目标测试 ID。

这确保合并的测试可以被明确识别，同时保持 Markdown 有效且渲染安全。
