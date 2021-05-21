- [Spring IOC](#spring-ioc)
  - [bean的注册流程](#bean的注册流程)
  - [创建bean](#创建bean)
  - [生命周期](#生命周期)
  - [Environment](#environment)
  - [如何从配置文件加载bean](#如何从配置文件加载bean)
  - [Springboot 启动流程](#springboot-启动流程)
# Spring IOC
## bean的注册流程
xml / 注解 / yml / properties 抽象为beandefination -> 通过扩展 完成功能的扩展(BeanFacotryProcesser) -> 通过反射实例化 -> 生成可用的bean

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