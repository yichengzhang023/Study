- [Spring IOC](#spring-ioc)
  - [bean的注册流程](#bean的注册流程)
  - [创建bean](#创建bean)
  - [生命周期](#生命周期)
  - [Environment](#environment)
  - [如何从配置文件加载bean](#如何从配置文件加载bean)
  - [Springboot 启动流程](#springboot-启动流程)
  - [Spring 事务机制](#spring-事务机制)
  - [Spring事务失效](#spring事务失效)


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
