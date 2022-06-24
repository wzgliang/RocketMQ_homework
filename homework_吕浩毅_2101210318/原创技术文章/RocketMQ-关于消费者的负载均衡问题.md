# RocketMQ-关于消费者的负载均衡问题

本文是基于初次接触RocketMQ的新手视角，来展开的

我觉得非常有必要记录一下，这次的分析思路和解决问题的方式

## 抛出问题

==“如果把消费者打成jar包，在本地同时运行两个消费者进程，观察终端输出，解释一下原因”==

下面开始设计实验，分析原因

## 实验参数设置

topic: lhy-test-topic

topic维持的队列数目

- read-queue：16

- write-queue：16

consumerGroup: lhy-test-consumerGroup 

## 测试

### 第一轮

先发送六条消息

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314094521856.png" alt="image-20220314094521856" style="zoom:50%;" />

启动pushConsumer 接收到6条消息

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314094614700.png" alt="image-20220314094614700" style="zoom:50%;" />

### 第二轮

再启动一个消费者进程后，发送六条消息

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314094647507.png" alt="image-20220314094647507" style="zoom:50%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314094825329.png" alt="image-20220314094825329" style="zoom:50%;" />

消费者消费情况：

第一次注册的消费者并未接收到任何消息

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314095054243.png" alt="image-20220314095054243" style="zoom:50%;" />

后注册的消费者收到了后续六条消息。

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314094905615.png" alt="image-20220314094905615" style="zoom:50%;" />

### 第三轮

再次尝试，一共发送12条消息

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314095311904.png" alt="image-20220314095311904" style="zoom:50%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314095643410.png" alt="image-20220314095643410" style="zoom:50%;" />

接收情况

pushConsumer1

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314100110257.png" alt="image-20220314100110257" style="zoom:50%;" />

pushConsumer2

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314095929462.png" alt="image-20220314095929462" style="zoom:50%;" />



接收消息情况并无明显规律，类似随机消费？

## 猜想

仔细分析接收情况，从队列id上看，每个消费者维持一组queue-id，且存在一定规律：

- 数量上：*两个消费者平分所有队列？
- id上：id是分块连续的 例如：pushconsumer1：[0、1、2]、[1、2、3、4、5]

根据以上规律，我决定将topic的队列数量改为1试试

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314100758437.png" alt="image-20220314100758437" style="zoom:50%;" />



<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314100825318.png" alt="image-20220314100825318" style="zoom:50%;" />

结果发现所有的消息都被consumer1消费

再次发送消息测试

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314100925459.png" alt="image-20220314100925459" style="zoom:50%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314100957433.png" alt="image-20220314100957433" style="zoom:50%;" />

依旧全部被consumer1消费

单从测试的结果来看：

- 消费的消息所属的队列id有部分连续性的规律*
- 从数量上看，有均分消息的趋势

- 当topic的队列数量(:1)小于消费者数量(:2)时，所有的消息都被其中一个消费者consumer1消费了

猜想，消费者消费消息的状况，跟topic设置的队列有关？

## 查阅资料

之前简单学习过Kafka，里面有分区partition的概念

为了详细了解RocketMQ的消息队列queue，查询了相关资料

[订阅关系一致-阿里云](https://help.aliyun.com/document_detail/43523.html#section-yrz-pzr-w40)

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314103452582.png" alt="image-20220314103452582" style="zoom:50%;" />

[RocketMq中topic被多个queue分片，那么consumer是如何拿到完整的topic的呢？ - 中间件兴趣圈的回答 - 知乎](https://www.zhihu.com/question/448951368/answer/1776558336)

查阅资料得知：

- 同一个ConsumerGroup的consumer必须消费同一个Topic
- 一个Topic维持多个队列，类似Kafka的分区
- 一个队列只能分配给一个消费者
- 如果队列数量小于消费者数量，部分消费者将无法接收消息

**再次实验测试**

这次我将topic的队列数量设置为两个

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314104355168.png" alt="image-20220314104355168" style="zoom:50%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314104406733.png" alt="image-20220314104406733" style="zoom:50%;" />

发现两个消息队列被两个消费者均分

再次测试

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314104521765.png" alt="image-20220314104521765" style="zoom:50%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314104545091.png" alt="image-20220314104545091" style="zoom:50%;" />

两个PushConsumer，分别分配到了队列0和队列1，并且*一直是如此的分配关系*（之后通过源码阅读得知，是因为每次分配前都进行了排序，所以分配关系不变）

每个consumer接收到了一半的消息，并且队列内的消息是有序的

## 分析源码

有了具体的思路和基础知识，下面对源码进行分析

在`Consumer.java`中，`cmd+b`转到`DefaultMQPushConsumer`的声明处

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314113128758.png" alt="image-20220314113128758" style="zoom:50%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314113206003.png" alt="image-20220314113206003" style="zoom:50%;" />

翻译过来是：平均队列分配算法（均衡队列分配算法？）

继续定位

`org/apache/rocketmq/client/consumer/rebalance/AllocateMessageQueueAveragely.java`

> 详细分析写在了注释中

```java
/**
 * Average Hashing queue algorithm
 */
public class AllocateMessageQueueAveragely implements AllocateMessageQueueStrategy {
    private final InternalLogger log = ClientLogger.getLog();

    @Override
    public List<MessageQueue> allocate(String consumerGroup, String currentCID, List<MessageQueue> mqAll,
        List<String> cidAll) {
        // 检查当前cunsumer的 ID是否为空
        if (currentCID == null || currentCID.length() < 1) {
            throw new IllegalArgumentException("currentCID is empty");
        }
        // 检查当前Topic的queue列表是否非空
        if (mqAll == null || mqAll.isEmpty()) {
            throw new IllegalArgumentException("mqAll is null or mqAll empty");
        }
        // 检查所有的consumer ID列表
        if (cidAll == null || cidAll.isEmpty()) {
            throw new IllegalArgumentException("cidAll is null or cidAll empty");
        }
		// 用ArrayList存储queue的分配结果
        List<MessageQueue> result = new ArrayList<MessageQueue>();
        if (!cidAll.contains(currentCID)) {
            log.info("[BUG] ConsumerGroup: {} The consumerId: {} not in cidAll: {}",
                consumerGroup,
                currentCID,
                cidAll);
            return result;
        }
		// 当前消费者在消费者列表中的索引
        int index = cidAll.indexOf(currentCID);
        // 模值取 当前topic队列数 % 消费者总数目
        int mod = mqAll.size() % cidAll.size();
     	// 计算当前消费者分配的队列数
        int averageSize =
            mqAll.size() <= cidAll.size() ? 1 : (mod > 0 && index < mod ? mqAll.size() / cidAll.size()
                + 1 : mqAll.size() / cidAll.size());
        // 计算分配的队列起始索引
        int startIndex = (mod > 0 && index < mod) ? index * averageSize : index * averageSize + mod;
        // 计算分配的队列索引范围
        int range = Math.min(averageSize, mqAll.size() - startIndex);
        // 分配
        for (int i = 0; i < range; i++) {
            result.add(mqAll.get((startIndex + i) % mqAll.size()));
        }
        return result;
    }

    @Override
    public String getName() {
        return "AVG";
    }
}

```

### 具体计算逻辑

1. 首先 mod = topic维持的总队列数 % 消费者总数
2. 如果topic维持的队列数 小于或等于 当前消费者总数，分配的尺寸为 1，即每个消费者分配1个queue
3. 如果topic维持的队列数 大于 当前消费者总数
	     
	1. 如果当前消费者的index < mod，分配的队列个数就为 topic维持的总队列数 / 消费者总数 + 1
		
	2. 如果当前消费者的index >= mod, 分配的队列个数就为 topic维持的总队列数 / 消费者总数

4. 然后根据此计算 分配的队列起始索引，进行分配

### 举个例子

如果一开始设置`Topic`维持的队列数为16，当前总共有3个消费者

 >mqAll.size() = 16;
 >
 >cidAll.size() = 3;
 >
 >这时无法平均分配，即每个消费者分到5个队列后，还会剩一个
 >
 >此时，均衡分配算法会将**余出的队列**按序号从**小到大再分配**一次

具体过程：

计算得到的`mod=16 % 3 = 1`，如果当前为第一个消费者，其索引`index=0`，0小于1，

则为其分配 `16/3+1=6`个队列，队列索引为[0,1,2,3,4,5]

其余的两个消费者，索引`index=1,2`>=1，分配`16/3=5`个队列

则分配情况如下

消费者1: [0, 1, 2, 3, 4, 5]

消费者2: [6, 7, 8, 9, 10]

消费者3: [11,12,13,14,15]

### 实验验证

>Topic维持的队列数量: 16
>
>消费者数目: 3
>
>生产消息数目：32条

发送消息

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314124615990.png" alt="image-20220314124615990" style="zoom:50%;" />

消费消息

**消费者1接收情况**

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314124911405.png" alt="image-20220314124911405" style="zoom:50%;" />

分配到的queue ID分别为0,1,2,3,4,5，一共6个队列，符合之前的分析

**消费者2接收情况**

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314124955947.png" alt="image-20220314124955947" style="zoom:50%;" />

分配到的queue ID分别为6,7,8,9,10，一共5个队列，符合之前的分析

**消费者3接收情况**

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314125151030.png" alt="image-20220314125151030" style="zoom:50%;" />

分配到的queue ID分别为11,12,13,14,15，一共5个队列，符合之前的分析

---

### 打断点，调试一下

调试时的参数设置：

> 当前Topic维持的队列数量: 2
>
> 消费者数目: 3

根据之前的分析，由于队列数目小于消费者数目，第三个消费者会无法获取消息。

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314125715887.png" alt="image-20220314125715887" style="zoom:50%;" />

开始debug后，消费者总数目是3，总共只有两个队列可供分配

当前消费者index=2，经过计算，起始的分配索引为4，`range=-2`实际上根本不会为这个消费者分配队列，也就是当前消费者无法获取消息。

继续F8，步过，会进入`org/apache/rocketmq/client/impl/consumer/RebalanceImpl.java`

进行再均衡，重新分配队列。实际上其他消费者还在线的话，当前消费者仍不会分配到队列:)

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314121320907.png" alt="image-20220314121320907" style="zoom:50%;" />



## 总结

- 每个Topic维持多个队列queue，队列内消息是有序的
- 同一个ConsumerGroup的消费者只能订阅一个topic
- 一个队列只能分配给一个消费者
- pushConsumer默认的队列分配算法是：均衡队列分配算法`AllocateMessageQueueAveragely`
- 如果一个topic内的队列数目小于消费这个topi的消费者总数，会有消费者无法接收到消息

消息队列的分配涉及到consumer端的负载均衡

之后继续阅读`Rebalance`相关代码后得知，每当集群中的消费者实例数量发生变化，都会进行一次负载均衡：

将消息队列按照队列分配算法分配给每个消费者实例，默认是平均分配算法。

## 后话

rocketmq有五种消息队列分配算法

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314131621806.png" alt="image-20220314131621806" style="zoom: 50%;" />

都是实现了`AllocateMessageQueueStrategy`接口

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-04-27/image-20220314131759600.png" alt="image-20220314131759600" style="zoom:50%;" />
