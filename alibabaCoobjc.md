[coobjc](https://github.com/alibaba/coobjc/blob/master/README_cn.md) 

##### 这个库为Objective-C 和Swift提供了协程功能，coobjc支持await、generator和actor model，接口参考了C#、Javascript和Kotlin中的很多设计，我们还提供了[cokit]()库为Foundation和UIkit中的部分API

### 分析

> 协程是计算机程序的一类组件,推广了非抢先多任务的子程序，允许“执行”被挂起与被恢复。相对子例程而言，协程更为一般和灵活，但在实践中使用没有子例程那样广泛。协程源自Simula和Modula-2语言，但也有其他语言支持。协程更适合于用来实现彼此熟悉的程序组件，如合作式多任务、异常处理、事件循环、迭代器、无限列表和管道。用术语“coroutine”并用于构建汇编程序。


+ 协程的历史还是比较悠久的，只是Objective-C不支持，比如Python以及swift，甚至C语言都是支持协程的。


#### 协程的作用

基于Block的异步编程回调是目前iOS使用最广泛的异步编程方式，iOS系统提供的GCD库让异步开发变得很简单方便，但是基于这种编程方式的缺点也有很多，主要有以下几点：

+ 容易进入“嵌套地狱”。
+ 错误处理复杂和冗长。
+ 容易忘记调用completion handler
+ 条件执行变得很困难
+ 从相互独立的调用中组合返回结果变得极其困难。
+ 在错误的线程中继续执行
+ 难以定位原因的多线程崩溃
+ 锁的信号量滥用带来的卡顿、卡死

在coroutine_context.h，主要定义了三个方法

```
extern int coroutine_getcontext (coroutine_ucontext_t *__ucp);
extern int coroutine_begin (coroutine_ucontext_t *__ucp);
extern void coroutine_makecontext (coroutine_ucontext_t *__ucp, IMP func, void *arg, void *stackTop);


extern int coroutine_setcontext (coroutine_ucontext_t *__ucp);

coroutine_setcontext(coroutine_ucontext_t *__ucp), setContext的上下文cut应该通过getContext或者makeContext取得，如果调用成功则不返回，如果上下文是通过调用getContext()取得，程序会继续执行这个调用，如果上下文是通过调用makeContext取得，程序会调用makecontext函数的第二个参数指向的函数，如果func函数返回，则恢复makecontext第一个参数指向的上下文第一个参数指向的上下文context_t中指向的uc_link,如果uc_link为NULL,则线程退出。
	
```

#### 协程的操作

> 协程拥有和线程一样类似的操作，例如创建，启动，出让控制权，恢复，以及死亡。对应的，我们在coroutine.h看到了如下几个函数声明：

```
    /**
     Close and free a coroutine if dead.
     关闭一个协程如果它已经死亡
     @param co coroutine object
     */
    void coroutine_close_ifdead(coroutine_t *co);
    
     /**
     Add coroutine to scheduler, and resume the specified coroutine whatever.
     添加协程到调度器，并且立刻启动
     */
    void coroutine_resume(coroutine_t *co);
    
     /**
     Add coroutine to scheduler, and resume the specified coroutine if idle.
     添加协程到调度器
     */
    void coroutine_add(coroutine_t *co);
    
     /**
     Yield the specified coroutine now.
     出让控制权
     */
    void coroutine_yield(coroutine_t *co);
   
为了更好的控制各个操作中的数据，coobjc还提供了以下两个方法：

    /**
     Set/Get userdata.

     @param co       coroutine object
     @param userdata userdata pointer
     @param userdata_dispose callback when coroutine free.
     */
    void coroutine_setuserdata(coroutine_t *co, void *userdata, coroutine_func userdata_dispose);
    void *coroutine_getuserdata(coroutine_t *co);
    
以上就是coobjc的核心代码。
    
```
















 