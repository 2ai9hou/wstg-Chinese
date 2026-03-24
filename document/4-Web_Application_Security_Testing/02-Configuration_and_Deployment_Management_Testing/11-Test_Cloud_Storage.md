# 测试云存储

|ID          |
|------------|
|WSTG-CONF-11|

## 概述

云存储服务允许 Web 应用程序和服务在存储服务中存储和访问对象。然而，访问控制配置不当可能导致敏感信息泄露、数据篡改或未经授权的访问。

一个众所周知的例子是 Amazon S3 存储桶配置错误，尽管其他云存储服务也可能面临类似风险。默认情况下，所有 S3 存储桶都是私有的，只能由被明确授予访问权限的用户访问。用户可以授予公众访问权限，不仅可以访问存储桶本身，还可以访问存储在该存储桶中的单个对象。这可能导致未经授权的用户能够上传新文件、修改或读取存储的文件。

## 测试目标

- 评估存储服务的访问控制配置是否正确到位。

## 如何测试

首先，识别访问存储服务中数据的 URL，然后考虑以下测试：

- 读取未授权数据
- 上传新的任意文件

您可以使用 curl 进行测试，使用以下命令并查看未经授权的操作是否能成功执行。

要测试读取对象的能力：

```bash
curl -X GET https://<cloud-storage-service>/<object>
```

要测试上传文件的能力：

```bash
curl -X PUT -d 'test' 'https://<cloud-storage-service>/test.txt'
```

在上述命令中，在 Windows 机器上运行命令时，建议将单引号（'）替换为双引号（"）。

### 测试 Amazon S3 存储桶配置错误

Amazon S3 存储桶 URL 遵循两种格式之一：虚拟主机样式或路径样式。

- 虚拟主机样式访问

```text
https://bucket-name.s3.Region.amazonaws.com/key-name
```

在以下示例中，`my-bucket` 是存储桶名称，`us-west-2` 是区域，`puppy.png` 是 key-name：

```text
https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png
```

- 路径样式访问

```text
https://s3.Region.amazonaws.com/bucket-name/key-name
```

与上面一样，在以下示例中，`my-bucket` 是存储桶名称，`us-west-2` 是区域，`puppy.png` 是 key-name：

```text
https://s3.us-west-2.amazonaws.com/my-bucket/puppy.png
```

对于某些区域，可以使用不指定区域特定端点的旧版全局端点。其格式也是虚拟主机样式或路径样式。

- 虚拟主机样式访问

```text
https://bucket-name.s3.amazonaws.com
```

- 路径样式访问

```text
https://s3.amazonaws.com/bucket-name
```

#### 识别存储桶 URL

对于黑盒测试，S3 URL 可以在 HTTP 消息中找到。以下示例显示存储桶 URL 在 HTTP 响应的 `img` 标签中发送。

```html
...
<img src="https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png">
...
```

对于灰盒测试，您可以从 Amazon 的 Web 界面、文档、源代码和任何其他可用来源获取存储桶 URL。

#### 使用 AWS-CLI 进行测试

除了使用 curl 进行测试外，您还可以使用 AWS 命令行工具进行测试。在这种情况下，使用 `s3://` URI 方案。

##### 列出

以下命令在存储桶配置为公共时列出存储桶的所有对象：

```bash
aws s3 ls s3://<bucket-name>
```

##### 上传

以下是上传文件的命令：

```bash
aws s3 cp arbitrary-file s3://bucket-name/path-to-save
```

以下示例显示上传成功时的结果：

```bash
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload: ./test.txt to s3://bucket-name/test.txt
```

以下示例显示上传失败时的结果：

```bash
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload failed: ./test2.txt to s3://bucket-name/test2.txt An error occurred (AccessDenied) when calling the PutObject operation: Access Denied
```

##### 删除

以下是删除对象的命令：

```bash
aws s3 rm s3://bucket-name/object-to-remove
```

## 工具

- [AWS CLI](https://aws.amazon.com/cli/)

## 参考资料

- [使用 Amazon S3 存储桶](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html)
- [flAWS 2 - 学习 AWS 安全](http://flaws2.cloud)
- [curl 教程](https://curl.se/docs/manual.html)
