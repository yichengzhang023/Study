- [java执行顺序](#java执行顺序)
- [jvm](#jvm)
- [类加载器](#类加载器)
  - [加载顺序 (自顶向下)](#加载顺序-自顶向下)
  - [**重要** 双亲委派加载关系](#重要-双亲委派加载关系)
  - [什么时候初始化对象](#什么时候初始化对象)
  - [Linking阶段](#linking阶段)
  - [Initializing](#initializing)
  - [volatile的实现](#volatile的实现)
  - [synchronized的实现](#synchronized的实现)
- [对象的内存布局](#对象的内存布局)
  - [创建过程](#创建过程)
  - [存储布局](#存储布局)
    - [普通对象](#普通对象)
    - [数组对象](#数组对象)
  - [对象头](#对象头)
  - [定位](#定位)
  - [分配](#分配)
- [运行时数据区](#运行时数据区)
  - [Program Counter](#program-counter)
  - [Stacks](#stacks)
  - [Native Mehtod Stacks](#native-mehtod-stacks)
  - [Direct Memory](#direct-memory)
  - [Heap](#heap)
  - [Method area (即perm space 1.7前/ meta Space 1.8之后)](#method-area-即perm-space-17前-meta-space-18之后)
- [GC](#gc)
  - [判断垃圾的方法](#判断垃圾的方法)
  - [GC Algorithms](#gc-algorithms)
    - [Nark-Sweep](#nark-sweep)
    - [Copying](#copying)
    - [Mark-Compact](#mark-compact)

# java执行顺序
**javac** -> .class文件 -> ClassLoader(包含java类库 如String等) -> 解释器或者即时编译 ->执行引擎

# jvm
是一种规范 和java无关 和class文件有关
JVM -> JRE = JVM + core library -> JDK = jre + development kit

# 类加载器
class 
-> loading 
-> linking 
    verification 验证class是否规范
    preparation 静态变量赋默认值 int = 0
    resolution 常量池中的符号引用转为能引用的
-> Initializing
    static 代码块, 将静态值赋初始值 int = 8

ClassLoader
属于顶级父类 所有class文件都是由classLoader的子类加载进内存的 进入内存中时 会生成两块内容：
1. 二进制文件
2. Class对象 指向上面的二进制内容(位于metaspace中?)
Method Area 1.7之前叫Permanent Generation 1.8之后叫metaspace
## 加载顺序 (自顶向下)
* Bootstrap 加载基础类 C++书写的
* Extension 扩展类 /ext
* App 加载classpath指定内容 自己写的类文件等
* CustomClassloader 自定义的classLoader

## **重要** 双亲委派加载关系
Bootstrap 加载extesin/app 的classloader
如果自定义classloader未找到 去父加载器app 加载找 如果再没找到 去extension找 最后去bootstrap
之后bootstrap会反过来委派
![classloader_process](../static/img/classloader.png)
为什么双亲委派
为了防止对基础类的修改 (主要是安全问题)
其次可以省资源
## 什么时候初始化对象
1. new get/put static invokestatic (打印final的值不需要加载)
2. 反射
3. 子类初始化 父类必须初始化
4. jvm启动的时候 执行主类
5. MethodHandle

## Linking阶段
* verification 
  判断class是否符合规范
* preparation
  静态成员变量赋默认值 (int = 0...)
* Resolution (解析)
  ...
## Initializing
调用初始化代码

## volatile的实现
jvm层面 -> 内存读写都加屏障 保证所有数据对所有处理器可见
storestoreBarrier
写操作
storeloadBarrier

LoadLoadBarrier
读操作
storeloadBarrier

## synchronized的实现
monitorenter
代码块
monitorexit

有异常会monitorexit退出
jvm层面
操作系统提供的同步机制
硬件层面
lock 指令
# 对象的内存布局
## 创建过程
1. classloading
2. class linking
3. class initializing
4. 申请内存
5. 成员变量复制
6. 调用构造方法
## 存储布局
### 普通对象
1. 对象头 markword 8 bytes
2. Class指针 4/8 bytes 不压缩为8字节
   1. UserCompressClassPointers压缩自己的指针
3. 实例数据 成员变量
   1. UseCompressOops 开启时 引用类型的变量只占4bytes
4. Paddding 对齐 8的倍数
### 数组对象
1. 对象头 markword 8 bytes
2. Class指针 4/8 bytes 不压缩为8字节
3. 数组长度 4 bytes
4. 数组数据
5. Paddding 对齐 8的倍数

## 对象头
主要两部分 锁的指针 和gc分代年龄
GC默认年龄为15 是因为对象头上的分代年龄只有4bytes
当一个对象使用了自带的identityHashCOde计算了hashcode时 不能进入偏向锁模式(因为存放偏向锁的位置被占据了) 重量级可以存
## 定位
1. 句柄池
a变量指向句柄池 句柄池指向实例数据 gc的时候效率较高
2. 直接指针 
a变量直接指向堆中实例数据对象保存方法区类型数据指针，直接的访问实例数据 (gc效率比较低)
## 分配
![object_allocate](../static/img/object_allocate.png)
stack -> 分配的下 分配到栈中 并且直接pop
如果对象比较大 会进入堆内存中 (老年代)
如果对象不大 会进入线程本地分配（TLAB） 分配的下进入线程本地内存 分配不下进入eden区
进入gc过程 年龄到了进入老年代 没到继续gc
# 运行时数据区
## Program Counter 
->下一条指令存放位置
## Stacks 
-> 每一个线程独有一个栈 每一个方法对应一个栈帧 装在Stack中(frame)
* 局部变量 
  用局部变量表来存
* Operand Stack  
* 动态连接 
* 返回地址
  返回下一个栈帧的地址
  invokeSpecial invokeVirtual invokeDynamic(匿名表达式) invokeInterface
new 对象 为半初始化 并且复制一份在栈帧中，之后弹出复制并且计算 调用init方法之后才会初始化
## Native Mehtod Stacks 
-> 等同于本地方法的栈
## Direct Memory 
直接内存 nio相关 (访问内核空间的内存)
以前的访问需要java复制到jvm中的内存空间 用户才能访问 现在用户可以通过直接内存区访问 (Zero-Copy)
## Heap
线程间共享
## Method area (即perm space 1.7前/ meta Space 1.8之后)
包含常量池和class 线程共享 1.8之前fgc不回收

# GC
## 判断垃圾的方法
1. 引用计数(RC)
   无法解决循环引用
2. 可达性分析
   从root对象开始搜索 stack变量 static变量 常量池 JNI指针 程序启动的时候马上需要的对象称为根对象

## GC Algorithms
### Nark-Sweep
### Copying
### Mark-Compact