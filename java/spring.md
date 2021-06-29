- [Spring IOC](#spring-ioc)
  - [bean的注册流程](#bean的注册流程)
  - [创建bean](#创建bean)
  - [生命周期](#生命周期)
  - [Environment](#environment)
  - [如何从配置文件加载bean](#如何从配置文件加载bean)
  - [Springboot 启动流程](#springboot-启动流程)
  - [Spring 事务机制](#spring-事务机制)
  - [Spring事务失效](#spring事务失效)
  - [事务的传播机制](#事务的传播机制)
    - [PROPAGATION_REQUIRED](#propagation_required)
    - [PROPAGATION_REQUES_NEW](#propagation_reques_new)
    - [PROPAGATION_NOT_SUPPORT](#propagation_not_support)
  - [谈谈自己对于Spring IOC和AOP的理解](#谈谈自己对于spring-ioc和aop的理解)
    - [IOC](#ioc)


# Spring IOC

## bean的注册流程

xml / 注解 / yml / properties 抽象为beanDefinition -> 通过扩展 完成功能的扩展(BeanFactoryProcessor) -> 通过反射实例化 -> 生成可用的bean

## 创建bean

- 实例化
  在堆中开辟一片空间 属性都是默认值 
- 初始化
  赋值 (填充属性) populateBean
  执行初始化方法

## 生命周期

实例化之后-> 给属性赋值 -> 执行aware接口的方法 (自定义的一些设置上下文/beanname之类的方法) -> before ｜ init ｜ after (归属于BeanPostProcessor) 扩展bean方法 ->销毁

## Environment

环境对象 StandardEnvironment -> 对象创建前已经定义了标准的环境对象

## 如何从配置文件加载bean

*refresh方法*
refreshBeanFactory
BeanDefinition核心属性：
BeanDefinitionMap/BeanDefinitionNames

## Springboot 启动流程

启动当前应用程序的main方法 
-> Springapplication.run() 
-> new SpringApplication()
  1. 配置resource loader
  2. 判断当前应用程序的类型(servlet/reactive/none)
  3. 获取初始化器的实例对象
  4. 获取监听器的实例对象
  5. 获取主类 开始执行
-> run方法启动
  1. 设置启动时间(计时)
  2. 设置应用上下文
  3. 设置异常报告器
  4. 设置(java.awt.headless 提供给无输入的平台使用)
基本的配置都位于spring.factories文件中 

## Spring 事务机制

1. 底层基于数据库事务和AOP机制
2. 对于加了@Transactional注解的Bean 会创建一个代理Bean
3. 修改数据库连接中的autocommit为false
4. 然后执行sql 并判断是否回滚或者提交
5. 事务传播机制是基于数据库连接来做的 如果事务中包含另外一个事务 那么会先建立一个数据库连接 再执行sql

## Spring事务失效

被代理对象不会生效
private方法会失效 因为底层是基于cglib来实现的 子类不能重载父类的private的方法

## 事务的传播机制

事务的传播性一般用在事务嵌套的场景，比如一个事务方法里面调用了另外一个事务方法，那么两个方法是各自作为独立的方法提交还是内层的事务合并到外层的事务一起提交，这就是需要事务传播机制的配置来确定怎么样执行。

### PROPAGATION_REQUIRED

Spring默认的传播机制，能满足绝大部分业务需求，如果外层有事务，则当前事务加入到外层事务，一块提交，一块回滚。如果外层没有事务，新建一个事务执行

### PROPAGATION_REQUES_NEW

该事务传播机制是每次都会新开启一个事务，同时把外层事务挂起，当当前事务执行完毕，恢复上层事务的执行。如果外层没有事务，执行当前新开启的事务即可

### PROPAGATION_NOT_SUPPORT

该传播机制不支持事务，如果外层存在事务则挂起，执行完当前代码，则恢复外层事务，无论是否异常都不会回滚当前的代码

## 谈谈自己对于Spring IOC和AOP的理解

### IOC

IOC（Inversion Of Control，控制反转）是一种设计思想，就是将原本在程序中手动创建对象的控制权，交由给Spring框架来管理。IOC在其他语言中也有应用，并非Spring特有。IOC容器是Spring用来实现IOC的载体，IOC容器实际上就是一个Map(key, value)，Map中存放的是各种对象。

将对象之间的相互依赖关系交给IOC容器来管理，并由IOC容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。在实际项目中一个Service类可能由几百甚至上千个类作为它的底层，假如我们需要实例化这个Service，可能要每次都搞清楚这个Service所有底层类的构造函数，这可能会把人逼疯。如果利用IOC的话，你只需要配置好，然后在需要的地方引用就行了，大大增加了项目的可维护性且降低了开发难度。

Spring时代我们一般通过XML文件来配置Bean，后来开发人员觉得用XML文件来配置不太好，于是Spring Boot注解配置就慢慢开始流行起来。