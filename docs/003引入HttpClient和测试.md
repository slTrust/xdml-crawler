## 项目目标

- 爬取新浪新闻页面，爬取文章 https://sina.cn/
    - 新闻站基本不做反爬
    - 它有手机站
    - 它的字符编码是 utf-8，这一点非常难能可贵
    
## 第一行代码、CI配置以及git协作

> 001 引入 httpclient 发请求

- google 搜索 [httpclient maven](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient)
- 得到maven坐标，放到你的 pom.xml里
```
<!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.9</version>
</dependency>
```
- 搜索 [httpclient apache](https://hc.apache.org/)
    - 找到 [HttpClient](https://hc.apache.org/httpcomponents-client-ga/index.html)
    - [quick start](https://hc.apache.org/httpcomponents-client-ga/quickstart.html)
    - 抄代码
```
public class Main {
    public static void main(String[] args) throws IOException {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("https://sina.cn/");
        CloseableHttpResponse response1 = httpclient.execute(httpGet);
        try{
            System.out.println(response1.getStatusLine());
            HttpEntity entity1 = response1.getEntity();
            EntityUtils.consume(entity1);
        } finally {
            response1.close();
        }
    }
}
```
- 注意这里的代码可以优化 try 这个地方
    - 每当你看到黄色标示的东西，意思是可以简化代码 alt + enter
    - 'try' with resources

```
public class Main {
    public static void main(String[] args) throws IOException {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("https://sina.cn/");
        try (CloseableHttpResponse response1 = httpclient.execute(httpGet)) {
            System.out.println(response1.getStatusLine());
            HttpEntity entity1 = response1.getEntity();
            System.out.println(EntityUtils.toString(entity1));
        }
    }
}
```

此时跑代码是 OK的

### 引入测试依赖到 pom.xml

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.4.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.4.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

> 冒烟测试

- https://zhuanlan.zhihu.com/p/39786718
```
package com.github.hcsp;
import org.junit.jupiter.api.Test;
public class SmokeTest  {
    @Test
    public void test() {
    }
}
```
- 运行 `mvn test`
    - 你会得到 BUILD SUCCESS

