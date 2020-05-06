## mybatis

mybatis是java世界 orm框架的事实标准,如果你用orm跑不了的要用 mybatis

> 引入 mybatis

- mybatis maven

```
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.2</version>
</dependency>
```

> 看文档

- https://mybatis.org/mybatis-3/zh/index.html
- 参考[我这个](https://sltrust.github.io/2019/10/31/ZB-034-01MyBatis/)也可以
- 你的资源文件路径`resources/db/mybatis/config.xml`
- 然后抄文档 配置数据库连接串

MyBatisCrawlerDao.java 实现

```
public class MyBatisCrawlerDao implements CrawlerDao {
    private SqlSessionFactory sqlSessionFactory;

    public MyBatisCrawlerDao() {
        try {
            String resource = "db/mybatis/config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public synchronized String getNextLinkThenDelete() throws SQLException {
        try (SqlSession session = sqlSessionFactory.openSession(true)) {
            String url = session.selectOne("com.github.hcsp.MyMapper.selectNextAvailableLink");
            if (url != null) {
                session.delete("com.github.hcsp.MyMapper.deleteLink", url);
            }
            return url;
        }
    }

    @Override
    public void insertNewsIntoDatabase(String url, String title, String content) throws SQLException {
        try (SqlSession session = sqlSessionFactory.openSession(true)) {
            session.insert("com.github.hcsp.MyMapper.insertNews", new News(url, title, content));
        }
    }

    @Override
    public boolean isLinkProcessed(String link) throws SQLException {
        try (SqlSession session = sqlSessionFactory.openSession()) {
            int count = (Integer) session.selectOne("com.github.hcsp.MyMapper.countLink", link);
            return count != 0;
        }
    }

    @Override
    public void insertProcessedLink(String link) {
        Map<String, Object> param = new HashMap<>();
        param.put("tableName", "links_already_processed");
        param.put("link", link);
        try (SqlSession session = sqlSessionFactory.openSession(true)) {
            session.insert("com.github.hcsp.MyMapper.insertLink", param);
        }
    }

    @Override
    public void insertLinkToBeProcessed(String link) {
        Map<String, Object> param = new HashMap<>();
        param.put("tableName", "links_to_be_processed");
        param.put("link", link);
        try (SqlSession session = sqlSessionFactory.openSession(true)) {
            session.insert("com.github.hcsp.MyMapper.insertLink", param);
        }
    }
}
```

MyMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.github.hcsp.MyMapper">
    <select id="selectNextAvailableLink" resultType="String">
      select link
      from LINKS_TO_BE_PROCESSED
      LIMIT 1
    </select>

    <delete id="deleteLink" parameterType="String">
        DELETE
        FROM LINKS_TO_BE_PROCESSED
        where link = #{link}
    </delete>

    <insert id="insertNews" parameterType="com.github.hcsp.News"
    >
      insert into news (url, title, content, created_at,MODIFIED_AT)values(#{url},#{title},#{content},now(),now())
    </insert>

    <select id="countLink" parameterType="String" resultType="int">
        select count(link)
        from LINKS_ALREADY_PROCESSED
        where link = #{link}
    </select>

    <insert id="insertLink" parameterType="HashMap">
        insert into
        <choose>
            <when test="tableName == 'links_already_processed'">
                links_already_processed
            </when>
            <otherwise>
                links_to_be_processed
            </otherwise>
        </choose>
        (link)
        values #{link}
    </insert>

</mapper>
```


### 总结

> 注意事项1: mybatis 的表名问题

- 不能 `#{tableName}` 这样参数化
- 必须使用 `<choose>`等条件语句才能实现表名的动态化

> 注意事项2:`<when test="tableName == 'links_already_processed'">`

- 这里的`links_already_processed` 必须用单引号包裹
    - 因为你不用单引号 mybatis认为他是 变量名
        - param是 map，就会去 param 里找key为 `links_already_processed` 的值
        - param是 News 这样的实体类,就会遵循 javaBean约定去找对应的 get 方法

> Mybatis

- 动态sql是 mybatis的灵魂，是它的精髓