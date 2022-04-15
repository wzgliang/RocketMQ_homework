### 基于阿里云EventBridge，将上传到OSS的事件分析结果，推送到钉钉群
1. 在 `dafault` 事件总线中创建新的规则
2. 设置事件模式。通过匹配object信息中的mimeType筛选出上传的图片。
```json
{
    "source": [
        "acs.oss"
    ],
    "type": [
        "oss:ObjectCreated:PostObject"
    ],
    "data": {
        "oss": {
            "object": {
                "objectMeta": {
                    "mimeType": [
                        {
                            "prefix": "image"
                        }
                    ]
                }
            }
        }
    }
}
```
3. 设置事件目标。服务类型为钉钉。地址和密钥从钉钉群的机器人设置里获得。
>提取的变量如下：
```json
{
    "name":"$.data.oss.object.key",
  	"time":"$.data.eventTime",
    "size":"$.data.oss.object.size",
    "type":"$.data.oss.object.objectMeta.mimeType",
    "aliyunregionid":"$.aliyunregionid",
    "bucketname":"$.data.oss.bucket.name"
}
```
>发送的信息模板如下：
```json
{
    "msgtype": "markdown",
    "markdown": {
        "title":"Pic",
        "text": "#### 新的图片 \n ###### name: ${name} \n ###### size: ${size} \n ###### type: ${type} \n ###### time: ${time} \n  ![](https://${bucketname}.oss-${aliyunregionid}.aliyuncs.com/${name}) \n ###### BY：王昊然 "
    }
}
```
4.结果
![](pic01.png)