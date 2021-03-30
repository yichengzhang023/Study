- [基础概念](#基础概念)
- [总体架构](#总体架构)

## 基础概念
1. 用户在配置中心进行修改并发布
2. 配置中心通知使用apollo的客服端更新配置 重新注入
3. Apollo客服端从配置中心拉取最新的配置 更新本地配置并通知到应用

## 总体架构
架构图:
![Apollo架构](../static/img/Apollo-architecture.png)
其中 client通过一个长连接和config service建立联系 并且首次启动的时候 client会去注册中心获取config service的信息。
另一方面 potral为前端页面的接口 通过Admin service去管理配置(添加/更新) 以及管理相应的用户权限
* Config Service提供配置的读取、推送等功能，服务对象是Apollo客户端
* Admin Service提供配置的修改、发布等功能，服务对象是Apollo Portal（管理界面）
* Config Service和Admin Service都是多实例、无状态部署，所以需要将自己注册到Eureka中并保持心跳
* 在Eureka之上我们架了一层Meta Server用于封装Eureka的服务发现接口
* Client通过域名访问Meta Server获取Config Service服务列表（IP+Port），而后直接通过IP
+Port访问服务，同时在Client侧会做load balance、错误重试
* Portal通过域名访问Meta Server获取Admin Service服务列表（IP+Port），而后直接通过IP+Port访问服务，同时在Portal侧会做load balance、错误重试
* 为了简化部署，我们实际上会把Config Service、Eureka和Meta Server三个逻辑角色部署在同一个JVM进程中.