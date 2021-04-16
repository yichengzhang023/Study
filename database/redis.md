- [几种 I/O 模型](#几种-io-模型)
  - [文件描述符(File Descriptor FD)](#文件描述符file-descriptor-fd)
  - [阻塞IO](#阻塞io)
  - [非阻塞IO](#非阻塞io)
  - [多路复用IO](#多路复用io)
    - [select](#select)
    - [poll](#poll)
    - [epoll](#epoll)

## 几种 I/O 模型
### 文件描述符(File Descriptor FD)
> 表示指向文件引用的抽象化概念
> 每个Unix进程均有三个标准的文件描述符 即**stdin** **stdout** **stderr**
### 阻塞IO
当使用 read 或者 write 对某一个文件描述符进行读写时，如果当前的FD不可读或者不可写 则服务会阻塞至可以读写。这种模型下 每一个连接会创建一个线程处理 直到连接断开。
例子：
应用发送读请求 -> 请求系统内核 -> 等待数据 -> 阻塞直到数据准备好 -> 返回数据 -> 应用处理

### 非阻塞IO
类似阻塞模型 如果数据没有准备好则kernel会返回一个bwouldblock错误 应用会轮询直到数据准备好。

### 多路复用IO
java中的NIO使用的就是多路复用机制。不同操作系统多路复用采用的模型不同 比如windows系统采用select linux采用epoll。select的函数调用可以同时监控多个FD的可读可写情况 当某些FD变为可读可写时 select方法会返回FD的个数。
#### select
select 监视三种fd 分别是read write 和except 调用select会阻塞到有数据 或者超时。返回后可以通过遍历fdset找到就绪的描述符。有最大监视限制(Linux 上为1024个)

#### poll
poll将三个fdset视为一个pollfd 管理监视的event和发生的event 类似select 轮询pollfd来获取就绪的描述符
#### epoll
主要是三个接口
```c
int epoll_create(int size)
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *events)
int epoll_wait(int epfd, epoll_event *events, int maxevents, int timeout)
```
create 方法创建一个size为传入参数的fd限制最大的fd值

ctl方法对指定的fd执行op操作 告诉内核需要监听的事件
```c
struct epoll_event {
    /*
    epoll events
    EPOLLIN 可读
    EPOLLOUT 可写
    EPOLLPRI  优先读
    EPOLLERR 错误
    EPOLLHUP 中断
    EPOLLET 工作模式(Edge Trigger Level Trigger)
    EPOLLONESHOT 只监听一次事件
    */
    __uint32_t events;
    epoll_data_t data;
}
```
wait方法即等待时间的产生 用于返回
工作模式：
ET edge trigger
即高速模式 即read只会读取buffer一次 如果有新数据in不会再次读
LT level trigger
默认的模式 只要fd还有数据可读，每次wait方法都会返回fd的事件

在 select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()来注册一 个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait() 时便得到通知。(此处去掉了遍历文件描述符，而是通过监听回调的的机制。这正是epoll的魅力所在。)

