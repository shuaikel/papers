#### RunLoop 运行循环，在程序运行过程中循环做一些事情，__应用范畴__

 + 1. 定时器（Timer）、PerformSelector
 + 2. GCD
 + 3. 事件响应、手势识别、界面刷新
 + 4. 网络请求
 + 5. AutoreleasePool

 

 
#### RunLoop对象 iOS中有两套API来访问和使用RunLoop
 
 > 1. Fundation : NSRunLoop
 
 > 2. Core Fundation : CFRunLoop
 
 NSRunLoop 是基于 CFRunLoop的一层OC包装，CFRunLoop是开源的，[地址为](https://opensource.apple.com/tarballs/CF/)
 
#### RunLoop与线程

+ 1. 每一条线程都有唯一的一个与之相对应的RunLoop对象

+ 2. RunLoop保存在一个全局的Dictionary里，线程作为key，RunLoop作为Value;

+ 3.  线程刚创建的时候，并没有RunLoop对象，RunLoop会在第一次创建她时创建。

+ 4. RunLoop会在线程结束时销毁

+ 5. 主线程的RunLoop已经自动获取（创建），子线程默认没有开启RunLoop


#### 获取RunLoop对象

<h4> Fundation </h4>

> 1. 获取当前线程的RunLoop对象[NSRunLoop currentRunLoop];

> 2. 获取主线程的RunLoop对象[NSRunLoop mainRunLoop]
> 

<h4> Core Foundation </h4>

> 1. 获取当前线程的RunLoop对象CFRunLoopGetCurrent()

> 2. 获取主线程的RunLoop对象CFRunLoopGetMain()
> 


#### RunLoop 相关类 Core Foundation中关于RunLoop 一共有5个类

> 1. CFRunLoopRef

> 2. CFRunLoopModeRef
 
> 3. CFRunLoopSourceRef

> 4. CFRunLoopTimerRef

> 5. CFRunLoopObserverRef


其中主要有下面几个：

```
	typedef struct __CFRunLoop *CFRunLoopRef;
	struct __CFRunLoop {
		pthread_t _pthread;
		CFMutableSetRef _commonModes;
		CFMutableSetRef _commonModeItems;
		CFRunLoopModeRef _currentMode;
		CFMutableSetRef _modes;
	};
	_pthread : 记录当前线程
	_commonModes : 
	_commonModeItems : 
	_currentMode : 当前mode类型
	_modes : 存放CFRunLoopMode 来查看一下CFRunLoopMode 都存放了哪些东西;
	我们在RunLoop源码中搜索 CFRunLoopMode 来查看一下CFRunLoopMode都存放了哪些东西
	
	typedef struct __CFRunLoopMode *CFRunLoopModeRef;
	struct __CFRunLoopMode {
		CFStringRef _name;
		CFMutableSetRef _sources0;
		CFMutableSetRef _sources1;
		CFMutableArrayRef _observers;
		CFMutableArrayRef _timers;
	}
	
	
	+ Source0: 
		> 处理触摸事件
		> performSelector:onThread:
	+ Source1:
		> 基于Port的线程间通信，
		> 系统事件的捕捉
	+ Timer:
		> NSTimer，
		> performSelector:withObject:afterDelay:
	+ Observers： 
		> 用于监听RunLoop的状态
		> UI刷新
		> Autorelease pool（BeforeWaiting）
	
```


#### CFRunLoopModeRef

1. CFRunLoopModeRef 代表着RunLoop的运行模式
2. 一个RunLoop 包含若干个Mode,每个Mode又包含若干个 Source0/Source1/Timer/Observer
3. RunLoop 启动的时候只能选择其中一个Mode作为currentMode;
4. 如果要切换Mode,只能退出当前的Loop,再重新选择一个Mode进入，不同组 Source0/Source1/Timer/Observer 互不影响
5. 如果Mode里面没有任何Source0/Source1/Timer/Observer，RunLoop会立刻退出

#### CFRunLoopModeRef常见的Mode

1. KCFRunLoopDefaultMode(NSDefaultRunLoopMode): App的默认Mode,通常主线程是这个Mode下运行的
2. UITrackingRunLoopMode： 界面追踪Mode,用于ScrollView 追踪触摸滑动，保证界面滑动时不受其他Mode 影响。

#### CFRunLoopObserverRef RunLoop的几种状态

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
		kCFRunLoopEntry = (1UL << 0), //即将进入runloop
		kCFRunLoopBeforeTimers = (1UL << 1), //即将处理timer
		kCFRunLoopBeforeSources = (1UL << 2), //即将处理source
		kCFRunLoopBeforeWaiting = (1UL << 5), //即将进入休眠
		kCFRunLoopAfterWaiting = (1UL << 6),  //刚从休眠中唤醒
		kCFRunLoopExit = (1UL << 7),          //即将退出Loop
		kCFRunLoopAllActivities = 0x0FFFFFFFU
};
	
```

#### RunLoop的运行逻辑

![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/images/RunLoop7.png)

每次运行RunLoop, 线程的RunLoop会自动处理之前未处理的消息，并通知相关的观察者，具体的调用顺序为：

+ 1. 通知观察者（observer）RunLoop即将启动，
+ 2. 通知观察者（observer）处理任何即将要开始的定时器
+ 3. 通知观察者（observer）即将处理source0事件
+ 4. 处理source0；
+ 5. 如果有source1，跳到第9步
+ 6. 通知观察者（observers）线程即将进入休眠
+ 7. 将线程置于休眠直到任一下面的事件发生
	+ source0事件触发
	+ 定时器触发
	+ 外部手动唤醒
+ 8. 通知观察者（observers）线程即将唤醒
+ 9. 处理唤醒时收到的时间，之后跳回2
	+ 1. 如果用户定义的定时器启动，处理定时器事件
	+ 2. 如果source0启动，传递相应的消息
+ 10. 通知观察者RunLoop结束


#### RunLoop 休眠原理

![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/images/RunLoop8.png)

在RunLoop即将休眠的时候，通过mach_msg()方法来让软件和硬件交互

+ 1. 在即将休眠的时候，程序调用mach_msg() 传递给CPU,并告诉CPU停止运行
+ 2. 即将启动RunLoop的时候，程序调用mach_msg()传递给CPU,告诉CPU开始工作



[链接](https://github.com/SunshineBrother/JHBlog/blob/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%BA%95%E5%B1%82/8%E3%80%81RunLoop.md)





