# 测试盒中的猫

|ID          |
|------------|
|WSTG-FOO-001|

## 摘要

[盒子](https://en.wikipedia.org/wiki/Box)是一个有形物体，通常由六个矩形面组成。它通常具有打开或关闭以及容纳物品的能力。盒子通常用于运输其他物品，或临时或永久存放物品。盒子可以用各种材料制成，如纸板、木材或钢材。

盒子可能包含也可能不包含[猫](https://en.wikipedia.org/wiki/Cat)。

如果盒子里有一只猫，而盒子主人不知道，如果他们意外发现这只猫，盒子主人可能会受到惊吓。

## 测试目标

评估盒子是否包含猫。

## 如何测试

要测试盒子里是否有猫，可以使用以下任何方法。

### 打开盒子并观察其内部

这些说明假设盒子是用纸板制成的。您可能需要稍微修改步骤以适应其他材料。

[![盒子](images/box.jpg "一个由瓦楞纤维板制成的空盒子")](https://en.wikipedia.org/wiki/Box)\
*图 999.1-1：空开纸板盒子的图片*

使用以下步骤打开盒子。

1. 如果盒子用胶带封好，用刀割开胶带。小心不要把刀太深入盒子，万一它确实包含一只猫的话。
2. 如果盒子没有用胶带封好，抓住每个纸板瓣并打开它。

一旦盒子打开，观察盒子的内部以确定是否有猫在里面。

### 迫使猫现身

此测试基于观察盒子里猫的反应（如果有的话）。虽然这是一种有效的方法，但它不如第一种测试方法确定。

反应可能是：

- 喵喵声或小猫爪子轻轻移动的声音。
- 猫的各个部位（如耳朵、尾巴或爪子）从盒子的任何孔洞或开放区域出现。
- 一只猫从盒子里出来。

尝试使用以下任何策略来迫使盒子里的猫做出反应：

- 轻轻抓挠盒子的外部。
- 在盒子附近放置猫薄荷。
- 使用碗和一些玉米粒，复制将干猫粮倒入猫碗的声音。

如果没有猫现身，盒子里不太可能有猫。然而，这个测试是不确定的。很有可能我们可以说盒子里既有猫也没有猫。

[![GHZ 状态](images/ghz-state.svg "量子计算中 GHZ 状态的方程式")](https://en.wikipedia.org/wiki/Schr%C3%B6dinger%27s_cat)\
*图 999.1-2：量子计算"猫状态"或 GHZ 状态的方程式*

### 使用盒子相机

在盒子内设置无线摄像头以持续监控猫的存在。此测试需要以下步骤。

#### 1. 获取相机

相机必须：

1. 能装入盒子内。
2. 在盒子内为猫留出足够的空间。
3. 能够无线显示其视频馈送。

#### 2. 设置相机

使用提供的硬件或胶带将相机安装在盒子内部。按照制造商的说明设置相机并使视频馈送可用。例如，您可能能够通过以 `https://localhost:` 开头的地址在本地网络上查看视频馈送。

#### 3. 监控视频馈送

定期手动查看视频馈送，以确定是否有猫进入视野。或者，使用图像识别软件监控馈送并在检测到猫时发送警报。以下是一些用于定义模型的[相关 TensorFlow 代码](https://www.tensorflow.org/tutorials/images/segmentation#define_the_model)：

```py
base_model = tf.keras.applications.MobileNetV2(input_shape=[128, 128, 3], include_top=False)

# 使用这些层的激活
layer_names = [
    'block_1_expand_relu',   # 64x64
    'block_3_expand_relu',   # 32x32
    'block_6_expand_relu',   # 16x16
    'block_13_expand_relu',  # 8x8
    'block_16_project',      # 4x4
]
layers = [base_model.get_layer(name).output for name in layer_names]

# 创建特征提取模型
down_stack = tf.keras.Model(inputs=base_model.input, outputs=layers)

down_stack.trainable = False
```

如果您在视频馈送中观察到一只猫，那么盒子里很可能有一只猫。

## 捕获包含猫的输出

这是一个包含猫的适当缩写捕获输出示例。

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

## 相关测试用例

- [模板说明](2-Template_Explanation.md)
- [HTTP 请求和响应的格式](3-Format_for_HTTP_Request_Response.md)

## 修复

不要养成把猫放进盒子的习惯。尽可能让盒子远离猫。

## 工具

- 盒子
- 相机

## 参考资料

- [薛定谔的猫](https://en.wikipedia.org/wiki/Schr%C3%B6dinger%27s_cat)
