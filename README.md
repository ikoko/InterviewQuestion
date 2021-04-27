[TOC]

## 项目

### APM 

- **知识点**：[多线程](#多线程)、[Runloop](#Runloop)、[网络](#网络)、启动流程、VC生命周期
- **项目难点**：卡顿函数定位
- **整体方案**：一个单例类控制 APM 开关，各个监控模块可单独关闭，服务器下发监控开关。

- **[卡顿监控](https://cloud.tencent.com/developer/article/1427933)**

  - **卡顿**

    ​		出现界面不响应或者界面渲染粘滞的情况。而应用界面的渲染及事件响应是在主线程完成的，出现卡顿的原因可以归结为主线程阻塞。一般卡顿产生的原因为大量的 IO 操作、大量的计算、复杂的布局、主线程在等锁。

  - **卡顿判定**

    ​		`RunLoop` 的调用主要在 `BeforeSources` 和 `AfterWaiting` 两个状态。因此，若是发现这个两个时间内耗时过长，就可以判定此时主线程出现卡顿。 

  - **监控方案**

    ​		创建一个 `Observer` 添加到主线程 `runloop` ，开启一个子线程，在子线程中[开启一个持续的循环](https://www.jianshu.com/p/d0aab0eb8ce4)来监控主线程 `runloop` 的状态。如果发现主线程 `runloop` 的状态卡在为 `BeforeSources` 或者 `AfterWaiting` 超过设定的阈值时，说明主线程当前卡顿。

  - **堆栈获取**

    https://github.com/woshiccm/RCBacktrace 

  - **卡顿函数定位**

    维护一个VC显示栈，卡顿的时候上报栈顶VC

    将符号表和堆栈文件给到后端同一符号化

  - **卡顿的堆栈怎么符号化？**

     

    

- **网络监控**

  - **监控指标**

    ​		DNS 时间、SSL 时间、首包时间、响应时间、网络错误

  - **整体方案**

    ​		hook `NSURLSession` 相关函数，利用 `NSURLSessionTaskMetrics` 获取各部分时间戳，计算后得出相应时间。

  

- **启动时长监控**

  - **APP 启动流程**

    			1. 创建进程，加载可执行文件 Mach - O
     		2. 加载动态链接库，进行 `rebase` 指针调整和 `bind` 符号绑定
     		3.  ` Runtime` 初始化，`ObjC` 相关 `Class` 的注册、`category` 注册、`selector` 唯一性检查等
     		4. 初始化，执行 `+load` 方法、调用 `attribute((constructor))` 修饰的函数、创建 C++ 静态全局变量等
     		5. 设置线程入口，执行 `main` 函数
     		6. `didFinishLaunchingWithOptions` ，程序加载完成，设置根视图
     		7. 首页加载完成

  - **启动时长定义**

    ​		从用户点击icon，到首页加载结束，用户看到数据（冷启动首页请求使用缓存）

  - **启动时长统计**

    ​		参照美团，通过 `sysctl` 函数获得进程信息，以App的进程创建时间为开始时间（ms），以用户看到首页数据为结束时间。

  ```objective-c
  #import <sys/sysctl.h>
  #import <mach/mach.h>
  
  + (BOOL)processInfoForPID:(int)pid procInfo:(struct kinfo_proc*)procInfo {
      int cmd[4] = {CTL_KERN, KERN_PROC, KERN_PROC_PID, pid};
      size_t size = sizeof(*procInfo);
      return sysctl(cmd, sizeof(cmd)/sizeof(*cmd), procInfo, &size, NULL, 0) == 0;
  }
  
  + (NSTimeInterval)processStartTime {
  	  // 共用体结构，秒和微秒分别在__p_starttime.tv_s、__p_starttime.tv_usec，相加即是时间戳
      struct kinfo_proc kProcInfo; 
      if ([self processInfoForPID:[[NSProcessInfo processInfo] processIdentifier] procInfo:&kProcInfo]) {
          return kProcInfo.kp_proc.p_un.__p_starttime.tv_sec * 1000.0 + kProcInfo.kp_proc.p_un.__p_starttime.tv_usec / 1000.0;
      } else {
          NSAssert(NO, @"无法取得进程的信息");
          return 0;
      }
  }
  ```

  

- **页面渲染时长监控**

  - **页面渲染时长定义**

    ​		从 `loadView` 到 `viewDidAppear`

    

### 夜间模式

- 整体方案

  通过一个标记位来表示当前模式，改变时发出通知，在Tabbar里接受到通知后处理，改变tabbar颜色，遍历subVC，看有没有实现夜间模式改变的协议函数，实现了即调用，可拓展APP换肤，节日换肤。

- 文本、背景色

  通过plist文件进行映射

- UI 组件

  TableViewCell 注册两套 id，模式改变时刷新table

- 本地图片

  runtime 添加前缀获取，或区分不同Bundle

- 网络图片

  下载完成时添加滤镜

- 性能

  

### 盲人模式

- 整体方案

- 本地图片



### iPad 分配适配

- 整体方案



### Crash 防护

- 系统 API 加固

  ​	NSArray、NSDic

- 系统 API 版本检测

  ​	Clang静态检查提供了iOS低版本调用高版本API检查的功能，Build Settings页面，在“Other C Flags”和“Other C++ Flags”中增加“-Wunguarded-availablility”。

- unrecognized selector

- KVO 防护

- Timer 防护



### JS 交互

- 整体方案
- 接口定义



### 实时涂鸦

- 整体方案

  通过socket传输画笔坐标信息，使用CAShapeLayer绘制

- 数据处理

  

- 界面绘制



## OC 语言



- 怎么理解 OC 是动态语言？

- 数组和链表的区别

- 遍历数组时增删数组元素

- NSArray 和 NSSet 有什么不同？

- NSArray 的底层结构

- NSArray copy 和NSMutableArray copy 的区别

- NSString 的底层结构

- `nil`、`NIL`、`NSNULL` 有什么区别？

- 如何定义一台 `iOS` 设备的唯一性?

- 在哪里下载苹果的源代码？- [链接](https://github.com/liberalisman/iOS-InterviewQuestion-collection/blob/master/Foundation/16.第十六题.md)

- isKindOfClass ， isM

- `objc_getClass()`、`object_getClass()`、`Class` 这三个方法用来获取类对象有什么不同？





## 内存



### [内存分区](evernote:///view/3106006/s17/76b195ad-9c1a-4b8d-82ca-9d1ad8a5a825/76b195ad-9c1a-4b8d-82ca-9d1ad8a5a825/)

​		iOS的内存分为五个区，代码区、常量区、全局区、堆、栈。常说的内存管理主要是说管理堆上的内存，栈内存是不需要我们管理的，CPU直接管理入栈出栈，比如说每次函数调用的时候都会有入栈操作，如果函数递归调用栈空间一直不释放就会栈溢出。



### [引用计数](https://juejin.cn/post/6844903847622606861#heading-25)

​		堆上的内存是使用引用计数来管理的，引用计数为0时会调用dealloc释放对象的内存空间，不过现在都是 `ARC`，基本不需要我们手动 `retain/release`了，除了`Core Foundation` 框架。

​       一般引用计数直接存储在[优化过的 `isa` 指针](https://juejin.cn/post/6844903898205913096)的 `extra_rc` 字段中（19位的内存存储引用计数减一），当引用计数值过大 这个字段存不下的时候，就会找到 `Runtime` 维护的全局哈希表（`SideTables`），`SideTables `里边存储的都是 `SideTable` 结构体，结构体里存着引用计数表（`refcnts`）和弱引用计数表（`weak_table`）

```c++
struct SideTable {
    spinlock_t slock;	// 保证原子操作的自旋锁
    RefcountMap refcnts;	// 引用计数 hash 表
    weak_table_t weak_table;	// weak 引用计数 hash 表
}
```



### [循环引用](https://www.jianshu.com/p/ddfd1b3c0298)

​		两个对象互相强引用彼此，彼此的引用计数始终都是 `1 `，导致无法释放。比如 `delegate`、`Block`、`Timer`。



### [weak](evernote:///view/3106006/s17/42dc51d2-7c96-4193-80a6-c245f194cf32/42dc51d2-7c96-4193-80a6-c245f194cf32/)

​		为了避免循环引用，可以使用 `weak` 修饰变量，`weak` 就是弱引用，弱引用是单独计数表，不会增加引用计数，当对象被释放的时候，所有指向它的弱引用都会自动被置空。根据对象内存地址（Key）获取所有` weak` 指针地址（Value）的数组，然后遍历数组设为 `nil`，最后从 `weak` 表中移除。

​		 **释放过程**：调用 `dealloc` 后会判断一些对象的标记，如 `TaggedPointer` 对象不需要释放，有没有优化过 `isa`（`NONPOINTER_ISA`），有无关联对象、有无析构函数、有无弱引用、有没有用到 `SideTable`，没有的话会释放更快。

​		**TaggedPointer**（标记指针）：为了节省内存占用，苹果提出了 `Tagged Pointer`，用来存储小的对象（`NSNumber`、`NSDate`、`NSString`，`NSIndexPath`），`Tagged Pointer` 分成两部分，一部分直接保存数据，一部分作为特殊标记，直接存储在栈中，访问速度更快。（`Tagged Pointer` 没有 `isa`，通过标记位判断）



### 总结

​		`ARC` 在编译期自动插入管理引用计数的代码，`TaggedPointer` 直接存在栈上，`extra_rc` 管理引用计数，思路都是减轻开发负担，减少堆内存开销提高性能，让开发者不必过度关心内存管理。



- **常见内存泄漏场景**

  - 循环引用。
  - 注册 NSNotification 没有移除。
  - Core Foundation 框架的函数没有手动释放。
  - 大循环产生大量的临时对象，循环结束才释放，可能导致内存泄漏，可在循环中创建 autoReleasePool，及时释放占用内存大的临时变量，减少内存占用峰值。

  

- **如何检测内存泄漏**

  - Analyzer（静态分析）
  - MLeaksFinder (第三方工具)
  - Instruments Leaks (动态检测)



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

![img](https://user-gold-cdn.xitu.io/2019/7/27/16c3202d97f29bae?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- [[self class] 和 [super class] 区别](https://www.jianshu.com/p/6bcef7ebc8af)

  class 函数是基类（NSObject）实现的

### 消息传递



### 消息转发



### Method Swizzling





- 对象调用方法的流程



## Runloop

- RunLoop 是用来处理事件的消息循环，在没有事件处理的时候，会使线程进入睡眠模式，从而节省 CPU 资源，提高程序性能。

- OC 中的 Runloop，**NSRunLoop** 和 **CFRunLoopRef**。
  - CFRunLoopRef 是在 CoreFoundation 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。
  - NSRunLoop 是基于 CFRunLoopRef 的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。



### Runloop 的构成

​		一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。如果需要切换 Mode，只能w退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。

![RunLoop_0](https://blog.ibireme.com/wp-content/uploads/2015/05/RunLoop_0.png)一		

- **Mode**

  | Mode                        | 功能                                                         |
  | --------------------------- | ------------------------------------------------------------ |
  | kCFRunLoopDefaultMode       | 默认Mode，通常主线程是在这个Mode下运行                       |
  | UITrackingRunLoopMode       | 界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响 |
  | UIInitializationRunLoopMode | 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用     |
  | GSEventReceiveRunLoopMode   | 接受系统事件的内部 Mode，通常用不到                          |
  | kCFRunLoopCommonModes       | 占位用的Mode，作为标记 kCFRunLoopDefaultMode 和 UITrackingRunLoopMode |

  

  > **CommonModes** ：Mode 可以将自己标记为 "Common" 属性，主线程的 RunLoop 里 DefaultMode 和 UITrackingMode。这两个 Mode 都已经被标记为"Common"属性。
  >
  > 比如 NSTimer，想在两个 Mode 中都能得到回调，一种办法就是将这个 Timer 分别加入这两个 Mode。还有一种方式，就是将 Timer 加入到 RunLoop 的 **CommonModes** 中。

  

  - **Source**

    事件。。。。。。。。。。

    

  - **Timer**

    ​	是基于时间的触发器，它和 NSTimer 是 toll-free bridged 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

    > 如果子线程没有创建 Runloop ，NSTimer 是不会回调的。

    

  - **Observer**

    ​	观察者，每个 Observer 都包含了一个回调（函数指针），当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。

    

  > 上面的 Source/Timer/Observer 被统称为 **mode item**，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。如果一个 mode 中一个 item 都没有，则 RunLoop 会直接退出，不进入循环。
  >
  > 参考：常驻线程，AFN常驻线程

  

- **Mode 的相关接口**

```objective-c
// 添加事件
CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
// 添加监听
CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
// 添加定时器
CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
// 移除Source、Observer、Timer
CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
```



>卡顿检测中的，监测主线程 Runloop 状态，就是通过 `CFRunLoopAddObserver` 进行监听的，传入**CommonModes**。



### Runloop 的状态

```c
 	kCFRunLoopEntry     = (1UL << 0), 		// 即将进入Loop

  kCFRunLoopBeforeTimers = (1UL << 1), 	// 即将处理 Timer

  kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source

  kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠

  kCFRunLoopAfterWaiting = (1UL << 6), 	// 刚从休眠中唤醒

  kCFRunLoopExit     = (1UL << 7), 			// 即将退出Loop
```





### Runloop 和线程

​		线程和 [RunLoop](https://blog.ibireme.com/2015/05/18/runloop/) 之间是一一对应的，保存在一个全局的 Dictionary 里。RunLoop 是通过懒加载的方式创建的，在线程结束时销毁 RunLoop。只能在当前线程的内部获取其 RunLoop（主线程除外）。



### AutoreleasePool

​		App启动后，系统在主线程 RunLoop 里注册了两个 Observer，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。

​		第一个 Observer 监视的事件是 **Entry (即将进入Loop)**，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

​		第二个 Observer 监视了两个事件： 

​		**BeforeWaiting(准备进入休眠)** 时调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；

​		**Exit(即将退出Loop)** 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

​		在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。



### PerformSelecter

​		当调用 NSObject 的 performSelecter:afterDelay: 后，实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中。所以如果当前线程没有 RunLoop，则这个方法会失效。

​		当调用 performSelector:onThread: 时，实际上其会创建一个 Timer 加到对应的线程去，同样的，如果对应线程没有 RunLoop 该方法也会失效。



### 使用场景

- **AFNetworking 常驻线程**

  ​		单独创建一个线程，并在这个线程中启动了一个 RunLoop，RunLoop 启动前内部必须要有至少一个 Timer/Observer/Source，所以 AFNetworking 在` [runLoop run] ` 之前先创建了一个新的 NSMachPort 添加进去了。通常情况下，调用者需要持有这个 NSMachPort (mach_port) 并在外部线程通过这个 port 发送消息到 loop 内；但此处添加 port 只是为了让 RunLoop 不至于退出，并没有用于实际的发送消息。

```objective-c
@autoreleasepool {
    [[NSThread currentThread] setName:@"AFNetworking"];
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    [runLoop run];
}
```

​			当需要这个后台线程执行任务时，通过调用 `[NSObject performSelector:onThread:..]` 将这个任务扔到了后台线程的 RunLoop 中。



- **定时器 NSTimer**





### 常见问题

- 子线程使用 NSTimer，没有获取过 Runloop 会怎样 ？

  ​	定时器不会回调，因为子线程执行完任务后就会销毁，没有 Runloop 来管理线程。

​		



## 多线程

### 进程

​		是指在系统中**正在运行**的一个应用程序,每个进程之间是**独立的**，每个进程均运行在其专用且**受保护的内存空间内**。

- **进程通信**

  ​	进程通信通过虚拟内存。。。。。

### 线程

​		一个进程要想执行任务，必须要有线程（每个进程至少要有1个线程）,一个进程的所有任务都在线程中执行。iOS程序运行后，默认会开启1个线程，称为主线程。

- **使用场景**

  ​	网络请求、加载图片、下载文件、数据存储等。

- **多线程的优点**

  ​	适当的使用多线程可以提高执行效率，提高资源利用率。（CPU、内存）

- **多线程的缺点**

  	1. 512KB的栈空间，以及1KB的内核空间。
  	2. 如果开启大量的线程，会降低程序的性能。
  	3. CPU开销大，线程越多CPU调度线程的开销就越大。
  	4. 增加程序设计复杂性，比如线程之间的通信、多线程的数据共享。

  

- **串行：**按顺序执行任务，在同一时间内，一个线程只能执行一个任务。
- **并行：**同一个队列先后添加的多个任务可以同时并列执行。（同步不行）
- **并发：**其实是CPU快速地在多条线程之间切换，造成并行的假象。
- **同步：**阻塞当前线程，在当前线程中执行任务，不具备开启新线程的能力，**都是串行**执行任务。
- **异步：**不阻塞当前线程，在新的线程中执行任务，具备开启新线程的能力。
- **队列：**队列是一种先进先出（FirstInFirstOut，FIFO）的线性表。
- **线程间通信：**一个例子，子线程请求数据成功后将数据传到主线程刷新 UI



**多线程执行状态**

| 重要*****            | 并发队列                             | 手动创建的串行队列                   | 主队列                               |
| -------------------- | ------------------------------------ | ------------------------------------ | ------------------------------------ |
| **同步（ `sync）`**  | **没有开启新线程<br />串行执行任务** | **没有开启新线程<br />串行执行任务** | **没有开启新线程<br />串行执行任务** |
| **异步（ `async）`** | **开启了子线程<br />并发执行任务**   | **开启了子线程<br />串行执行任务**   | **没有开启新线程<br />串行执行任务** |



**多线程的四种方案**

| 方案            | 简介                                                         | 语言 | 声明周期   | 使用频率 |
| --------------- | ------------------------------------------------------------ | ---- | ---------- | -------- |
| **pthread**     | 一套通用的多线程API<br />适用于Unix/Linux/Win等系统<br />跨平台、可移植<br />使用难度大 | C    | 开发者管理 | 几乎不用 |
| **NSThread**    | 面向对象<br />简单易用，可直接操作线程对象                   | OC   | 开发者管理 | 偶尔使用 |
| **GCD**         | 旨在替代NSThread等线程计数<br />充分利用了设备的多核         | C    | 自动管理   | 经常使用 |
| **NSOperation** | 基于GCD的封装<br />更加面向对象<br />比GCD多了一些简单实用的功能 | OC   | 自动管理   | 经常使用 |



**问答**

- **线程和 Runloop**

  ​	`RunLoop`对象和线程是一一对应关系；

  ​	`RunLoop`保存在一个全局的`Dictionary`里，线程作为`key`，`RunLoop`作为`value`；

  ​	如果没有`RunLoop`，线程执行完任务就会退出；如果没有`RunLoop`，主线程执行完`main()`函数就会退出，程序就不能处于运行状态；

  ​	`RunLoop`创建时机：线程刚创建时并没有`RunLoop`对象，`RunLoop`会在第一次获取它时创建；

  ​	`RunLoop`销毁时机：`RunLoop`会在线程结束时销毁；

  ​	主线程的`RunLoop`已经自动获取（创建），子线程默认没有开启`RunLoop`；

  ​	主线程的`RunLoop`对象是在`UIApplicationMain`中通过`[NSRunLoop currentRunLoop]`获取，一旦发现它不存在，就会创建`RunLoop`对象。



- **线程和队列的关系？**

  ​	1

  

- **如果线程非常非常多会怎样？**

  ​	CPU 会在 N 多线程之间调度，消耗大量的 CPU 资源，而且每条线程被调度执行的频次会降低（参考同时下载)

  

- **多线程读写同一文件会怎样？如何确保线程安全？**

  ​	多线程读写同一文件可能会文件错误，使用串行队列或者加锁，确保同一时间只有一个线程在读写文件。

  

- **`NSMutableArray`、和 `NSMutableDictionary`是线程安全的吗？`NSCache`呢？**

  ​	官方文档说明 `NSMutableArray`、和 `NSMutableDictionary` 不是线程安全的，多线程增删可能会越界等错误。`NSCache` 是线程安全的。

  

- **使用多线程的注意点**

  - UI 操作都在主线程执行
  - 耗时的操作尽量不在主线程，可能会产生卡顿



### 锁

​		锁是一种同步机制，用于多线程环境中对资源访问的限制。可以理解为用于排除并发的一种策略。



- **死锁**

  ​	指多个线程在运行过程中因争夺资源而造成的一种互相等待，如果没有外力作用，这些线程都将无法继续执行。（可以举例同步，现金买可乐）

  

  - **死锁产生必须同时满足的条件**

    1. **互斥条件：**指进程对所分配到的资源进行排它性使用，即在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求该资源，则请求者只能等待，直至占有该资源的进程用毕释放。
    2. **请求和保持条件：**指进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源又被其它进程占有，此时请求进程阻塞，但又对自己获得的其它资源保持不放。
    3. **不剥夺条件：**指进程已获得资源，在使用完之前，不能被剥夺，只能在使用完时由自己释放。
    4. **环路等待条件：**指在发生死锁时，必然存在一个进程—资源的环形链，即进程集合（P0，P1,P2,…,Pn）中的P0正在等待一个P1占用的资源；P1正在等待一个P2占用的资源，……,Pn正在等待已被P0占用的资源。

    

  - **解决/避免死锁**

    	1. **按顺序加锁：**线程间加锁的顺序各不一致，导致死锁，每个线程都按同一个加锁顺序就不会出现死锁。
    	2. **使用时限锁：**每个获取锁的时候加上个时限，如果超过某个时间就放弃获取锁之类的。
    	3. **死锁检测：**按线程间获取锁的关系检测线程间是否发生死锁，如果发生死锁就执行一定的策略，如终断线程或回滚操作等。



- 互斥锁
- 同步锁
- 自旋锁
- 条件锁
- 递归锁
- 信号量

​		



### NSThread



- `[NSThread currentThread]` 获取当前线程
- 



### GCD

​		为多核并行运算提出的解决方案，会自动合理地利用更多的CPU内核（比如双核、四核），最重要的是它会自动管理线程的生命周期（创建线程、调度任务、销毁线程），完全不需要我们管理。基于 `c语言`，使用 Block 调用更加方便。



- **dispatch_queue_create**

  ​	创建并行/串行队列

- **dispatch_get_global_queue**

  ​	按优先级获取全局并发队列（高、中、低、后台）四个优先级

- **dispatch_get_main_queue**

  ​	获取主线程队列

- **dispatch_group_t**

  ​	任务组，等待一组任务都完成后执行后续任务。有同步（wait）和异步（notify）两种方式。

- **dispatch_barrier_async**

  ​	`	barrier` 是栅栏的意思，前面的任务执行完之后才执行，后面的任务等它执行结束后才去执行。确保提交的任务在特定时间上是指定队列上唯一被执行的任务。

  ![Dispatch-Barrier](https://camo.githubusercontent.com/4df517d9d9c6d0cc767b8cac6f36b740213e065189cbcd475cddba909fc21e8b/68747470733a2f2f6b6f656e69672d6d656469612e72617977656e6465726c6963682e636f6d2f75706c6f6164732f323031342f30312f44697370617463682d426172726965722e706e67)

  ```c
  dispatch_barrier_async(queue, ^{
  		NSLog(@"您在执行barrier");
  });
  ```

  > **传入的** `queue` **不能是全局的并发队列**，否则就是异步并发，栅栏可能无效。
  >
  > 串行队列没必要使用栅栏执行任务，本来就是按顺序执行。

  

- **dispatch_once**

  ​	`dispatch_once()` 以线程安全的方式执行且仅执行其代码块一次，常用来实现单例模式。（这只是让访包含的代码线程安全。它绝对没有让类本身线程安全）

- **dispatch_after**

  ​	定时延迟执行任务。

- **dispatch_apply**

  ​	像一个 for 循环，但它能并发执行不同的迭代。这个函数是同步的，所以和普通的 for 循环一样，它只会在所有工作都完成后才会返回。

- **信号量**

  - **dispatch_semaphore_create (M)**

    创建一个值为 `M` 的信号量。

  - **dispatch_semaphore_signal (信号量)**

    释放信号量，使得该信号量的值加 `1`。

  - **dispatch_semaphore_wait (信号量，等待时间)**

    如果该信号量的值大于 `0`，则使其信号量的值减 `1`，否则，阻塞线程直到该信号量的值大于 `0` 或者达到等待时间。到达等待时间时该函数返回非 `0`。（参考卡顿监测）

    

- **GCD 并发队列实现机制？**

  ​	



- **`dispatch_async` 什么时候开启新线程？**

  ​	`dispatch_async` 提交任务到当前线程的队列时不会开新线程

  ​	对于`dispatch_async`,如果你在线程所属的队列使用dispatch_async提交一个Block到该队列,那么它就不会新开线程,否则它就会新开一个线程.

  

- **GCD 如何取消任务？**

  ​	方案一、取消函数 `dispatch_block_cancel`，不过这个也只能用于 `dispatch_block_create` 创建的 `dispatch_block_t`。

  ​	方案二、声明一个全局静态变量 `BOOL` ，当做判断条件，不满足就不执行任务。

  

- **GCD 如何做任务依赖？**

  ​	串行队列、信号量、线程组、栅栏，看具体需求。

  

- **GCD 设置线程最大并发数量？**

  ​	使用信号量控制

  

- **YYDispatchQueuePool**



### NSOperation



### 资源共享



### 线程同步





## UI 视图

### 绘制原理



### 异步绘制

​	文本渲染、图像绘制都是比较消耗性能的操作，而UILabel等控件都是在主线程进行的文本绘制。这会对性能产生比较大的影响。异步绘制将耗时的绘制操作放到子线程，减少主线程开销。

​	**CoreGraphic通常是线程安全的，所以可以进行异步绘制，显示的时候再放回主线程**

```objective-c
- (void)display {
   dispatch_async(backgroundQueue, ^{
       CGContextRef ctx = CGBitmapContextCreate(...);
       // draw in context...
       CGImageRef img = CGBitmapContextCreateImage(ctx);
       CFRelease(ctx);
       dispatch_async(mainQueue, ^{
           layer.contents = img;
       });
   });
}
```



### 事件传递机制



### 离屏渲染

​	当使用圆角，阴影，遮罩的时候，图层属性的混合体被指定为在未预合成之前不能直接在屏幕中绘制，所以就需要屏幕外渲染被唤起。（参考PS图层合并）所以当使用离屏渲染的时候会很容易造成性能消耗，因为在OPENGL里离屏渲染会单独在内存中创建一个屏幕外缓冲区并进行渲染，而屏幕外缓冲区跟当前屏幕缓冲区上下文切换是很耗性能的。



### 界面性能优化

1. **避免主线程阻塞**
2. **异步绘制**
3. **简化视图结构**
4. **简化视图结构**
5. **Cell高度缓存**



### 图像处理



### 动画

- UIView CALayer之间的关系

  - UIView 继承 UIResponder，接收点击事件，CALayer 直接继承 NSObject，并没有相应的处理事件的接口。

  - UIView 是 CALayer 的delegate

  - UIView 主要处理事件，CALayer 负责绘制就更好

  

- 事件响应链

- viewcontroller 生命周期

- setNeedsDisplay` 和 `layoutIfNeeded

- 如何以通用的方法找到当前显示的`ViewController`

- `Bounds` 和 `Frame` 的区别?

- `LoadView`方法了解吗？



## 网络

### HTTP

​	[http协议详解](https://blog.csdn.net/hansionz/article/details/86137260)

- **三次握手** (可以举例小兔子和月亮写信)

  1. 客户端  - >  服务端，发送报文（等待回复中，不知道能否收到，不知道是否完整）
  2. 服务端 - > 客户端，我收到你的报文啦，你看看是不是这个（收到回复，确认完整性）
  3. 客户端  - >  服务端，对就是这个！可以建立连接啦（告诉服务器一切正常）

  

- 四次挥手

  1. 客户端  - >  服务端，我要结束啦
  2. 服务端 - > 客户端，好的，这是你收到的信息你看对不对，对就可以结束了
  3. 服务端 - > 客户端，顺便看看我收到的信息对不对，对的话我也可以结束了
  4. 客户端  - >  服务端，我的信息正确，你的也正确，可以断开啦

   

- 为什么是四次挥手？

  ​	客户端与服务端建立的连接是双通的，这也就意味着客户端可以向服务端发送请求，服务端向客户端返回响应，同样的也可以从服务端向客户端发送，然后由客户端向服务端返回，如果你只关闭了客户端，向服务端通道，这只叫半关闭状态，并没有完全的切断客户端与服务端的关联。

  

### HTTPS

​		[非常详细易懂](https://github.com/youngwind/blog/issues/108)



### TCP/UDP

​	https://blog.csdn.net/zhang6223284/article/details/81414149

- TCP / UDP 的区别
  - TCP 面向连接，UDP是无连接的，即发送数据之前不需要建立连接



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

### Swift枚举的本质

### Swift中struct和class的区别



## 常见问题

### 内存和虚拟内存如何映射

### 图片的内存如何计算

### URL 不变，图片资源变了怎么处理

### 热修复原理

### 无痕埋点

### APNS 推送机制

### APP 签名机制

### Charles原理

​		https://github.com/youngwind/blog/issues/108



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

### 时间复杂度

### 空间复杂度



- BFS(广度优先搜索)

 广度优先搜索算法（Breadth-First-Search），是一种图形搜索算法。简单的说，BFS是从根节点开始，沿着树(图)的宽度遍历树(图)的节点。如果所有节点均被访问，则算法中止。BFS同样属于盲目搜索。一般用队列数据结构来辅助实现BFS算法。



- DFS（深度优先搜索）

深度优先搜索算法（Depth-First-Search），是搜索算法的一种。它沿着树的深度遍历树的节点，尽可能深的搜索树的分支。当节点v的所有边都己被探寻过，搜索将回溯到发现节点v的那条边的起始节点。这一过程一直进行到已发现从源节点可达的所有节点为止。如果还存在未被发现的节点，则选择其中一个作为源节点并重复以上过程，整个进程反复进行直到所有节点都被访问为止。DFS属于盲目搜索。

 深度优先搜索是图论中的经典算法，利用深度优先搜索算法可以产生目标图的相应拓扑排序表，利用拓扑排序表可以方便的解决很多相关的图论问题，如最大路径问题等等。一般用堆数据结构来辅助实现DFS算法。


### 选择排序

```objective-c
//选择排序
- (void)selectionSort 
{
	NSMutableArray *array = @[@"3",@"2",@"5",@"4",@"7",@"9",@"8",@"9",@"10",@"33"].mutableCopy;
	for (int i = 0; i < array.count; i++) {
		for (int j = i+1; j <array.count ; j++) {
			if ([array[i] intValue] > [array[j] intValue]) {
				int temp = [array[i] intValue];
				array[i] = array[j];
				array[j] = [NSNumber numberWithInt:temp];
			}
		}
	}
	NSLog(@"选择排序后: %@",array);
}
```



### 冒泡排序

```objective-c
///冒泡排序
- (void)bubbleSort:(NSMutableArray *)arr {
	//两两比较 需要比较 count - 1 次
	for (int i = 0; i < arr.count-1; i++) {
	//以下 for 循环走完后即可将最大值放到最后，此处count-1 后还需要 - i
		for (int j = 0; j < arr.count-1-i; j++) {
		// 如果 前面的数大于后面的数，就将这两个数交换位置
			if ([arr[j] intValue] > [arr[j+1] intValue]) {
				int temp = [arr[j] intValue];
				arr[j] = arr[j+1];
				arr[j+1] = [NSNumber numberWithInt:temp];
			}
		}
	}
}
NSMutableArray *arr = @[@2, @4, @9, @8, @1, @0, @3, @5, @2].mutableCopy;
NSLog(@"%@", arr);
[self bubbleSort:arr];
NSLog(@"%@", arr);
```



### 字符串反转

```objective-c
- (NSString *)stringByReversed:(NSString *)str
{
    NSMutableString *mutableSgtr = [[NSMutableString alloc]init];
    for (NSInteger i = str.length; i>0; i--) {
        [mutableSgtr appendString:[str substringWithRange:NSMakeRange(i-1, 1)]];
    }
    return mutableSgtr;
}
```



### 链表反转



### 链表是否有环



### 有序数组合并

```objective-c
- (NSArray *)mergeOrderArrayWithFirstArray: (NSMutableArray *)array1 secondArray: (NSMutableArray *)array2 {
  // 全为空不处理
  if (!array1.count && !array2.count) {
    return @[];
  }
  // 一个为空返回另外一个
  if (!array1.count) {
    return array2;
  }
  if (!array2.count) {
    return array1;
  }
  NSMutableArray *endArray = [NSMutableArray array];
  while (1) {
    if ([array1[0] integerValue] < [array2[0] integerValue]) {
      [endArray addObject:array1[0]];
      [array1 removeObjectAtIndex:0];
    }else {
      [endArray addObject:array2[0]];
      [array2 removeObjectAtIndex:0];
    }
    if (!array1.count) {
      [endArray addObjectsFromArray:array2];
      break;
    }
    if (!array2.count) {
      [endArray addObjectsFromArray:array1];
      break;
    }
  }
  return endArray;
}
```



### 二叉树的前缀遍历