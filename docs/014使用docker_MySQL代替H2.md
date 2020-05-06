## 使用MySQL 替换H2数据库

> 十分推荐 docker

```
docker run -v "$PWD/data":/var/lib/mysql --name my-mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=news -p 3306:3306 -d mysql:5.7.27

# -v 代表持久化数据目录 下次删了容器还能继续运行
```

> 替换数据库配置文件 config.xml

- mysql driver class

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/news?characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="db/mybatis/MyMapper.xml"/>
    </mappers>
</configuration>
```

> 安装依赖,修改flyway信息

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.17</version>
</dependency>

。。。

 <plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>5.2.4</version>
    <configuration>
        <url>jdbc:mysql://localhost:3306/news</url>
        <user>root</user>
        <password>root</password>
    </configuration>
</plugin>
```

> 通过 flyway 构建数据库

```
# 清楚数据库内容
mvn flyway:clean 

# 初始化数据库迁移脚本
mvn flyway:migrate

mvn flyway:clean && mvn flyway:migrate
```

> 运行程序

报错信息
```
Exception in thread "main" org.apache.ibatis.exceptions.PersistenceException: 
### Error updating database.  Cause: java.sql.SQLSyntaxErrorException: Table 'news.news' doesn't exist
### The error may exist in db/mybatis/MyMapper.xml
### The error may involve com.github.hcsp.MyMapper.insertNews-Inline
### The error occurred while setting parameters
### SQL: insert into news (url, title, content, created_at, MODIFIED_AT)         values (?, ?, ?, now(), now())
### Cause: java.sql.SQLSyntaxErrorException: Table 'news.news' doesn't exist
```

- 表名大写～～
    - 原因是 mysql 表名区不分区分大小写是可以配置的

> 再次运行程序

- 结果 News 里数据都是乱码

### Mysql著名的坑

**mysql的utf-8不是正经的utf-8,mysql正经的utf-8是 utf-8mb4**

- 帮当前数据库字符集改成utf-8的

> 搜索  jdbc mysql utf-8

- https://stackoverflow.com/questions/5405236/jdbc-mysql-utf-8-string-writing-problem

> 搜索 jdbc mysql utf8 alter database

- https://mathiasbynens.be/notes/mysql-utf8mb4

```
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

**结果你重新看程序运行后 news里还是乱码**

- 因为你只改了数据库的字符集，没改连接数据库字符串的字符集设置

```
<url>jdbc:mysql://localhost:3306/news?characterEncoding=utf-8</url>
```

