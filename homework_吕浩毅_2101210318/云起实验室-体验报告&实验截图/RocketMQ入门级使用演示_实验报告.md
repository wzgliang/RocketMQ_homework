# RocketMQ入门级使用演示

> 姓名: 吕浩毅
>
> 学号: 2101210318

## 前言

[体验地址-云起实验室](https://developer.aliyun.com/adc/scenario/47efb0ab5a9741448e7a3e999336022e)

强烈建议老师增加每次体验的时长，一个小时内，实在来不及体验➕写实验报告

## 第一节：下载、编译最新版RocketMQ

> 大部分内容在之前的课程中已经实操过

### 配置环境

> 云起实验室中环境已经配置完毕

命令行输入 `echo -e  "$(git --version)\n$(mvn --version)\n$JAVA_HOME"`

进行检查

输出如下：

![image-20220515141010825](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515141010825.png)

没问题

### 下载源码

进入代码目录

`cd /tiger/tmp`

克隆代码

`git clone --branch release-4.9.3 https://github.com/apache/rocketmq.git`

**报错**

![image-20220515142059138](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515142059138.png)

---

尝试了一些解决办法后，决定曲线救国

在本地执行 `git clone --branch release-4.9.3 git@github.com:apache/rocketmq.git`通过`ssh`方式下载源码

然后压缩 `tar -zcvf code.tar.gz ./rocketmq`

使用`scp`命令传到服务器上

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515142855593.png" alt="image-20220515142855593" style="zoom:33%;" />

`scp ~/workspace/tmp/code.tar.gz root@139.224.249.109:/tiger/tmp/`

回到云起实验室

解压缩 `tar -xvf code.tar.gz && rm -rf ./code.tar.gz`

进入源代码目录 `cd ./rocketmq`

目录内文件如下

> ps: `tree`指令可以通过 `yum install tree`下载

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515143347469.png" alt="image-20220515143347469" style="zoom:50%;" />

### 编译并打包源码

使用`maven`

执行 `mvn -Prelease-all -DskipTests clean install -U`

控制台输出

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515143755786.png" alt="image-20220515143755786" style="zoom:67%;" />

打包结果：

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515143835411.png" alt="image-20220515143835411" style="zoom:67%;" />

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515143943224.png" alt="image-20220515143943224" style="zoom:67%;" />

## 第二节：部署一个简单的RocketMQ集群

### 安装Namesrv, Broker

创建部署临时目录

`mkdir -p /tiger/rocketmq/namesrv1 && mkdir -p /tiger/rocketmq/broker1`

拷贝rocketmq-4.9.4-SNAPSHOT里面的内容，分别拷贝到 `/tiger/rocketmq/namesrv1, /tiger/rocketmq/broker1,`

执行命令`cp -R /tiger/tmp/rocketmq/distribution/target/rocketmq-4.9.4-SNAPSHOT/rocketmq-4.9.4-SNAPSHOT/* /tiger/rocketmq/namesrv1 && cp -R /tiger/tmp/rocketmq/distribution/target/rocketmq-4.9.4-SNAPSHOT/rocketmq-4.9.4-SNAPSHOT/* /tiger/rocketmq/broker1`

结果如下

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515144542843.png" alt="image-20220515144542843" style="zoom:67%;" />

### 修改日志配置、jvm配置等其他配置

> **实验环境下建议至少修改JVM的配置**
>
> 减少分配的堆最大值

#### 修改namesrv日志配置

进入namesrv部署根目录, 修改日志配置文件

主要修改点：日志默认存储路径， 保存天数，每个日志文件大小等

执行命令

```shell
cd /tiger/rocketmq/namesrv1/

vim conf/logback_namesrv.xml
```

例如，将日志默认存储路径修改为 `/tiger/logs/...`

在vim命令模式输入 `%s/${user.home}/\/tiger/g`

批量修改完成后，效果如下

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515153437449.png" alt="image-20220515153437449" style="zoom:67%;" />

#### 修改namesrv JVM配置

进入namesrv部署根目录，修改JVM配置

`vim bin/runserver.sh`

![image-20220515145805993](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515145805993.png)

#### 同理修改broker的配置

> 修改点类似，具体的配置文件不同

- 日志配置文件

	```
	vim conf/logback_broker.xml
	```

- JVM配置文件

	```
	vim bin/runbroker.sh
	```

​	<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515152318463.png" alt="image-20220515152318463" style="zoom:67%;" />

### 启动集群并测试

#### 启动nameserver

进入安装目录

`cd /tiger/rocketmq/namesrv1`

启动，在后台运行

`nohup sh bin/mqnamesrv > output.file 2>&1 &`

> ps: output.file 为控制台输出

`tail -f ./output.file`查看控制台输出

启动成功

![image-20220515151516237](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515151516237.png)

#### 启动broker

进入安装目录

`cd /tiger/rocketmq/broker1`

启动

`nohup sh bin/mqbroker -n localhost:9876 > output.file 2>&1 &`

启动成功

![image-20220515152426275](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515152426275.png)

#### 检查相关进程

`ps aux|grep rocketmq`

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515152606946.png" alt="image-20220515152606946" style="zoom:67%;" />

### 发送、消费消息

#### 设置namesrv环境变量

`export NAMESRV_ADDR=localhost:9876`

进入broker安装目录

`cd /tiger/rocketmq/broker1`

#### 发送消息

执行命令

`sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer`

发送成功截图

![image-20220515153826312](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515153826312.png)

#### 消费消息

执行命令

`sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer`

消费成功截图

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515153931517.png" alt="image-20220515153931517" style="zoom:67%;" />

## 第三节：如何使用Java客户端发送和消费消息

示例代码的[GitHub仓库](https://github.com/ApacheRocketMQ/01-java-demos)

### 克隆01-java-demo

进入实验环境的demo目录

`cd /data/demos`

> 下载demo代码
>
> `git clone https://github.com/ApacheRocketMQ/01-java-demos.git`
>
> *实验环境中已经下载好了

### maven打包并执行demo

打包 `mvn clean package`

查看当前ECS的公网IP

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515155027911.png" alt="image-20220515155027911" style="zoom:67%;" />

执行命令

```shell
export namesrv_ip=101.133.129.237

mvn exec:java -Dexec.args="$namesrv_ip:9876" -Dexec.mainClass="org.apache.rocketmqdemos.Startup" -Dexec.classpathScope=runtime
```

控制台输出

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515155459479.png" alt="image-20220515155459479" style="zoom:67%;" />

### java client 代码

关于RocketMQ的java client 使用，以生产者为例子

代码位置： `01-java-demos/blob/main/src/main/java/org/apache/rocketmqdemos/Startup.java`

```java
public static void startOneProducer() throws InterruptedException, MQClientException {
    // 1. 启动一个生产者实例
    PRODUCER = new DefaultMQProducer(PRODUCER_GROUP_NAME);

    // 2. 设置生产者参数
    PRODUCER.setNamesrvAddr(NAMESRV_ADDRESSES);

    // 3. 启动生产者
    PRODUCER.start();
    System.out.printf("Producer Started.%n");

    // 4. 发送消息
    for (int i = 1; i <= MESSAGE_COUNT; i++) {
        try {
            Message msg = new Message(TOPIC_NAME, "TagA",
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
            );
            SendResult res = PRODUCER.send(msg);
            System.out.printf("[正在发送消息] %s = %s\n", res.getMsgId(), new String(msg.getBody()));
        } catch (Exception e) {
            e.printStackTrace();
            Thread.sleep(1000);
        }
    }// end for

    System.out.printf("\n\n\n%s", "");
}
```

大致流程：

1. 先创建一个生产者实例
2. 配置相关参数，例如nameserver address
3. 然后启动生产者
4. 执行发送消息的逻辑，本demo中，发送了10条消息
	1. topic: "tiger_topic_01"
	2. tag: "TagA"

## 第四节：使用Spring接入

> 由于时长限制，本节开启的一次新实验

示例代码的[GitHub仓库](https://github.com/ApacheRocketMQ/02-spring-demos)

### 克隆02-spring-demos

进入实验环境的demo目录

`cd /data/demos`

下载代码

```shell
git clone https://github.com/ApacheRocketMQ/02-spring-demos.git
cd ./02-spring-demos/
```

### 打包并执行demo

打包 `mvn clean package`

查看当前ECS的IP

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515163307640.png" alt="image-20220515163307640" style="zoom:67%;" />

执行命令

```shell
export namesrv_ip=101.133.134.92

mvn exec:java -Dexec.args="$namesrv_ip:9876" -Dexec.mainClass="org.apache.rocketmqdemos.Startup" -Dexec.classpathScope=runtime
```

> **PS**:demo中配置的nameserver的地址为127.0.0.1:39876
>
> 实际运行时，将端口号改成了9876

控制台输出

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515165637404.png" alt="image-20220515165637404" style="zoom:67%;" />

### Demo代码

摘抄自实验手册

#### 生产者使用

1. 创建RoketMQ客户端模版对象：RocketMQTemplate, 并且注入namesrv等参数
2. 调用RocketMQTemplate实例的方法

#### 消费者使用

继承RocketMQListener类，实现onMessage()方法即可

## 第5节：使用Golang接入

示例代码的[GitHub仓库](https://github.com/ApacheRocketMQ/03-golang-demo)

### 克隆示例代码

进入实验环境的demo目录

`cd /data/demos`

下载代码

```shell
git clone https://github.com/ApacheRocketMQ/03-golang-demo.git
cd ./03-golang-demo/main
```

### 打包并执行demo

进入`main`文件夹，打包 `go build`

执行

```shell
./main $namesrv_ip:9876
```

控制台输出

![image-20220515171038691](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515171038691.png)

![image-20220515171116855](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515171116855.png)

### Demo代码

跟Java client 使用方法类似，区别在于，生产者和消费者的参数是在创建时，进行设置

以生产者为例

![image-20220515182218814](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515182218814.png)

## 第6节：使用Python接入

示例代码的[GitHub仓库](https://github.com/ApacheRocketMQ/04-python-demo)

### 环境配置

> 实验环境已经配置好
>
> 这里摘抄自实验手册

#### python2.7环境

~~python2？？？~~

#### 安装cpp动态库

```shell
wget https://github.com/apache/rocketmq-client-cpp/releases/download/2.0.0/rocketmq-client-cpp-2.0.0-centos7.x86_64.rpm
sudo rpm -ivh rocketmq-client-cpp-2.0.0-centos7.x86_64.rpm
```

#### 安装RocketMQ python客户端

```shell
pip install rocketmq-client-python
```

### 克隆示例代码

进入实验环境的demo目录

`cd /data/demos`

下载代码

```shell
git clone https://github.com/ApacheRocketMQ/04-python-demo.git
cd ./04-python-demo/src
```

### 执行demo

生产者

`python producer.py 101.133.134.92:9876`

![image-20220515170738472](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515170738472.png)

消费者

`python consumer.py 101.133.134.92:9876`

![image-20220515170821503](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515170821503.png)

### Demo代码

使用Python接入RocketMQ的方式跟Java client完全一致

同样以生产者为例

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515183158481.png" alt="image-20220515183158481" style="zoom:67%;" />

1. 初始化producer实例

2. 设置参数

3. 启动生产者

3. 执行发送逻辑

## 第7节：使用C++接入

示例代码的[GitHub仓库](https://github.com/ApacheRocketMQ/05-cpp-demo)

### 环境配置

> 实验环境已经配置好
>
> 这里摘抄自实验手册

#### 安装g++,g++,make

```shell
yum install gcc gcc-c++ make -y 
```

#### 安装cpp动态库

```shell
wget https://github.com/apache/rocketmq-client-cpp/releases/download/2.0.0/rocketmq-client-cpp-2.0.0-centos7.x86_64.rpm
sudo rpm -ivh rocketmq-client-cpp-2.0.0-centos7.x86_64.rpm
```

### 克隆示例代码

进入实验环境的demo目录

`cd /data/demos`

下载代码

```shell
git clone https://github.com/ApacheRocketMQ/05-cpp-demo.git
cd ./05-cpp-demo
```

### 执行demo

使用`make`编译并打包

`make clean && make`

执行demo

`./main 101.133.134.92:9876`

控制台输出如下

![image-20220515171508978](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515171508978.png)

### Demo代码

使用C++接入RocketMQ的方式跟Java client完全一致

下面稍微修改了一下发送逻辑

<img src="https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515172442170.png" alt="image-20220515172442170" style="zoom:67%;" />

修改发送消息逻辑后，控制台输出如下：

![image-20220515172101178](https://lhy-oss-tuchuang.oss-cn-beijing.aliyuncs.com/uPic/2022-05-15/image-20220515172101178.png)