---
title: Spring学习笔记  
date: 2020-12-21  
tags:
- Java
- Spring

categories:
- Spring
---

# spring framework支持几个模块
  * 支持IOC和AOP的容器
  * 支持JDBC和ORM数据访问模块
  * 支持声明式事务的模块
  * 支持基于Servlet的MVC开发
  * 支持基于Reactive的WEB开发
  * 以及集成JMS、JavaMail、JMX、缓存等其他模块。
  
# IOC容器 
管理所有的轻量级JavaBean组件, 支持组件的生命周期管理、配置和组装, 提供AOP的支持, 以及AOP基础上的声明式事务  
  * 传统new的方式有以下缺点:
    1. 实力化一个组件很难, 很多组件可能涉及到配置解析, 费时费力
    2. 没必要分别创建公用的组件, 但是不好管理
    3. 很多组件需要销毁来释放资源, 但是如果是公用组件, 不好确保它的使用方全部被销毁
    4. 如果更多的组件进来, 依赖关系会变得更复杂
    5. 测试麻烦
  * 核心问题:
    1. 创建时, 谁负责创建
    2. 谁根据依赖关系组装
    3. 销毁时, 如何根据正确的依赖关系销毁
  * 不直接new, 而是通过注入的方式的好处:
    1. 组件更关心自己的事情, 不在乎依赖的组件是怎么创建的
    2. 共享组件变得十分简单
    3. 注入的组件更自由, 测试更简单
  * **IOC使组件的创建+配置与组件的使用分离, 由IOC容器来负责组件的声明周期的管理**
  * ***无侵入式容器***
    1. 既可以在spring容器中运行 也可以自行创建对象
    2. 测试的时候可以并不依赖spring容器, 进行单独测试, 提高开发效率
  * XML装配bean
    1. Bean的类文件, 值依赖, 对象依赖
    2. 编写application.xml配置文件
       1. bean要有id作为唯一标识
       2. 对象引用要用property name ref 值引用要用property name value
    3. new ClassPathXmlApplicationContext("application.xml")或者new XmlBeanFactory(new ClassPathResource("application.xml"));区别在于BeanFactory不主动加载, 而ApplicationContext会一次性创建所有的Bean(ApplicationContext继承自BeanFactory)
  * Annotation装配bean
    1. @component注解及其子注解配置在要注入的bean上
    2. @autowired注解来自动注入依赖
    3. @configuration和@componentscan注解来配置当前文件为配置文件, 且@componentscan自动扫描当前类所在的包及其子包
  * 定制bean
    1. @Scope 可以配置单例模式和原型模式(每次都获取一个新的实例)
    2. 注入List @Autowired List<Validator> validators; @component @order(1) 来保证有序  
    3. 可选注入 @Autowired(required = false)
    4. 创建第三方Bean: 在@Configuration的类中的方法上使用@Bean注解来返回bean

      ```
      @Configuration
      @ComponentScan
      public class AppConfig {
        // 创建一个Bean:
        @Bean
        ZoneId createZoneId() {
          return ZoneId.of("Z");
        }
      }
      ```
    5. 初始化和销毁  
       1. 导入javax.annotation-api
       2. 初始化@postconstruct
           1. 构造方法构建bean实例
           2. 依赖注入
           3. 调用被标注的无参方法
       3. 销毁@predestory
           1. 销毁时 优先调用被标注的无参方法
    6. 使用别名
       1. 注册bean时 @bean("name") 或者@bean @qualifier("name")
       2. 注入bean时 @autowired @qualifier("name")
       3. 或者使用@Primary注解来表明主要bean 如果注入时没有声明则自动注入主要的bean
    7. 使用factorybean
       1. 实际调用getObject方法返回注册的对象
       2. getObjectType来确定注册的对象的类型
  * 使用Resource
  @Value("classpath:/logo.txt") private Resource resource; resource中可以获取到文件流对象

      tip：var 关键字 jdk 10 新特性 可以用在局部变量或者for循环中 预测编译后的对象类型
  * 注入配置
    1. @propertySource读取到的配置是针对IOC全局的，任何bean都可以读取已经读取到的配置文件
    2. @PropertySource("app.properties") 来读取classpath下的配置文件
    3. @Value("${app.zone:Z}")来注入属性为app.zone的值 如果不存在 则这个值为Z
    4. 也可以先把配置读取到javaBean中，然后通过#{smtpConfig.host}来调用smtpConfig这个bean的getHost方法来注入值

           这样做的好处是只需要在这个javaBean中一处修改（例如更换数据源），其他的依赖的这个javaBean的@Value 不需要修改
  * 使用条件装配
    1. @Profile注解
       1. JVM启动时加参数-Dspring.profiles.active=test,master来指定以test和master环境来启动
       2. @Bean @Profile({ "test", "master" })（同时满足test和master才自动装配该bean） 或者@Bean @Profile("!test") （不是test环境才自动装配该bean）
    2. @Conditional注解
       1. @Conditional(OnSmtpEnvCondition.class)
       2. 实现Condition接口
       ```
       public class OnSmtpEnvCondition implements Condition {
         public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            return "true".equalsIgnoreCase(System.getenv("smtp"));
         }
       }
       ```
       3. 只有Condition这个类返回true时才会加载bean
       
# AOP
为了解决权限检查，日志记录，事务管理的重复代码而设计的面向切面编程技术  
原理：基于jvm的动态代理，要是实现了接口，就可以使用jdk动态代理，要是没有可以使用CGLIB来实现
  * 装配AOP
    1. 基本概念:
       1. Aspect 切面, 即一个横跨多个核心逻辑的功能,即系统的关切点
       2. JoinPoint 连接点, 即定义在应用程序的何时, 插入切面的执行
       3. PointCut 切入点 即一组连接点的组合
       4. Advice 增强, 即在特定连接点上的执行的动作
       5. Introduction 引介, 即为一个已有的java对象动态地增加新的接口
       6. Weaving 织入, 将切面织入到应用程序的流程中
       7. Interceptor 拦截器, 一种实现增强的方式
       8. Target Object  目标对象, 即真正执行业务的核心逻辑对象
       9. AOP Proxy 代理对象, 客户端所持有的增强后的代理对象
    2. 代理流程
       1. 定义执行方法, 并在方法注解上, 使用AspectJ注解定义在何处调用此方法
       2. 在定义执行方法的类上加上@component注解和@aspect注解
       3. 在@configuration的类上加上@enableAspectJAutoProxy注解
    3. 拦截器类型：
       1. @Before 注解: 先执行拦截方法 再执行目标代码，要是拦截方法抛出异常，则目标方法不执行
       2. @After 注解：先执行目标方法 不管其是否抛出异常 拦截方法都执行
       3. @AfterReturing 与 after不同的是 只有正常返回时拦截方法才会执行
       4. @AfterThrowing 与 after不同的是 只有在抛出异常时拦截方法才会执行
       5. @Around 完全控制目标代码是否执行，并在执行前后 抛出异常前后执行任意拦截代码
  * 使用自定义注解来进行装配aop
    1. 自定义注解类
    ```
    @Target(METHOD)
    @Retention(RUNTIME)
    public @interface MetricTime {
        String value();
    }
    ```
    2. 被拦截的方法上加上注解
    ```
    @Component
    public class UserService {
        // 监控register()方法性能:
        @MetricTime("register")
        public User register(String email, String password, String name) {
            ...
        }
        ...
    }
    ```
    3. 编写执行方法
    ```
    @Aspect
    @Component
    public class MetricAspect {
        @Around("@annotation(metricTime)")
        public Object metric(ProceedingJoinPoint joinPoint, MetricTime metricTime) throws Throwable {
            String name = metricTime.value();
            long start = System.currentTimeMillis();
            try {
                return joinPoint.proceed();
            } finally {
                long t = System.currentTimeMillis() - start;
                // 写入日志或发送至JMX:
                System.err.println("[Metrics] " + name + ": " + t + "ms");
            }
        }
    }
    ```
  * **Spring通过CGLIB创建的代理类，不会初始化代理类自身继承的任何成员变量，包括final类型的成员变量！**
    1. 访问被注入的Bean时，总是调用方法而非直接访问字段；即要调用访问方法
    2. 编写bean的时候, 如果这个类可能被代理, 那么就要避免使用public final来修饰


