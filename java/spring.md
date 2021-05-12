- [Spring IOC](#spring-ioc)
  - [bean的注册流程](#bean的注册流程)
  - [创建bean](#创建bean)
  - [生命周期](#生命周期)
  - [Environment](#environment)
  - [如何从配置文件加载bean](#如何从配置文件加载bean)
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