---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - spring
---
#          spring中的aop（Aspect Oriented Programming）(切面编程)

##          1、前言

在认识aop之前我们需需要了解一下动态代理（基于基础不扎实的原因，可能缺失）

####          1、什么是代理：

*、在以前，我们买东西是从厂商直接拿：消费者----->厂商（生产和销售），这样的弊端就是厂商的成本太高，既要关乎生产，又要关乎销售。

*、逐渐的，我们到现在的消费模式是：消费者--->销售商（代理商）---->厂商，这样就减少了厂商的运营成本。

####          2、java中的代理模式：

以上的模式咱们也可以在java代码中实现，在什么情况实现呢呢，举个例子：咱们使用JDBC进行数据持久化的时候，我们在正式持久化之前，需要先加载驱动，建立连接；在持久化完成之后，我们需要关闭连接，释放资源；此类的重复的工作，我们可以建立一个工具类帮助我们完成。

######          1、静态代理

实现原理：

​	这里我就不写静态代理的源码了，大概含义就是，咱们写一个jdbc的实现类，对每个方法执行前，咱们先执行需要执行的工作，比如加载驱动，建立连接，执行之后咱们关闭连接，释放资源，这样我们的重复代码肯定会减少特别多的冗余，使代码的可读性更高，到这里，咱们就有了aop的意思了。

弊端：

​	如果持久化中咱们有n个需要进行的动作，那么咱们就需要写n个实现类去进行切面式的代理，而且如果临时更改一些驱动，那么咱们就需要对**源码进行修改，耦合性比较高**。

######          2、动态代理

实现原理：

  所谓动态代理，就是让静态代理中，需要创建实现类的方法，得到了优化，在哪里实现某个动作，就在哪里去代理这个对象，即：**随用随代理，并且实现了不改变源码的情况下对原有方法进行增强**

实现代码：使用的就是jdk官方提供的接口动态代理Proxy.newProxyInstance,去编写咱们的动态代理

~~~java
AccountService accountService1 = (AccountService)Proxy.newProxyInstance(accountService.getClass().getClassLoader(), accountService.getClass().getInterfaces(), new InvocationHandler() {
            /**
             *原生动态代理代码  接口动态代理  提供者  jdk官方
             * @param proxy 代理对象的引用
             * @param method 代理的方法
             * @param args 参数列表
             * @return
             * @throws Throwable
             */
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object rtValue = null;
                try {
                    System.out.println("前置 原生接口动态代理");
                    if ("saveAccount".equals(method.getName())) {
                        rtValue = method.invoke(accountService,args);
                    }
                    System.out.println("后置 原生接口动态代理");
                    return rtValue;
                } catch (Throwable t) {
                    System.out.println("异常 原生接口动态代理");
                } finally {
                    System.out.println("最终 原生接口动态代理");
                }
                return rtValue;
            }
        });
        accountService1.saveAccount();
~~~

这样就实现了，随用随代理，对源码并没有改动。

##          2、spring中的aop

与动态代理模式一个思想，用spring的方法去实现动态代理

####          1、思想和源码

话不多说 上代码   该例子是模拟一个账户的添加修改删除操作，并且在这些进行这些操作时，记录日志。

1、beans.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop  http://www.springframework.org/schema/aop/spring-aop.xsd
">


    <!--开启ioc注解扫描-->
    <context:component-scan base-package="cn.tfs"/>
    
    <!--开启aop注解扫描-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

    <!--手动编写aop实现的xml-->
    <!--<aop:config>-->
        <!--<aop:pointcut id="pt1" expression="execution(* cn.tfs.service.impl.*.*(..))"/>-->
        <!--<aop:aspect ref="logger">-->
            <!--<aop:before method="beforeLog" pointcut-ref="pt1"/>-->
            <!--<aop:after-returning method="afterReturningLog" pointcut-ref="pt1"/>-->
            <!--<aop:after-throwing method="afterThrowingLog" pointcut-ref="pt1"/>-->
            <!--<aop:after method="afterLog" pointcut-ref="pt1"/>-->
            <!--&lt;!&ndash;<aop:around method="AroundLog" pointcut-ref="pt1"/>&ndash;&gt;-->
        <!--</aop:aspect>-->
    <!--</aop:config>-->




</beans>
~~~

2、AccountService.java

~~~java
package cn.tfs.service;

import org.springframework.stereotype.Service;

public interface AccountService {
    void saveAccount();
    void updateAccount(int i);
    int deleteAccount();
}

~~~

3、AccountServiceImpl.java

~~~java
package cn.tfs.service.impl;

import cn.tfs.service.AccountService;
import org.springframework.stereotype.Service;

@Service("accountService")
public class AccountServiceImpl implements AccountService {
    public void saveAccount() {
        System.out.println("保存了账户....");
//       int i= 1/0;
    }

    public void updateAccount(int i) {
        System.out.println("更新了账户....");

    }

    public int deleteAccount() {
        System.out.println("删除了账户....");
        return 0;
    }
}

~~~

4、TestAop.java

~~~java
public class TestAop {

public static void main(){
AccountService accountService = new ClassPathXmlApplicationContext("beans.xml").getBean("accountService",AccountService.class);
        accountService.saveAccount();
	}
}
~~~

5、pom.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.tfs</groupId>
    <artifactId>aop</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.7</version>
        </dependency>
    </dependencies>
</project>
~~~

6、Log.java

~~~java
package cn.tfs.utils;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * 日志通知类  模拟aop实现切面编程
 */
@Component("logger")
@Aspect
public class Log {
    //到方法名： 访问修饰符 返回值 全限定类名.方法名(参数列表)
    @Pointcut("execution(* cn.tfs.service.impl.*.*(..))")
    public void po1(){ }
    //前置通知
    @Before("po1()")
    public void beforeLog(){
        System.out.println("beforeLog....前置通知");
    }
    //后置通知
    @AfterReturning("po1()")
    public void afterReturningLog(){
        System.out.println("afterReturningLog....后置通知");
    }
    //异常通知
    @AfterThrowing("po1()")
    public void afterThrowingLog(){
        System.out.println("afterThrowingLog....异常通知");
    }
    //最终通知
    @After("po1()")
    public void afterLog(){
        System.out.println("afterLog....最终通知");
    }
	//环绕通知
//    @Around("po1()")
    public Object AroundLog(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try{
            System.out.println("AroundLog....前置通知");

            Object[] args = pjp.getArgs();
            rtValue = pjp.proceed(args);

            System.out.println("AroundLog....后置通知");
            return rtValue;
        }catch (Throwable t){

            System.out.println("AroundLog....异常通知");
        }finally {

            System.out.println("AroundLog....最终通知");
        }
        return  rtValue;
    }
}

~~~



####          2、需要注意的问题

aspectj基于全注解的aop，在分别实现aop的前置后置异常最终通知，他的位置会有一点点出入（后置通知/异常通知  会执行在最终通知之后）



所以建议的是使用xml配置形式或者基于全注解的环绕通知的形式进行spring中各个层之间的事务控制