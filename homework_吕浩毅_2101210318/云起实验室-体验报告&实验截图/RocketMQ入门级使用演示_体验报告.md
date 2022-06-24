# RocketMQ入门级使用演示_体验报告

### 修改日志配置、jvm配置等其他配置

> **实验环境下建议至少修改JVM的配置**
>
> 减少分配的堆最大值

### 使用spring接入时

克隆代码后

执行命令

```shell
export namesrv_ip=101.133.134.92

mvn exec:java -Dexec.args="$namesrv_ip:9876" -Dexec.mainClass="org.apache.rocketmqdemos.Startup" -Dexec.classpathScope=runtime
```

demo中配置的nameserver的地址为127.0.0.1:39876

实际运行时，将端口号改成了9876

解决方案：

1. 修改命令行指令
2. 修改配置文件

如果希望按照云起实验室的实验报告，可以一步步做下来，则最好修改demo的配置文件

已经提了ISSUE和PR

ISSUE：https://github.com/ApacheRocketMQ/02-spring-demos/issues/1#issue-1236241465

PR：https://github.com/ApacheRocketMQ/02-spring-demos/pull/2#issue-1236241791

