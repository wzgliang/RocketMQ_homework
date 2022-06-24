# RocketMQ高级使用演示

## 实验环境

### **启动一个集群**

1. 启动namesrv。观察到启动成功的日志后， ctrl + c，终止日志输出。

```
cd /usr/local/services/5-rocketmq/namesrv-01

restart.sh
```

![image-20220523164931158](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523164931158.png)

2. 启动broker

修改broker配置项brokerIP1为**实验公网IP**，启动broker。观察到启动成功的日志后， ctrl + c，终止日志输出。

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523165054261.png" alt="image-20220523165054261" style="zoom:50%;" />



操作命令如下：

```
cd /usr/local/services/5-rocketmq/broker-01

vim ./conf/broker.conf 

restart.sh
```

![image-20220523165126281](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523165126281.png)



3. 启动dashboard

```
cd /usr/local/services/7-rocketmq-datashboard

./restart.sh
```

![image-20220523165210755](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523165210755.png)





4. 验证集群启动情况

访问 http://实验公网IP:30904#/cluster, 可以正常查看到集群节点信息，集群则是正常启动。

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523165320110.png" alt="image-20220523165320110" style="zoom:50%;" />



### **下载全部demo代码**

```
cd /data/demos

git clone https://github.com/ApacheRocketMQ/06-all-java-demos.git
```

**问题**

由于国内相关DNS污染严重，代码克隆速度过慢

使用镜像站点代替 https://hub.fastgit.xyz/

## 第一节、发送和消费并发消息

并发消息，也叫普通消息，是相对顺序消息而言的，普通消息的效率最高。

本次实验使用纯java client发送和消费消息

当前环境已经安装了一个1Namesrv + 1Broker的集群

### **1. 下载java代码demo（已下载则忽略操作）**

### **2. 打包，执行代码demo**

再执行命令， 可以看到正常生产和消费输出

```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876" -Dexec.mainClass="org.apache.rocketmqdemos.ConcurrentMessageDemo" -Dexec.classpathScope=runtime
```

打包结果

![image-20220523165816200](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523165816200.png)

运行结果

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523165937966.png" alt="image-20220523165937966" style="zoom:50%;" />

### **3. Demo代码说明**

Demo代码可以查看实验本地或者[github](https://github.com/ApacheRocketMQ/06-all-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/ConcurrentMessageDemo.java)。

并发消息，意思是生产者可以并发的向topic中发送消息， 消费端不区分顺序的消息，这种模式效率最好。生产者demo代码如下：

![img](https://ucc.alicdn.com/pic/developer-ecology/1c004658aba74842b80717a3b72ca87a.png)



最后留一个思考题给大家： 生产者实例和消费者实例，都是线程安全的吗？

是的。

---

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523170247225.png" alt="image-20220523170247225" style="zoom:50%;" />

我认为，此处注释有误，每一个生产者实例只发送了一条消息

模拟的十个生产者，总共发送了十条（并发的形式）

## 第二节、发送和消费顺序消息

### **1. 下载java代码demo（已下载则忽略操作）**

```
cd /data/demos

git clone https://github.com/ApacheRocketMQ/06-all-java-demos.gi
```

### **2. 打包，执行代码demo**

再执行命令， 可以看到正常生产和消费输出。 消费输出注意看相同queue id的消息输出内容中的数字，按照从小到大就是正确的。

```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876" -Dexec.mainClass="org.apache.rocketmqdemos.OrderMessageDemo1" -Dexec.classpathScope=runtime
```

结果

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523170727639.png" alt="image-20220523170727639" style="zoom:50%;" />

dashboard查询

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523171013623.png" alt="image-20220523171013623" style="zoom: 33%;" />

### **3. Demo代码说明**

Demo代码可以查看实验本地或者[github](https://github.com/ApacheRocketMQ/06-all-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/OrderMessageDemo1.java)。

- 生产者说明

生产者会根据设置的keys做hash，相同hash值的消息会发送到相同的queue中。所以相同hash值的消息需要保证在同一个线程中顺序的发送。

![img](https://ucc.alicdn.com/pic/developer-ecology/dd2a1f521e574af8a3cd6c67ca37d6eb.png)



- 消费者说明

消费者使用相对比较简单， 消息监听类实现org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly接口即可。相同queue的消息需要串行处理，这样救保证消费的顺序性

![img](https://ucc.alicdn.com/pic/developer-ecology/c1449a0e6b3f4811a9eaaa6fb4c17404.png)

## 第三节、发送和消费延迟消息

### **1. 下载java代码demo（已下载则忽略操作）**

```
cd /data/demos

git clone https://github.com/ApacheRocketMQ/06-all-java-demos.git
```

### **2. 打包，执行代码demo**

执行命令， 可以看到正常生产和消费输出。 目前[RocketMQ支持多种延迟级别](https://github.com/apache/rocketmq/blob/fd554ab12072225325c957ff6bdf492fc67821af/store/src/main/java/org/apache/rocketmq/store/config/MessageStoreConfig.java#L134), 不过每种延迟级别都是基于RocketMQ自身，实际延迟时间会加上Broker-Client端的网络情况不同而略有差异。

```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876" -Dexec.mainClass="org.apache.rocketmqdemos.DelayMessageDemo" -Dexec.classpathScope=runtime
```

输出

![image-20220523171151861](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523171151861.png)



### **3. Demo代码说明**

Demo代码可以查看实验本地或者[github](https://github.com/ApacheRocketMQ/06-all-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/DelayMessageDemo.java)。

- 生产者说明

生产者在发送消息的时候需要设置延迟级别，[RocketMQ支持多种延迟级别](https://github.com/apache/rocketmq/blob/fd554ab12072225325c957ff6bdf492fc67821af/store/src/main/java/org/apache/rocketmq/store/config/MessageStoreConfig.java#L134)。如果把延迟时间算作一个以空格分割的数组，延迟级别就是延迟时间数组的下标index+1。[RocketMQ如何解析延迟级别和延迟时间映射关系。](https://github.com/apache/rocketmq/blob/fd554ab12072225325c957ff6bdf492fc67821af/store/src/main/java/org/apache/rocketmq/store/schedule/ScheduleMessageService.java#L276)

![img](https://ucc.alicdn.com/pic/developer-ecology/8ab25ae897254e57a310f60e48cfd066.png)



- 消费者说明: 消费者按照并发消息消费即可

## 第四节、发送和消费事务消息

事务消息，是RocketMQ解决分布式事务的一种实现，极其简单好用。

一个事务消息大致的生命周期如下图

![img](https://ucc.alicdn.com/pic/developer-ecology/0d8963e1864c4d9a81a288722717186c.jpeg)

概括为如下几个重要点：

1. 生产者发送half消息（事务消息）
2. Broker存储half消息
3. 生产者处理本地事务，处理成功后commit事务
4. 消费者消费到事务消息

### **1. 下载java代码demo（已下载则忽略操作）**

```
cd /data/demos

git clone https://github.com/ApacheRocketMQ/06-all-java-demos.git
```

### **2. 打包，执行代码demo**

执行命令， 可以看到事务消息的全部过程。

```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876" -Dexec.mainClass="org.apache.rocketmqdemos.TransactionMessageDemo" -Dexec.classpathScope=runtime
```

控制台输出

![image-20220523171447014](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523171447014.png)

### **3. Demo代码说明**

Demo代码可以查看实验本地或者[github](https://github.com/ApacheRocketMQ/06-all-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/TransactionMessageDemo.java)。

在事物消息中，消费代码和普通消息的消费一样，主要代码在生产者端。

生产者端的主要代码包含3个步骤：

1. 初始化生产者，设置回调线程池、设置本地事物处理监听类。

这里注意事物消息的生产者类是: org.apache.rocketmq.client.producer.TransactionMQProducer, 而不是普通生产者类。

![img](https://ucc.alicdn.com/pic/developer-ecology/d132be57502b41b2be41bd923476d65d.png)



事物监听类需要实现2个方法，这里的逻辑都是mock的，实际使用的时候需要根据实际修改。

![img](https://ucc.alicdn.com/pic/developer-ecology/f7fa7783ce2845c0a3a9f85deced3c3c.png)



1. 发送事物消息。调用sendMessageInTransaction()方法发送事物消息， 而不是以前的send()方法。

![img](https://ucc.alicdn.com/pic/developer-ecology/f86df4f6dd084305aaa1a5a51c461f89.png)

### 问题

错别字

事物消息-->事务消息

## 第五节、生产者消费者如何同步发送、消费消息

Request-Reply

request-reply模式，可以满足目前类似RPC同步调用的场景

### **1. 下载java代码demo（已下载则忽略操作）**

```
cd /data/demos

git clone https://github.com/ApacheRocketMQ/06-all-java-demos.git
```

### **2. 打包，执行代码demo**

执行命令， 可以看到正常生产和消费输出。



```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876" -Dexec.mainClass="org.apache.rocketmqdemos.RequestReplyMessageDemo" -Dexec.classpathScope=runtime
```



通过代码结果和代码比较， 我们得知request-reply类似RPC同步调用的效果。

控制台输出

![image-20220523171933772](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523171933772.png)



### **3. Demo代码说明**

Demo代码可以查看实验本地或者[github](https://github.com/ApacheRocketMQ/06-all-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/RequestReplyMessageDemo.java)。

request-reply模式，在生产者和消费者两端都和一般的生产消费有区别，下面分别介绍下demo代码。

生产者demo主要代码, 主要区别在于调用request()，而不是send()方法。

![img](https://ucc.alicdn.com/pic/developer-ecology/c186ea06275143119371a6a1bcf2d823.png)



消费者demo主要代码: 消费代码主要增加了“回复”逻辑。回复是利用消息发送直接向生产者发送一条消息。 有点类似事物消息中broker回查生产者。

![img](https://ucc.alicdn.com/pic/developer-ecology/141953bb1b784c188feef57ba803a441.png)



### 一个小问题

事物消息和request-reply消息时，生产者的生产者组名有什么要求嘛？

## 第六节、如何有选择性的消费消息

有时候我们只想消费部分消息， 当然全部消费，在代码中过滤。 假如消息海量时， 会有很多资源浪费，比如浪费不必要的带宽。我们可以通过tag，sql92表达式来选择性的消费

### **0. 启动一个集群**

- 进入broker目录

```
cd /usr/local/services/5-rocketmq/broker-01
```

- 编辑配置文件，修改broker配置

```
vim conf/broker.conf
```

增加配置项值：

```
// 是否支持重试消息也过滤
filterSupportRetry=true

// 支持属性过滤
enablePropertyFilter=true
```

修改后：

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523172123426.png" alt="image-20220523172123426" style="zoom:50%;" />

- 重启broker

```
./restart.sh
```

![image-20220523172148342](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523172148342.png)

### **1. 下载java代码demo（已下载则忽略操作）**

```
cd /data/demos

git clone https://github.com/ApacheRocketMQ/06-all-java-demos.git
```

### **2. 打包，执行tag过滤代码demo**

执行命令， 可以看到正常生产和消费输出

```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876 tag" -Dexec.mainClass="org.apache.rocketmqdemos.FliterMessageDemo" -Dexec.classpathScope=runtime
```

### **3. 执行sql过滤代码demo**

执行命令， 可以看到正常生产和消费输出。



```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876 sql" -Dexec.mainClass="org.apache.rocketmqdemos.FliterMessageDemo" -Dexec.classpathScope=runtime
```



控制台输出

![image-20220523172512629](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523172512629.png)



### **4. Demo代码说明**

Demo代码可以查看实验本地或者[github](https://github.com/ApacheRocketMQ/06-all-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/FliterMessageDemo.java)。以下分别介绍生产者和消费者主要demo代码。

- 生产者

在生产tag消息的时候， 消息中需要加上发送tag；sql92过滤的时候，加上自定义k-v。

![img](https://ucc.alicdn.com/pic/developer-ecology/4a3d15e433114b6b9dbfbf642a808139.png)





- 消费者

tag过滤消费时，在订阅topic时， 也添加上tag订阅

![img](https://ucc.alicdn.com/pic/developer-ecology/17f8ad29b0c74fd0a8dc6860bf6ce2a0.png)



SQL92过滤时，添加上SQL过滤订阅。至于SQL92除了等号，还是支持什么，大家可以自行自行查看或者到群里问。

![img](https://ucc.alicdn.com/pic/developer-ecology/e5949d15d3234ad59192bbed61b28083.png)

## 第七节、如何使用ACL客户端生产、消费消息

ACL，全称是Access Control List，是RocketMQ设计来做访问和权限控制的。更多文档参见github wiki: https://github.com/apache/rocketmq/wiki/RIP-5-RocketMQ-ACL

### **0. 启动一个集群**

- 进入broker目录

```
cd /usr/local/services/5-rocketmq/broker-01
```

- 编辑配置文件，修改broker配置

```
vim conf/broker.conf
```



增加配置项

```
aclEnable=true
```

修改后：

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523172645631.png" alt="image-20220523172645631" style="zoom:50%;" />



- 重启broker

```
./restart.sh
```

![image-20220523172714793](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523172714793.png)



### **1. 下载java代码demo（已下载则忽略操作）**

```
cd /data/demos

git clone https://github.com/ApacheRocketMQ/06-all-java-demos.git
```

### **2. 打包，执行代码demo**

执行命令， 可以看到正常生产和消费输出。 demo代码使用的admin权限发送和消费，实际使用需要对于每个topic，消费者组授权，才能正常生产消费。



```
// 进入demo代码目录
cd /data/demos/06-all-java-demos/

// 打包
mvn clean package

// 运行代码
mvn exec:java -Dexec.args="127.0.0.1:39876" -Dexec.mainClass="org.apache.rocketmqdemos.ACLDemo" -Dexec.classpathScope=runtime
```

![image-20220523172924716](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-23/image-20220523172924716.png)

### **3. Demo代码说明**

Demo代码可以查看实验本地或者[github](https://github.com/ApacheRocketMQ/06-all-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/ACLDemo.java)。带ACL的生产者和消费者在初始化的时候，都必须给一个hook实例，构建方法如下：

```
static RPCHook getAclRPCHook(String accessKey, String secretKey) {
      return new AclClientRPCHook(new SessionCredentials(accessKey, secretKey));
}
```

在broker端secret key用来校验信息的完整性， access key用来校验用户权限。二者缺一不可。
