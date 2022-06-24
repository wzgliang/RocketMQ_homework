# 数据

{"InFlow":"1","ProjectName":"ProjectName-0","LogStore":"LogStore-0","OutFlow":"0"}
{"InFlow":"2","ProjectName":"ProjectName-1","LogStore":"LogStore-1","OutFlow":"1"}
{"InFlow":"3","ProjectName":"ProjectName-2","LogStore":"LogStore-2","OutFlow":"2"}
{"InFlow":"4","ProjectName":"ProjectName-0","LogStore":"LogStore-0","OutFlow":"3"}
{"InFlow":"5","ProjectName":"ProjectName-1","LogStore":"LogStore-1","OutFlow":"4"}
{"InFlow":"6","ProjectName":"ProjectName-2","LogStore":"LogStore-2","OutFlow":"5"}
{"InFlow":"7","ProjectName":"ProjectName-0","LogStore":"LogStore-0","OutFlow":"6"}
{"InFlow":"8","ProjectName":"ProjectName-1","LogStore":"LogStore-1","OutFlow":"7"}
{"InFlow":"9","ProjectName":"ProjectName-2","LogStore":"LogStore-2","OutFlow":"8"}
{"InFlow":"10","ProjectName":"ProjectName-0","LogStore":"LogStore-0","OutFlow":"9"}

# 处理代码

```java
class FileSourceExample {
public static void main(String[] args) {
    DataStreamSource source = StreamBuilder.dataStream("namespace", "pipelin source.fromFile("data.txt", true)
.map(message -> message)
.filter(message -> ((JSONObject)message).getInteger("InFlow") > .map(message -> ((JSONObject message).getString("ProjectName"))
.toFile("./result.txt")
.start();
}
}
```

# 结果

ProjectName-0
ProjectName-0