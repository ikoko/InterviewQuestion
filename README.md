[TOC]

## 项目

### APM 

- 知识点：[多线程](#多线程)、[Runloop](#Runloop)、[网络](#网络)、启动流程、VC生命周期
- 项目难点：卡顿函数定位
- 整体方案：一个单例类控制 APM 开关，各个监控模块可单独关闭，服务器下发监控开关。

- [卡顿监控](https://cloud.tencent.com/developer/article/1427933)

  - **卡顿：**出现界面不响应或者界面渲染粘滞的情况。而应用界面的渲染及事件响应是在主线程完成的，出现卡顿的原因可以归结为主线程阻塞。一般卡顿产生的原因为大量的 IO 操作、大量的计算、复杂的布局、主线程在等锁。
  - **卡顿判定**：`RunLoop` 的调用主要在 `BeforeSources` 和 `AfterWaiting` 两个状态。因此，若是发现这个两个时间内耗时过长，就可以判定此时主线程出现卡顿。 

  - **监控方案**：创建一个 `Observer` 添加到主线程 `runloop` ，开启一个子线程，在子线程中[开启一个持续的循环](https://www.jianshu.com/p/d0aab0eb8ce4)来监控主线程 `runloop` 的状态。如果发现主线程 `runloop` 的状态卡在为 `BeforeSources` 或者 `AfterWaiting` 超过设定的阈值时，说明主线程当前卡顿。

  - **卡顿时长统计**
  - **卡顿函数定位**

  - **卡顿的堆栈怎么符号化？**

    

- 网络监控

  网络监控指标

- 启动时长

  启动时长定义

- 页面渲染时长

  页面渲染时长定义

### 夜间模式

- 整体方案

  通过一个标记位来表示当前模式，

- UI 组件
- 本地图片
- 网络图片
- 热切换
- 性能



### 盲人模式

- 整体方案

- 本地图片



### iPad 分配适配

- 整体方案



### Crash 防护

- 系统 API 加固
- 系统 API 版本检测
- 野指针防护
- unrecognized selector
- KVO 防护
- Timer 防护



### JS 交互

- 整体方案
- 接口定义



### 实时涂鸦

- 整体方案
- 数据处理
- 界面绘制





## 内存



### [**内存分区**](evernote:///view/3106006/s17/76b195ad-9c1a-4b8d-82ca-9d1ad8a5a825/76b195ad-9c1a-4b8d-82ca-9d1ad8a5a825/)

​		iOS的内存分为五个区，代码区、常量区、全局区、堆、栈。常说的内存管理主要是说管理堆上的内存，栈内存是不需要我们管理的，CPU直接管理入栈出栈，比如说每次函数调用的时候都会有入栈操作，如果函数递归调用栈空间一直不释放就会栈溢出。

### **[引用计数](https://juejin.cn/post/6844903847622606861#heading-25)**

​		堆上的内存是使用引用计数来管理的，引用计数为0时会调用dealloc释放对象的内存空间，不过现在都是 `ARC`，基本不需要我们手动 `retain/release`了，除了`Core Foundation` 框架。

​       一般引用计数直接存储在[优化过的 `isa` 指针](https://juejin.cn/post/6844903898205913096)的 `extra_rc` 字段中（19位的内存存储引用计数减一），当引用计数值过大 这个字段存不下的时候，就会找到 `Runtime` 维护的全局哈希表（`SideTables`），`SideTables `里边存储的都是 `SideTable` 结构体，结构体里存着引用计数表（`refcnts`）和弱引用计数表（`weak_table`）

```c++
struct SideTable {
    spinlock_t slock;	// 保证原子操作的自旋锁
    RefcountMap refcnts;	// 引用计数 hash 表
    weak_table_t weak_table;	// weak 引用计数 hash 表
}
```

### **[循环引用](https://www.jianshu.com/p/ddfd1b3c0298)**

​		两个对象互相强引用彼此，彼此的引用计数始终都是 `1 `，导致无法释放。比如 `delegate`、`Block`、`Timer`。

### **[weak](evernote:///view/3106006/s17/42dc51d2-7c96-4193-80a6-c245f194cf32/42dc51d2-7c96-4193-80a6-c245f194cf32/)**

​		为了避免循环引用，可以使用 `weak` 修饰变量，`weak` 就是弱引用，弱引用是单独计数表，不会增加引用计数，当对象被释放的时候，所有指向它的弱引用都会自动被置空。根据对象内存地址（Key）获取所有` weak` 指针地址（Value）的数组，然后遍历数组设为 `nil`，最后从 `weak` 表中移除。

​		 **释放过程**：调用 `dealloc` 后会判断一些对象的标记，如 `TaggedPointer` 对象不需要释放，有没有优化过 `isa`（`NONPOINTER_ISA`），有无关联对象、有无析构函数、有无弱引用、有没有用到 `SideTable`，没有的话会释放更快。

​		**TaggedPointer**（标记指针）：为了节省内存占用，苹果提出了 `Tagged Pointer`，用来存储小的对象（`NSNumber`、`NSDate`、`NSString`，`NSIndexPath`），`Tagged Pointer` 分成两部分，一部分直接保存数据，一部分作为特殊标记，直接存储在栈中，访问速度更快。（`Tagged Pointer` 没有 `isa`，通过标记位判断）



### **总结**

​		`ARC` 在编译期自动插入管理引用计数的代码，`TaggedPointer` 直接存在栈上，`extra_rc` 管理引用计数，思路都是减轻开发负担，减少堆内存开销提高性能，让开发者不必过度关心内存管理。



-  **`ARC` 和 `MRC` 的区别**

  



- **一个 NSObject 对象占用多少字节**

​		 `iOS` 在分配对象内存时, 是以 `16` 倍数来分配的，即一个空 `NSObject` 对象占用 `16` 个字节，这些在 `Runtime` 源码可以看到。一个空 `NSObject` 对象实际使用了 `8` 个字节的空间(`64bit`环境下)，因为空对象仅包含` isa` 指针( ` 8 `个字节)

​		实际使用空间通过 ` class_getInstanceSize` 函数获得，系统分配空间使用 `malloc_size` 函数获得



- **一个自定义对象占用多少字节**

​		一个自定义对象占用了几个字节, 要看有多少成员变量, 同时还要计算上 `NSObject` 的 `isa` 指针大小, 同时为了内存对齐, 必须是 `16` 的倍数



- **为什么要内存对齐**

​		很多处理器拒绝读取未对齐数据，因为未对齐的数据，会大大降低 `CPU` 的性能。当一个程序要求这些` CPU` 读取未对齐数据时，这时` CPU` 会进入异常处理状态并且通知程序不能继续执行。可以举个搬砖夹子的例子。



- **什么是内存对齐**

​      堆上的内存是使用链表管理动态分配的，在`x86`或 `ARM` 处理器中，基本 `C` 数据类型通常并不存储于内存中的随机字节地址。实际情况是，除 `char` 外，所有其他类型都有“对齐要求”：`char` 可起始于任意字节地址，`2` 字节的 `short` 必须从偶数字节地址开始，`4` 字节的 `int` 或 `float` 必须从能被 `4` 整除的地址开始，`8` 比特的 `long` 和 `double` 必须从能被 `8` 整除的地址开始，即：**起始的字节地址需能被自身整除**。提高访问速度。为对齐而填充的字节空间会被浪费。



## Block

### Block 的本质

- `block` 是对 `C` 语言的扩充功能，就是匿名函数，在 `OC` 中 `block` 有 `isa` 指针，可以看做是一个 `OC` 对象。

- `block` 封装了函数调用以及函数调用环境



### Block 的类型

| 类型              | 副本的配置存储域 | 复制效果     |
| ----------------- | ---------------- | :----------- |
| **NSGlobalBlock** | 程序的数据区域   | 什么也不做   |
| **NSStackBlock**  | 栈               | 从栈复制到堆 |
| **NSMallocBlock** | 堆               | 引用计数增加 |

> **截获了自动变量的 Block 是 NSStackBlock 类型，没有截获自动变量的 Block 则是 NSGlobalBlock 类型, NSStackBlock 类型的 Block 进行 copy 操作之后其类型变成了 NSMallocBlock 类型。**



### Block 截获变量

| 变量类型          | 是否捕获到block内部 | 访问方式 |
| ----------------- | ------------------- | -------- |
| 局部变量 `auto`   | 是                  | 值传递   |
| 局部变量 `static` | 是                  | 指针传递 |
| 全局变量          | 否                  | 直接访问 |

> **自动变量在作用域结束时会被系统回收， block 很可能是在超出自动变量作用域的时候去执行，如果没有捕获自动变量，那么后面执行的时候，自动变量已经被回收了。对于 static 局部变量，它的生命周期不会因为作用域结束而结束，所以 block 只需要捕获这个变量的地址，在执行的时候通过这个地址去获取变量的值，这样可以获得变量的最新的值。而对于全局变量，在任何位置都可以直接读取变量的值。**
>
> **首先 block 变量截获只针对 block 内部使用的自动变量，不使用不截获，因为截获的自动变量会存储于block的结构体内部，会导致block体积变大。特别要注意的是默认情况下 block 只能访问，不能修改局部变量的值。**



- **`block` 里访问 ` self` 是否会捕获？**

  `self` 相当于 `this` 指针， 会当做调用 `block` 函数的参数，参数是局部变量，`block` 捕获了 `self` 。

  

- **`block` 里访问成员变量 ` self.xx` 是否会捕获？**

  会，先捕获 `self` ，使用 `self` 访问成员变量。

  

- **ARC 下，什么情况会自动将 `Block` ` copy` 到堆上？**

  1. 强引用  `block` 
  2. 将 `block` 作为函数返回值
  3. `GCD` 的 `block` 默认会做 `copy` 操作
  4. `block` 作为 Cocoa API 中方法名含有 `usingBlock` 的参数时

  

- **ARC 下，会自动将 `Block` ` copy` 到堆上的过程？**

  不知道



- **_ _strong  _ _typeof(self) strongSelf = weakSelf;  [为什么不会引起循环引用](http://ziecho.com/post/ios/2015-09-02) ？**

  指针连带关系 `self`  的引用计数还会增加，但是这个是在 `block` 里面，生命周期也只在当前 `block` 的作用域。所以,当这个 `block`结束, `strongSelf` 随之也就被释放了，也不会影响 `block` 外部的 `self` 的生命周期。



- **说说 Block** 

  先说说 `Block` 的本质是什么，有几种类型，每种类型存储在哪，`copy` 后怎么操作，再说说 `Block` 的变量捕获和 `__block`。

  

- 扩展问题：https://www.jianshu.com/p/4e79e9a0dd82



### __block 修饰符



- **`__block` int age = 10，系统做了哪些？**

  编译器会将 `__block` 变量包装成一个对象，有 `isa` 指针，`__forwarding` 指向变量的地址。



## Runtime

### isa 指针

```c
union isa_t  // isa 指针结构
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
    struct {
        uintptr_t nonpointer        : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 33; // MACH_VM_MAX_ADDRESS 0x1000000000
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 19;
    };
}
```

​		[iOS Runtime（重要](https://juejin.cn/post/6844903898205913096)），[isa 指针（重要](https://www.dazhuanlan.com/2019/12/05/5de7f61cd178f/)）

​		在 `arm64` 架构之前，`isa` 就是一个普通的指针，存储着`Class、Meta-Class`对象的内存地址，从 `arm64` 架构开始，对 `isa` 进行了优化，变成了一个共用体 `union ` 结构，使用位域来存储更多的信息。

​       **TaggedPointer**（标记指针）：为了节省内存占用，苹果提出了 Tagged Pointer，用来存储小的对象（`NSNumber`、`NSDate`、`NSString`，`NSIndexPath`），Tagged Pointer 分成两部分，一部分直接保存数据，一部分作为特殊标记，直接存储在栈中，访问速度更快。（Tagged Pointer 没有 `isa`，通过标记位判断）

​       **NONPOINTER_ISA**（指针优化）：`runtime` 会用 `isa` 的其他额外的位来存储对象的其他数据，例如引用计数，是否被弱引用等，避免额外的数据结构存储每个对象的引用计数和弱引用情况，也可以减少对象 ` -retain`，`-release`，`-alloc`，`-dealloc` 时通过查表修改和获取引用计数的时间.

未优化的 `isa` ，使用 `sidetable_retain() `管理引用计数，查表会增加耗时；已优化的 `isa` ， 这其中又分 `extra_rc` **溢出和未溢出**的两种情况。

- **未溢出时** `extra_rc + 1` 完事。
- **溢出时**，将 `extra_rc` 中一半值转移至 `sidetable` 中，然后将 `isa.has_sidetable_rc` 设置为` true`，表示使用了 `sidetable` 来管理引用计数。

​       总体来说，苹果将内存使用到了极致，小变量尽量避免使用堆空间，尽量不使用全局哈希表存储引用计数，使开发者不必过度关心内存的效率问题。



### 对象、类对象、元类对象



### 消息传递



### 消息转发



### Method Swizzling



## Runloop

### 事件循环



### 用户态



### 核心态



### 常驻线程



## 多线程

### GCD



### NSOperation



### 资源共享



### 线程同步



### 自旋锁



## UI 视图

### 绘制原理



### 异步绘制



### 事件传递机制



### 离屏渲染



### 界面性能优化



### 图像处理



### 动画



## 网络

### HTTP



### HTTPS



### TCP



### Socket



### DNS 解析



## 性能优化

### TableView 优化

### 启动速度优化

### 编译速度优化

### 应用包瘦身



## 设计模式



## 项目框架

### MVC



### MVVM



### MVP



### 组件化方案

- 路由



### CocoaPods





## 第三方库

### AFNetworking



### SDWebImage



## Swift

### RxSwift



## 常见问题

### 内存和虚拟内存如何映射

### 图片的内存如何计算

### 热修复原理

### 无痕埋点

### APNS 推送机制

### APP 签名机制

### Charles原理

### 线上闪退追踪解决

### Crash捕获流程

- 常见的 Crash 类型
- Crash 捕获流程



### 设计一个网络框架



### 实现Json转Model库



### 实现Masonry类似的库（链式调用）



### 提升自己的方式

- RSS 阅读器订阅一些大神的博客（喵神、唐巧、戴明等）
- 微信公众号关注一些技术博客，微信美团等。
- 沉浸 Github



## 算法

### 字符串反转



### 链表反转



### 链表是否有环



### 有序数组合并



### 二叉树的前缀遍历