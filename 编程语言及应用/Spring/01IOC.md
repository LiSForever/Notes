### 控制反转(从spring获取对象)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 使用无参构造,name可以取多个别名 -->
    <bean id="hello" class="com.individuals.springStudy.Hello" name="hello1,hello2,hello3">
        <property name="str" value="Spring"/>
    </bean>
    
    <!-- 有参构造,参数按index传递 -->
    <bean id="user" class="com.individuals.springStudy.User">
        <constructor-arg index="0" value="lda"/>
        <constructor-arg index="1" value="24"/>
    </bean>
    
    <!-- 有参构造,参数按类型传递,多个同类型参数无法使用 -->
    <bean id="user" class="com.individuals.springStudy.User">
        <constructor-arg type="int" value="24"/>
        <constructor-arg type="java.lang.String" value="lda"/>
    </bean>
    
    <!-- 有参构造,参数按参数名传递 -->
    <bean id="user" class="com.individuals.springStudy.User">
        <constructor-arg name="age" value="24"/>
        <constructor-arg name="name" value="lda"/>
    </bean>
    
    <!-- 别名,原名和别名都可以取得对象 -->
    <alias name="hello" alias="hello2"/>


</beans>
```

```java
import com.individuals.springStudy.Hello;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class myTest {
    public static void main(String[] args) {
        // 获取spring的上下文对象,beans.xml中的类已经实例化
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 从spring中取出所需对象
        Hello hello = (Hello) context.getBean("hello");
        hello.show();
    }
}
```

###  依赖注入(初始化类)

#### 构造器注入

#### Set注入

* 依赖注入
  * 依赖:bean对象的创建依赖于容器
  * 注入:bean中对象的所有属性由容器来注入

* 复杂类的注入

```java
package com.individuals.springStudy;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbys;
    private Map<String,String> card;
    private Set<String> games;
    private String wife;
    private Properties info;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    public String[] getBooks() {
        return books;
    }

    public void setBooks(String[] books) {
        this.books = books;
    }

    public List<String> getHobbys() {
        return hobbys;
    }

    public void setHobbys(List<String> hobbys) {
        this.hobbys = hobbys;
    }

    public Map<String, String> getCard() {
        return card;
    }

    public void setCard(Map<String, String> card) {
        this.card = card;
    }

    public Set<String> getGames() {
        return games;
    }

    public void setGames(Set<String> games) {
        this.games = games;
    }

    public String getWife() {
        return wife;
    }

    public void setWife(String wife) {
        this.wife = wife;
    }

    public Properties getInfo() {
        return info;
    }

    public void setInfo(Properties info) {
        this.info = info;
    }
}

```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

  
    <bean id="address" class="com.individuals.springStudy.Address"/>
    <bean id="student" class="com.individuals.springStudy.Student">
        <property name="name" value="lda"/>
        <property name="address" ref="address"/>
        <property name="books">
            <array>
                <value>wujia</value>
                <value>web</value>
                <value>nixiang</value>
            </array>
        </property>
        <property name="hobbys">
            <list>
                <value>game</value>
                <value>code</value>
            </list>
        </property>
        <property name="card">
            <map>
                <entry key="IDCard" value="666666"/>
                <entry key="bankCard" value="88888888"/>
            </map>
        </property>
        <property name="games">
            <set>
                <value>lol</value>
                <value>cf</value>
            </set>
        </property>
        <property name="wife">
            <null/>
        </property>
        <property name="info">
            <props>
                <prop key="xuehao">12345678</prop>
                <prop key="banji">6.1</prop>
                <prop key="sex">male</prop>
            </props>
        </property>
    </bean>

</beans>
```

### bean的作用域

```xml
<!-- 单例模式(singleton 默认) -->
<bean id="address" class="com.individuals.springStudy.Address" scope="singleton"/>

<!-- 原型模式,每次从容器中get时,都会产生一个新对象 -->
<bean id="address" class="com.individuals.springStudy.Address" scope="prototype"/>
```

* 还有request session application在web开发中使用

### Bean的自动装配

* Spring会在上下文中自动寻找,并自动给Bean装配属性
* Spring中的三种装配方式
  * xml显示配置
  * java中显示配置
  * 隐式的自动装配

#### byName byType自动装配

```java
package com.individuals.springStudy;

public class People {
    private String name;
    private Cat cat;
    private Dog dog;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }
}

```

* 所有bean的id唯一,而且注入的bean要和自动注入的属性的set要求的参数类型一致
* 所有bean的class唯一,而且注入的bean要和自动注入的属性的set要求的参数类型一致

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cat" class="com.individuals.springStudy.Cat"/>
    <bean id="dog" class="com.individuals.springStudy.Dog"/>
    <bean id="people" class="com.individuals.springStudy.People" autowire="byName">
        <property name="name" value="lda"/>
    </bean>

</beans>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cat" class="com.individuals.springStudy.Cat"/>
    <bean id="dog" class="com.individuals.springStudy.Dog"/>
    <bean id="people" class="com.individuals.springStudy.People" autowire="byType">
        <property name="name" value="lda"/>
    </bean>

</beans>
```

#### 注解实现自动装配

* 不用实现set,但是属性名需要一致

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
    <bean class="com.individuals.springStudy.Cat"/>
    <bean class="com.individuals.springStudy.Dog"/>
    <bean class="com.individuals.springStudy.People"/>
</beans>
```

```java
package com.individuals.springStudy;

import org.springframework.beans.factory.annotation.Autowired;

public class People {
    private String name;
    @Autowired
    private Cat cat;
    @Autowired
    private Dog dog;

    public String getName() {
        return name;
    }


    public Cat getCat() {
        return cat;
    }

   
    public Dog getDog() {
        return dog;
    }

}

```

* @Qualifier解决冲突问题,显示指定注入的bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
    <bean id="dog1" class="com.individuals.springStudy.Cat"/>
    <bean id="dog2" class="com.individuals.springStudy.Cat"/>
    <bean class="com.individuals.springStudy.Dog"/>
    <bean class="com.individuals.springStudy.People"/>
</beans>
```

```java
package com.individuals.springStudy;

import org.springframework.beans.factory.annotation.Autowired;

public class People {
    private String name;
    @Autowired
    private Cat cat;
    @Autowired
    @Qualifier(value="dog1")
    private Dog dog;

    public String getName() {
        return name;
    }


    public Cat getCat() {
        return cat;
    }

   
    public Dog getDog() {
        return dog;
    }

}

```

### Spring注解开发 

* @Autowired
* @Resource也是自动装配,采用byName byType 也可指定@Resource(name="")

* @Nullable 标记字段可以为null
* @Scope 标记Bean的作用域

#### 通过@Component让Spring托管Bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 扫描指定的包,使注解生效 -->
    <context:component-scan base-package="com.individuals.springStudy.pojo"/>


    <context:annotation-config/>
</beans>
```

```java
package com.individuals.springStudy.pojo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {
    @Value("lda")
    public String name;
}
 
```

#### @Component的衍生

* @Repository 用于DAO层,作用等同于@Component
* @Service 用于Service层,同上
* @Controller 用于Controller层,同上

### 完全使用Java配置Spring

#### @Configuration配置java Bean

```java
package com.individuals.springStudy.pojo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {

    public String name;

    public String getName() {
        return name;
    }

    @Value("lda")
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

```java
package com.individuals.springStudy.config;

import com.individuals.springStudy.pojo.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class userConfig {

    // 相当于<bean id="getUser" class="User"/>,class为返回值
    @Bean
    public User getUser(){
        return new User();
    }
}

```

```java
import com.individuals.springStudy.config.userConfig;
import com.individuals.springStudy.pojo.User;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class configTest {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(userConfig.class);
        User user = context.getBean("getUser", User.class);
        System.out.println(user);
    }
}

```

