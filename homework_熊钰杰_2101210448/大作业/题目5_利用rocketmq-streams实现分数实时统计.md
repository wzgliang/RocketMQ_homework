### 数据源

```
{"name":"张三","class":"3班","subject":"数学","score":90}
{"name":"张三","class":"3班","subject":"历史","score":81}
{"name":"张三","class":"3班","subject":"英语","score":91}
{"name":"张三","class":"3班","subject":"语文","score":70}
{"name":"张三","class":"3班","subject":"政治","score":84}
{"name":"张三","class":"3班","subject":"地理","score":99}
{"name":"李四","class":"3班","subject":"数学","score":76}
{"name":"李四","class":"3班","subject":"历史","score":83}
{"name":"李四","class":"3班","subject":"英语","score":82}
{"name":"李四","class":"3班","subject":"语文","score":92}
{"name":"李四","class":"3班","subject":"政治","score":97}
{"name":"李四","class":"3班","subject":"地理","score":89}
{"name":"王五","class":"3班","subject":"数学","score":86}
{"name":"王五","class":"3班","subject":"历史","score":88}
{"name":"王五","class":"3班","subject":"英语","score":86}
{"name":"王五","class":"3班","subject":"语文","score":93}
{"name":"王五","class":"3班","subject":"政治","score":99}
{"name":"王五","class":"3班","subject":"地理","score":88}
```

### 代码
```java
public class FileSourceExample {
    public static void main(String[] args) {
        DataStreamSource source = StreamBuilder.dataStream("namespace", "pipeline"); 
        source.fromFile("data.txt", true)
                .map(message -> message)
                .groupBy("name")
                .filter(message -> ((JSONObject)message).getInteger("score") > 90 ) .selectFields("subject")
                .toPrint(1)
                .start();
} }

```

### 结果
```
{"name":"张三","class":"3班","subject":"数学","score":90}
{"name":"张三","class":"3班","subject":"英语","score":91}
{"name":"张三","class":"3班","subject":"地理","score":99}
{"name":"李四","class":"3班","subject":"语文","score":92}
{"name":"李四","class":"3班","subject":"政治","score":97}
{"name":"王五","class":"3班","subject":"语文","score":93}
{"name":"王五","class":"3班","subject":"政治","score":99}
```