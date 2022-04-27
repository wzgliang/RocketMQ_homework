## 题目

开放题：基于阿里云EventBridge，将上传到OSS的事件分析结果，推送回本钉钉群。

例如：监听OSS的事件，把上传到OSS的图片文件的名称、格式、大小、图片内容发送回钉钉群，其余类型文件则不处理

## 实现

### 准备工作

先在钉钉电脑客户端，钉钉群中，新建钉钉机器人

- 保存Webhook地址
- 保存密钥

### Envent Bridge配置

登陆阿里云Envent Bridge控制台

1. 选择事件总线-default
2. 点击事件规则，创建规则

#### 编辑事件目标

输入webhook地址和密钥

变量映射

```json
{
    "name": "$.data.oss.object.key",
    "size": "$.data.oss.object.size",
    "bucketName": "$.data.oss.bucket.name",
    "region": "$.data.region",
    "time": "$.data.eventTime",
    "type": "$.data.oss.object.objectMeta.mimeType"
}
```

自定义模版

```json
{
    "msgtype": "markdown",
    "markdown": {
        "title": "上传图片",
        "text": "### 上传了一张新图片\n > #### 上传者: 吕浩毅\n > #### 大小: ${size}B\n > #### 格式: ${type}\n > #### 上传时间: ${time}\n > ![test](https://${bucketName}.oss-${region}.aliyuncs.com/${name})"
    }
}
```

#### 编辑事件模式

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220427142707374.png" alt="image-20220427142707374" style="zoom:50%;" />

监听`oss:ObjectCreated:PutObject`事件

### 上传图片测试

我监听的是PutObeject，即上传文件事件

在本地创建一个java项目，使用SDK进行文件上传

参考[官方文档](https://help.aliyun.com/document_detail/84781.htm?spm=a2c4g.11186623.0.0.2ec77dbbU6jUyi#t22269.html)

代码如下

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220427143456758.png" alt="image-20220427143456758" style="zoom:50%;" />

## 结果展示

[图片地址](https://temp-for-event-1.oss-cn-shanghai.aliyuncs.com/end.jpg)

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220427143319812.png" alt="image-20220427143319812" style="zoom:50%;" />