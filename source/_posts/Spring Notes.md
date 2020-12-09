---
title: Spring学习笔记
---

# spring framework支持几个模块
  * 支持IOC和AOP的容器
  * 支持JDBC和ORM数据访问模块
  * 支持声明式事务的模块
  * 支持基于Servlet的MVC开发
  * 支持基于Reactive的WEB开发
  * 以及集成JMS、JavaMail、JMX、缓存等其他模块。
  
# IOC容器 
管理所有的轻量级JavaBean组件, 支持组件的声明周期管理、配置和组装, 提供AOP的支持, 以及AOP基础上的声明式事务  
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
  3. new ClassPathXmlApplicationContext("application.xml") 或者 new XmlBeanFactory(new ClassPathResource("application.xml"));  区别在于BeanFactory不主动加载, 而ApplicationContext会一次性创建所有的Bean(ApplicationContext继承自BeanFactory)
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
    
    
 
 
 
 
 
 
