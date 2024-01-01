### 使用Mybatis几个核心操作：

#### 导包，maven约定大于配置的问题

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.individuals.learn</groupId>
    <artifactId>MybatisStudy</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>


    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version> <!-- 或其他5.1.x版本 -->
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.13</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version> <!-- 使用最新版本 -->
        </dependency>

    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>

</project>
```

* 必备包：mysql驱动，mybatis包
* 可选包：lombok 常用注解@Data
* 约定大于配置：maven约定大于配置会造成运行时找不到资源文件，\<build\>标签中添加的内容是为了解决这个问题

#### xml配置文件

* 放置于resources下，习惯命名为mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:4528/test?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/individuals/mybatisStudy/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

#### 工具类获取SqlSession

* SqlSession是mybatis进行数据库操作的类

```java
package com.individuals.mybatisStudy.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {

    private static String resource;
    private static InputStream inputStream;
    private static SqlSessionFactory sqlSessionFactory;

    static {
        resource = "mybatis-config.xml";
        try {
            inputStream = Resources.getResourceAsStream(resource);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}

```

#### 实体类

* 实体类（Entity Class）是指代表数据库中的表的Java类。实体类的每个实例通常对应于数据库表中的一行数据。通过使用实体类，Java应用程序可以方便地与数据库进行交互，执行数据库操作（如插入、更新、查询和删除）

```java
package com.individuals.mybatisStudy.entity;

import lombok.Data;

@Data
public class User {
    private int id;
    private String name;
    private String pwd;

    public User(){

    }

    public User(int id,String name,String pwd){
        this.id=id;
        this.name=name;
        this.pwd=pwd;
    }
}

```

#### 操作数据库的接口

```java
package com.individuals.mybatisStudy.dao;

import com.individuals.mybatisStudy.entity.User;

import java.util.List;

public interface UserDAO {
    List<User> getUserList();
}

```

#### 接口的实现UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.individuals.mybatisStudy.dao.UserDAO">
    <select id="getUserList" resultType="com.individuals.mybatisStudy.entity.User">
        select * from test.user
    </select>
</mapper>
```

### 使用

```java
package com.individuals.mybatisStudy.dao;

import com.individuals.mybatisStudy.entity.User;
import com.individuals.mybatisStudy.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;


public class UserDAOTest {

    @Test
    public void test(){
        // 获取sqlsession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        // getmapper
        UserDAO userDAO = sqlSession.getMapper(UserDAO.class);
        List<User> userList = userDAO.getUserList();

        for (User user:userList){
            System.out.println(user);
        }
    }
}

```

