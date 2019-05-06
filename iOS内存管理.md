### 定时器

---

三种常见定时器

+ 1、NSTimer
+ 2、CADisplayLink
+ 3、GCD


NSTimer 

NSTimer是iOS中最常用的定时器，其通过RunLoop来实现，一般情况下比较准确，但是当前循环耗时操作比较多时，会出现延迟问题，同时，也受所加入的RunLoopMode影响。

```
/// 构造并开启(启动NSTimer本质上是将其加入RunLoop中)
// "scheduledTimer"前缀的为自动启动NSTimer的，如:
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block

/// 构造但不开启
// "timer"前缀的为只构造不启用的，如:
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block

//定时器的释放一定要先将其终止，而后才能销毁对象
- (void)invalidate;
//立即执行(fire)
//我们对定时器设置了延时之后，有时需要让它立刻执行，可以使用fire方法:
- (void)fire;


```


#### 简单使用

```
	//方法1
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self 
selector:@selector(doTask) userInfo:nil repeats:YES];

//方法2
self.timer = [NSTimer timerWithTimeInterval:1.0 target:self 
selector:@selector(doTask) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
```


#### CADisplayLink

CADisplayLink是基于屏幕刷新的周期，所以其一般很准时，每秒刷新60次。其本质也是通过RunLoop，所以不难看出，当RunLoop选择其他模式或被耗时操作过多时，仍旧会造成延迟。

其使用步骤为 创建CADisplayLink->添加至RunLoop中->终止->销毁。代码如下

```
self.link = [CADisplayLink displayLinkWithTarget:self selector:@selector(doTask)];
[self.link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];

//在dealloc中
// 终止定时器
[disLink invalidate];
// 销毁对象
disLink = nil;
```

同时，由于其是基于屏幕刷新的，所以度量单位是“每帧",其提供了根据屏幕刷新来设置间隔的frameInterval属性，在日常开发中，适当使用CADisplayLink甚至有优化作用，比如对于需要动态计算进度的进度条，由于进度反馈主要是为了UI更新，那么当计算进度的频率超过帧数时，就造成了很多无畏的计算，如果将计算进度的方法绑定到CADisplayLink上来调用，则只在每次屏幕刷新的时候计算进度，优化了性能，


#### GCD 定时器实际上是使用了dispatch源(dispatch source)，dispatch源监听系统内核对象并处理，通过系统级调用，更加精准


利用NSProxy解决NSTimer内存泄漏的问题：

NSProxy是一个抽象类，必须继承实例化子类才能使用，NSProxy具体使用参考【官方示例】，在上面示例中通过消息转发实现了同时对NSProxy发送NSMutableString和NSMutableArray类型的消息间接的实现多重继承。


什么是NSProxy：

+ NSProxy是一个抽象的基类，是根类，与NSObject类似
+ NSProxy和NSObject都实现了协议
+ 提供了消息转发的通用接口

如何使用NSProxy来转发消息？

+ 需要继承NSProxy

+ 重写如下的两个方法：

	+ methodSignatureForSelector
	+ forwardInvocation:

代码使用，写一个继承自NSProxy的MyProxy类

```
	
	@interface MyProxy : NSProxy
+ (instancetype)proxyWithTarget:(id)target;
@property (weak, nonatomic) id target;
@end

@implementation MyProxy
+ (instancetype)proxyWithTarget:(id)target
{
// NSProxy对象不需要调用init，因为它本来就没有init方法
MyProxy *proxy = [MyProxy alloc];
proxy.target = target;
return proxy;
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
return [self.target methodSignatureForSelector:sel];
}

- (void)forwardInvocation:(NSInvocation *)invocation
{
[invocation invokeWithTarget:self.target];
}

@end


```

使用：

```
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:[MyProxy 
proxyWithTarget:self] selector:@selector(doTask) userInfo:nil repeats:YES];

现在在dealloc中销毁定时器，发现会执行dealloc方法

```

#### GCD封装

这三种定时器最准确的还是GCD,但是我们发现在使用的时候，我们会写很多代码，所以为了下一次使用的时候不写那么多的代码，我们就简单的封装一下：

**接口**在写具体的代码之前，我们首先需要考虑的就是我们应该提供什么接口：


+ **1、**肯定需要里面执行事件，所以需要一个回调
+ **2、**设置什么时候开始启动定时器
+ **3、**设置定时器的时间间隔
+ **4、**是否重复执行
+ **5、**是否多线程执行
+ **6、**提供取消任务的接口




