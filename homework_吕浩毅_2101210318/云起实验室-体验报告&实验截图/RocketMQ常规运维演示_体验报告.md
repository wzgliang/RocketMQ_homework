# RocketMQ常规运维演示_体验报告

## 启动集群

体验的过程中发现一个小bug

在启动集群，修改broker配置文件时：

用`vim`编辑器，键入`escape`键，期望从插入模式切换到普通模式，结果被当前打开的终端判定为窗口失焦。。。

最后，只能使用echo指令追加，``echo "brokerIP1=101.133.163.101" >> conf/broker.conf``

并在普通模式下删除原来的``brokerIP1=xxx.xxx.xxx.xxx``

## 动态修改Broker配置

一个小问题

查看broker全部配置的命令是

```shell
bin/mqadmin getBrokerConfig -b 127.0.0.1:30911
```

命令最后，监听的端口号，为之前启动集群时，broker的端口号

实验手册上

之前第0.小节中，启动集群时broker监听默认的是30911端口

在这一步之后是10911，需要自行修改



