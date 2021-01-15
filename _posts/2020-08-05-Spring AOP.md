# Spring AOP

## 什么是AOP

#### 了解AOP

AOP的全程为Aspect Oriented Programming，中文译为**面向切面编程**。面向切面编程是一种编程思想，是面向对象编程的一种补充。在下面的图中，原本的OOP编程是纵向编程，当多个模块之间要是实现某段同样的代码，最愚蠢的办法就是在每个模块都编写一段代码，在未切面时，一般采用的方法时将这段代码封装为方法，在各模块合适的地方进行调用，需求修改时只需修改封装的方法。但是当这一需求不再需要时或增加了新的需求，那么我们就必须再到每个模块进行修改，这样就增大了维护的工作量。于是AOP出现了。

AOP利用一种“**横切**”的思想，将封装的对象内部进行切分，切开的部分会穿插切面根据多个类的需求封装的可重用模块Aspect，并根据切面的配置情况，决定代码执行的先后顺序。（配置情况稍后讲解）至于原来的对象，其全然不知自己已经被切分，简言之，原本的对象只关注自己要实现的逻辑，至于切分的任务将将给Aspect完成。

![aop](D:\1work\学习笔记\spring\img\aop.png)

#### AOP中的基本概念

**Advice**：通知。AOP中特定的切入点上的增强处理，定义了切面将如何增强处理以及在何时处理。分为下列几种通知类型：

​		**前置通知**：在目标方法执行之前执行

​		**后置通知**：在目标方法执行之后执行（与异常通知互斥）

​		**异常通知**：在目标方法发生异常时执行（与后置通知互斥）

​		**最终通知**：无论目标方法中是否发生异常，都会执行（在后置通知或异常通知之后）

​		**环绕通知**：需要手动的执行目标方法，在环绕通知中，目标方法执行前的代码可理解为前置通知，目标方法之后的理解为后置通知，在catch块中的为异常通知，在finally块的为最终通知。

**Aspect**：切面。通常是一个封装好的类，是通知和切入点的结合。

**JointPoint**：连接点。定义了能够插入切面的一个点，即切面位置，这个点可以是方法的调用、异常的抛出。在 Spring AOP 中，连接点总是方法的调用。

**Pointcut**：切入点。带有通知的连接点，在程序中表现为切入点表达式。



## 基于XML的AOP配置

#### 引入依赖

pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.3.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.8.7</version>
    </dependency>
</dependencies>
```

#### 定义切面类

AOPAspect.java

```java
public class AOPAspect {
    /**
     * 前置通知
     */
    public void before(){
        System.out.println("AOPAspect前置通知");
    }
    /**
     * 后置通知
     */
    public void afterReturning(){
        System.out.println("AOPAspect后置通知");
    }
    /**
     * 异常通知
     */
    public void afterThrowing(){
        System.out.println("AOPAspect异常通知");
    }
    /**
     * 最终通知
     */
    public void after(){
        System.out.println("AOPAspect最终通知");
    }
    
    public Object around(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try {
            Object[] args = pjp.getArgs();  //得到方法执行所需的参数
            //写在proceed方法之前表示前置通知
            System.out.println("AOPAspect环绕通知 前置");
            rtValue = pjp.proceed(args);  //明确调用业务层方法（切入点方法）
            //写在proceed方法之后表示后置通知
            System.out.println("AOPAspect环绕通知 后置");
            return rtValue;
        } catch (Throwable throwable) {
            //写在catch中表示异常通知
            System.out.println("AOPAspect环绕通知 异常");
            throw new RuntimeException(throwable);
        }finally {
            //写在finally中表示最终通知
            System.out.println("AOPAspect环绕通知 最终");
        }
    }
}
```

#### 创建业务类

AccountService.java

```java
public class AccountService {
    public void saveAccount() {
        System.out.println("Account被保存了");
    }

    public void updateAccount() {
        System.out.println("Account被更新了");
    }
}
```

#### 配置xml

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置Spring的IOC，将Service对象配置进来-->
    <bean id="accountService" class="cn.yz.service.AccountService"></bean>

    <!--配置Logger类-->
    <bean id="AOPAspect" class="cn.yz.utils.AOPAspect"></bean>

    <!--配置AOP-->
    <aop:config>
        <aop:pointcut id="ptl" expression="execution(* cn.yz.service.*.*(..))"/>
        <!--配置切面-->
        <aop:aspect id="aspectAdvice" ref="AOPAspect">
            <!--配置前置通知，切入点方法执行之前执行-->
            <aop:before method="before" pointcut-ref="ptl"></aop:before>
            <!--配置后置通知，切入点方法执行之后执行，后置通知和异常通知只能执行一个-->
            <aop:after-returning method="afterReturning" pointcut-ref="ptl"></aop:after-returning>
            <!--配置异常通知，切入点方法产生异常后执行-->
            <aop:after-throwing method="afterThrowing" pointcut-ref="ptl"></aop:after-throwing>
            <!--配置最终通知，无论切入点方法是否产生异常，都会执行-->
            <aop:after method="after" pointcut-ref="ptl"></aop:after>

            <!--配置切入点表达式 id属性用于指定表达式的唯一标识，expression属性用于指定表达式内容
                此标签写在aop:aspect标签内部，只能当前切面使用
                它还可以写在aop:aspect标签外面，表示所有切面可用，但只能放在aop:aspect之前-->
        </aop:aspect>
    </aop:config>
</beans>
```

其中标签**aop:config**表明开始AOP的配置，标签**aop:aspect**表示开始切面的配置，在aop:aspect标签内部，**aop:before**、**aop:after-returning**、**aop:after-throwing**、**aop:after**、**aop:around**分别代表配置前置，后置，异常，最终，环绕通知，在这些标签内部，**method**表示用切面类的哪个方法作为相应的通知方法，**pointcut-ref**为切入点表达式。

切入点表达式可直接编写，或者以**aop:pointcut**标签来定义，aop:pointcut内部**id**为唯一标识，**expression**为切入点表达式，表达式写法为”访问修饰符 返回值 全限定类名.方法名(参数列表)“，访问修饰符可以省略，返回值可以使用通配符\*表示任意类型，包名也可使用通配符，有几级包名就要写几个\*,也可以使用..表示当前包和子包，参数个数也类似（具体规则可自行百度），全通配写法为：\* *..*.\*(..)

#### 测试

```java
public class AOPTest {
    public static void main(String[] args) {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.获取对象
        IAccoutService as = (IAccoutService) ac.getBean("accountService");
        //3.执行方法
        as.saveAccount();
        as.updateAccount();
    }
}
```

#### 执行结果

![执行结果](D:\1work\学习笔记\spring\img\aopxmlresult.jpg)

## 基于注解的AOP配置

#### 相关注解:

**@Aspec**t: 定义当前类为一个切面类，相当于xml中的**aop:aspect**标签

**@Pointcut**: 切入点表达式，相当于xml中的**aop:pointcut**标签，@Pointcut("")引号内定义表达式，例如@Pointcut("execution(* cn.yz.service.impl.*.*(..))")

**@Before**: 前置通知，参数为切入点表达式，如@Before("pt1()")，其他通知类似

**@AfterReturning**: 后置通知

**@AfterThrowing**: 异常通知

**@After**: 最终通知

**@Around**: 环绕通知

因此切面类可改写为：

```java
@Component("AOPAspect")
@Aspect 
public class AOPAspect {

    @Pointcut("execution(* cn.yz.service.*.*(..))")
    private void pt1(){}

    /**
     * 前置通知
     */
    //@Before("pt1()")
    public void before(){
        System.out.println("AOPAspect前置通知");
    }
    /**
     * 后置通知
     */
    //@AfterReturning("pt1()")
    public void afterReturning(){
        System.out.println("AOPAspect后置通知");
    }
    /**
     * 异常通知
     */
    //@AfterThrowing("pt1()")
    public void afterThrowing(){
        System.out.println("AOPAspect异常通知");
    }/**
     * 最终通知
     */
    //@After("pt1()")
    public void after(){
        System.out.println("AOPAspect最终通知");
    }

    @Around("pt1()")
    public Object around(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try {
            Object[] args = pjp.getArgs();  //得到方法执行所需的参数
            //写在proceed方法之前表示前置通知
            System.out.println("AOPAspect环绕通知 前置");
            rtValue = pjp.proceed(args);  //明确调用业务层方法（切入点方法）
            //写在proceed方法之后表示后置通知
            System.out.println("AOPAspect环绕通知 后置");
            return rtValue;
        } catch (Throwable throwable) {
            //写在catch中表示异常通知
            System.out.println("AOPAspect环绕通知 异常");
            throw new RuntimeException(throwable);
        }finally {
            //写在finally中表示最终通知
            System.out.println("AOPAspect环绕通知 最终");
        }
    }
}
```

#### 配置xml

开启spring对注解aop的支持

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置Spring创建容器时要扫描的包-->
    <context:component-scan base-package="cn.yz"></context:component-scan>

    <!--配置spring开启注解AOP的支持-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

</beans>
```

AccountService.java

```java
@Service("accountService")
public class AccountService {
    public void saveAccount() {
        System.out.println("account被保存了");
    }

    public void updateAccount(int i) {
        System.out.println("account被更新了");
    }

}
```

#### 测试

```java
public class AOPTest {
    public static void main(String[] args) {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.获取对象
        IAccoutService as = (IAccoutService) ac.getBean("accountService");
        //3.执行方法
        as.saveAccount();
        as.updateAccount();
    }
}
```

#### 执行结果

![执行结果](D:\1work\学习笔记\spring\img\aopannoresult.jpg)