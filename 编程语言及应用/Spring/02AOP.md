### 代理模式

代理模式是SpringAOP的底层!

* 代理模式定义:为其他对象提供一种代理以控制对这个对象的访问
* 意义:代理模式可以在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。
* 代理模式优点:
  * 可以使真实角色的操作更加纯粹,不去关心一些公共业务
  * 公共业务交给代理角色,实现角色分工
  * 公共业务发生变化时方便拓展
* 缺点:每个真实角色都有一个代理,增加了开发工作量

#### 静态代理

角色分析:

* 抽象角色:一般会使用接口或者抽象类来解决
* 真实角色:被代理的角色
* 代理角色:代理真实角色,代理真实角色后,我们一般会做一些附属操作
* 客户:访问代理对象的人

```java
//Client
package com.individuals.springStudy.demo;

public class Client {
    public static void main(String[] args) {
        Host host = new Host();

        //代理,中介帮房东出租,但是代理角色有附属(额外操作)
        Proxy proxy = new Proxy(host);

        //不用面对房东,直接找中介
        proxy.rent();
    }
}

//Host
package com.individuals.springStudy.demo;

public class Host implements Rent{

    @Override
    public void rent() {
        System.out.println("出租");
    }
}

//Proxy
package com.individuals.springStudy.demo;

public class Proxy {
    private Host host;

    public Proxy(){

    }
    public Proxy(Host host){
        this.host = host;
    }

    public void rent(){
        host.rent();
    }

    //看房
    public void  seeHouse(){
        System.out.println("带你看房");
    }
}

//Rent
package com.individuals.springStudy.demo;

public interface Rent {
    public void rent();
}

```

#### 动态代理

* 动态代理和静态代理的角色一样
* 动态代理的代理类是动态生成的,不是我们直接写好的
* 动态代理分为两大类:基于接口的动态代理,基于类的动态代理
  * 基于接口--JDK动态代理
  * 基于类:cglib
  * java字节码实现:javassist

```java
public interface Rent {
    public void rent();
}

public class Host implements Rent {

    @Override
    public void rent() {
        System.out.println("出租");
    }
}

public class Client {
    public static void main(String[] args) throws Throwable {
        Host host = new Host();
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        pih.setRent(host);
        Rent proxy = (Rent) pih.getProxy();
        proxy.rent();

    }
}

public class ProxyInvocationHandler implements InvocationHandler {

    //被代理的接口
    private Rent rent;

    public void setRent(Rent rent) {
        this.rent = rent;
    }

    //生存得到代理类
    public Object getProxy() throws Throwable {
        // 三个参数:类加载器 实现的接口 实现InvocationHandler的类
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), rent.getClass().getInterfaces(),this);
    }

    //处理代理实例,并返回结果
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //动态代理的本质,就是使用反射机制实现
        Object result = method.invoke(rent, args);

        return result;
    }
}
```

### AOP

术语解释:

* 横切关注点:日志 安全 缓存等和业务逻辑无关的部分
* 切面(ASPECT): 横切关注点被模块化的特殊对象,一个类
* 通知(Advice):切面必须要完成的工作,类中的一个方法
* 目标(Target):被通知对象
* 代理(Proxy):项目表对象应用通知之后创建的对象
* 切入点(PointCut):切面通知 执行的"地点"的定义
* 连接点(JointPoint):与切入点匹配的执行点 

#### 使用Spring API接口实现

```java
package com.individuals.springStudy.service;

public interface UserService {
    public void add();
    public void delete();
    public void update();
    public void select();
}

package com.individuals.springStudy.service;

public class UserServiceImpl implements UserService{

    @Override
    public void add() {
        System.out.println("add");
    }

    @Override
    public void delete() {
        System.out.println("delete");
    }

    @Override
    public void update() {
        System.out.println("update");
    }

    @Override
    public void select() {
        System.out.println("select");
    }
}

package com.individuals.springStudy.log;

import org.springframework.aop.AfterReturningAdvice;

import java.lang.reflect.Method;

public class AfterLog implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        System.out.println("执行"+method.getName()+"方法,返回结果为"+o);
    }
}

package com.individuals.springStudy.log;

import org.springframework.aop.MethodBeforeAdvice;

import java.lang.reflect.Method;

public class Log implements MethodBeforeAdvice {

    // 参数:要执行的目标对象的方法,参数,目标
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("Before Method: " + method.getName()+"即将执行");
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="com.individuals.springStudy.service.UserServiceImpl"/>
    <bean id="log" class="com.individuals.springStudy.log.Log"/>
    <bean id="afterLog" class="com.individuals.springStudy.log.AfterLog"/>

    <aop:config>
        <!--切入点:expression:表达式,execution(要执行的位置)-->
        <aop:pointcut id="pointcut" expression="execution(* com.individuals.springStudy.service.UserServiceImpl.*(..))"/>

        <!-- 执行环绕增加 -->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

#### 自定义切面(配置)

```java
package com.individuals.springStudy.diy;

// 横切关注点
public class DiyPointCut {
    public void before(){
        System.out.println("before");
    }
    public void after(){
        System.out.println("after");
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="com.individuals.springStudy.service.UserServiceImpl"/>
    <bean id="log" class="com.individuals.springStudy.log.Log"/>
    <bean id="afterLog" class="com.individuals.springStudy.log.AfterLog"/>


    <bean id="diy" class="com.individuals.springStudy.diy.DiyPointCut"/>

    <aop:config>
        <!-- 自定义切面 -->
        <aop:aspect ref="diy">
            <!-- 切入点 -->
            <aop:pointcut id="point" expression="execution(* com.individuals.springStudy.service.UserServiceImpl.*(..))"/>
            <!-- 通知 -->
            <aop:before method="before" pointcut-ref="point" />
            <aop:after method="after" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
</beans>
```

#### 自定义切面(注解)

```java
package com.individuals.springStudy.diy;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect  // 切面类
public class AnnotationPointCut {

    @Before("execution(* com.individuals.springStudy.service.UserServiceImpl.*(..))")
    public void before() {
        System.out.println("Before Method");
    }

    @After("execution(* com.individuals.springStudy.service.UserServiceImpl.*(..))")
    public void after() {
        System.out.println("After Method");
    }

    @Around("execution(* com.individuals.springStudy.service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Before Around");
        Signature signature = joinPoint.getSignature();
        System.out.println("Signature: " + signature);
        // 执行方法
        Object proceed = joinPoint.proceed();
        System.out.println("After Around");
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="com.individuals.springStudy.service.UserServiceImpl"/>
    <bean id="log" class="com.individuals.springStudy.log.Log"/>
    <bean id="afterLog" class="com.individuals.springStudy.log.AfterLog"/>


    <bean id="annotationPointCut" class="com.individuals.springStudy.diy.AnnotationPointCut"/>
    <aop:aspectj-autoproxy/>
</beans>
```

