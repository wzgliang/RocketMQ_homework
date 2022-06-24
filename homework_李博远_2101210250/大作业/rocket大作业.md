随着各行各业移动互联和云计算技术的普及发展，大数据计算已深入人心，最常见的比如 flink、spark 等。这些大数据框架，采用中心化的 Master-Slave 架构，依赖和部署比较重，每个任务也有较大开销，有较大的使用成本。[RocketMQ](https://so.csdn.net/so/search?q=RocketMQ&spm=1001.2101.3001.7020) Streams 着重打造轻量计算引擎，除了消息队列，无额外依赖，对过滤场景做了大量优化，性能提升 3-5 倍，资源节省 50%-80%。

RocketMQ Streams 适合大数据量->高过滤->轻窗口计算的场景，核心打造轻资源，高性能优势，在资源敏感场景中有很大优势，最低 1core，1g 可部署，建议的应用场景（安全，风控，边缘计算，消息队列流计算）。

RocketMQ Streams 兼容 Blink(Flink 的阿里内部版本) 的 SQL，UDF/UDTF/UDAF，多数 Blink 任务可以直接迁移成 RocketMQ Streams 任务。将来还会发布和 Flink 的融合版本，RocketMQ Streams 可以直接发布成 Flink 任务，既可以享有 RocketMQ Streams 带来的高性能、轻资源，还可以和现有的 Flink 任务统一运维和管理。

**环境要求**

1）JDK 1.8 版本以上；

2）Maven 3.2 版本以上。

**DSL SDK**

利用 DSL SDK 开发实时任务时，需要做如下的一些准备工作：

- 依赖准备

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-streams-clients</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

准备工作完成后，就可以直接开发自己的实时程序。

- 代码开发

```bash
DataStreamSource source=StreamBuilder.dataStream("namespace","pipeline");
source.fromFile("～/admin/data/text.txt",false).map(message->message + "--")
.toPrint(1)
.start();
```

其中：

1）Namespace 是业务隔离的，相同的业务可以写成相同的Namespace。相同的Namespace 在任务调度里可以跑在进程里，也可以共享一些配置；

2）pipelineName 可以理解成就是 job name ，唯一区分 job；

3）DataStreamSource 主要是创建 Source，然后这个程序运行起来，最终的结果就是在原始的消息里面会加"--"，然后把它打印出来。

- 部署执行

基于 DSL SDK 完成开发，通过下面命令打成 jar 包，执行 jar，或直接执行任务的 main 方法。

```properties
mvn -Prelease-all -DskipTests clean install -U
java -jar jarName mainClass &
```

 **SQL SDK**

- 依赖准备

```xml
<dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>rsqldb-clients</artifactId>
      <version>1.0.0-SNAPSHOT</version>
</dependency>
```

- 代码开发

首先开发业务逻辑代码， 可以保存为文件也可以直接使用文本：

```sql
CREATE FUNCTION json_concat as 'xxx.xxx.JsonConcat';

CREATE TABLE `table_name` (
    `scan_time` VARCHAR,
    `file_name` VARCHAR,
    `cmdline` VARCHAR,
) WITH (
     type='file',
     filePath='/tmp/file.txt',
     isJsonData='true',
     msgIsJsonArray='false'
);
-- 数据标准化
create view data_filter as
select
     *
from (
    select
        scan_time as logtime
        , lower(cmdline) as lower_cmdline
        , file_name as proc_name
    from
        table_name
)x
where
    (
        lower(proc_name) like '%.xxxxxx'
        or lower_cmdline  like 'xxxxx%'
        or lower_cmdline like 'xxxxxxx%'
        or lower_cmdline like 'xxxx'
        or lower_cmdline like 'xxxxxx'
    )
;

CREATE TABLE `output` (
     `logtime` VARCHAR
    , `lower_cmdline` VARCHAR
    , `proc_name` VARCHAR
) WITH (
    type = 'print'
);
 
 
insert into output
select
    *
from
    aegis_log_proc_format_raw
;
```

其中：

1）CREATE FUNCTION：引入外部的函数来支持业务逻辑， 包括 flink 以及系统函数；

2）CREATE Table：创建 source/sink；

3）CREATE VIEW：执行字段转化，拆分，过滤；

4）INSERT INTO：数据写入 sink；

5）函数：内置函数，udf 函数。

- SQL 扩展

RocketMQ streams 支持三种 SQL 扩展能力，具体实现细节请看：

1）通过 Blink UDF/UDTF/UDAF 扩展 SQL 能力；

2）通过 RocketMQ streams 扩展 SQL 能力，只要实现函数名是 eval 的 java bean 即可；

3）通过现有 java 代码扩展 SQL 能力，create function 函数名就是 java 类的方法名。

- SQL 执行

可以从下载最新的 RocketMQ Streams 代码并构建。

```properties
cd rsqldb/
mvn -Prelease-all -DskipTests clean install -U
cp rsqldb-runner/target/rocketmq-streams-sql-{版本号}-distribution.tar.gz 部署的目录
```

解压 tar.gz 包， 进入目录结构

```properties
tar -xvf rocketmq-streams-{版本号}-distribution.tar.gz
cd rocketmq-streams-{版本号
```

其目录结构如下 

1）bin 指令目录，包括启动和停止指令

2）conf 配置目录，包括日志配置以及应用的相关配置文件

3）jobs 存放 sql，可以两级目录存储

4）ext 存放扩展的 UDF/UDTF/UDAF/Source/Sink

5）lib 依赖包目录

6）log 日志目录

- 执行 SQL

```sql
#指定 sql 的路径，启动实时任务
bin/start-sql.sh sql_file_path
```

- 执行多个 SQL

如果想批量执行一批 SQL，可以把 SQL 放到 jobs 目录，最多可以有两层，把 sql 放到对应目录中，通过 start 指定子目录或 sql 执行任务。

- 任务停止

```sql
# 停止过程不加任何参数，则会将目前所有运行的任务同时停止
bin/stop.sh
# 停止过程添加了任务名称， 则会将目前运行的所有同名的任务都全部停止
bin/stop.sh sqlname
```

- 日志查看

目前所有的运行日志都会存储在 log/catalina.out 文件中。

