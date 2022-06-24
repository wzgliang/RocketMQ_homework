# RocketMQ常规运维演示_实验截图

## 一、编译、部署

启动集群

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624085447799.png" alt="image-20220624085447799" style="zoom:33%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624085504451.png" alt="image-20220624085504451" style="zoom:33%;" />



修改dashboard的namesrv地址

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624083408546.png" alt="image-20220624083408546" style="zoom:50%;" />

编译结果

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624083614262.png" alt="image-20220624083614262" style="zoom:33%;" />

启动dashboard

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624085830514.png" alt="image-20220624085830514" style="zoom:33%;" />

## 二、扩容

创建topic

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624090028971.png" alt="image-20220624090028971" style="zoom:33%;" />

扩容，增加读写队列数目

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624090137904.png" alt="image-20220624090137904" style="zoom:33%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624090200359.png" alt="image-20220624090200359" style="zoom:33%;" />



## 三、动态修改broker配置

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624091751591.png" alt="image-20220624091751591" style="zoom:33%;" />

更新MsgsNumBatch的值为128

```
bin/mqadmin updateBrokerConfig -b 127.0.0.1:30911 -k maxMsgsNumBatch -v 128
```

修改成功

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624091909464.png" alt="image-20220624091909464" style="zoom:33%;" />

再次查询

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624092042061.png" alt="image-20220624092042061" style="zoom:33%;" />

本地持久化保存的配置文件

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-06-24/image-20220624092223440.png" alt="image-20220624092223440" style="zoom:33%;" />