**iOS开发中常用的锁有如下几种：**

+ @synchronized

+ NSLock 对象锁

+ NSRecursiveLock 递归锁

+ NSConditionLock 条件锁

+ pthread_mutex 互斥锁（C语言）

+ dispatch_semaphore 信号量实现加锁（GCD）

+ OSSpinLock (暂不建议使用，原因参见[这里](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/))

>性能对比：

![](https://upload-images.jianshu.io/upload_images/2208956-4a024a1c6c6214db.png?imageMogr2/auto-orient/)

> + @synchronized 关键字加锁 互斥锁，性能较差不推荐使用

```
@synchronized(这里添加一个OC对象，一般使用self) {
       这里写要加锁的代码
  }
　注意点
　　 1.加锁的代码尽量少
　　 2.添加的OC对象必须在多个线程中都是同一对象
    3.优点是不需要显式的创建锁对象，便可以实现锁的机制。
    4. @synchronized块会隐式的添加一个异常处理例程来保护代码，该处理例程会在异常抛出的时候自动的释放互斥锁。所以如果不想让隐式的异常处理例程带来额外的开销，你可以考虑使用锁对象。

```

> + NSLock 互斥锁 不能多次调用 lock方法,会造成死锁

在Cocoa程序中NSLock中实现了一个简单的互斥锁。
所有锁（包括NSLock）的接口实际上都是通过NSLocking协议定义的，它定义了lock和unlock方法。你使用这些方法来获取和释放该锁。

NSLock类还增加了tryLock和lockBeforeDate:方法。
tryLock试图获取一个锁，但是如果锁不可用的时候，它不会阻塞线程，相反，它只是返回NO。

lockBeforeDate:方法试图获取一个锁，但是如果锁没有在规定的时间内被获得，它会让线程从阻塞状态变为非阻塞状态（或者返回NO）。

> + NSRecursiveLock 递归锁
> 
> 使用锁最容易犯的一个错误就是在递归或循环中造成死锁
如下代码中，因为在线程1中的递归block中，锁会被多次的lock，所以自己也被阻塞了

```
//创建锁
    _mutexLock = [[NSLock alloc]init];
  
    //线程1
    dispatch_async(self.concurrentQueue, ^{
        static void(^TestMethod)(int);
        TestMethod = ^(int value)
        {
            [_mutexLock lock];
            if (value > 0)
            {
                [NSThread sleepForTimeInterval:1];
                TestMethod(value--);
            }
            [_mutexLock unlock];
        };
        
        TestMethod(5);
    });
```

**此处将NSLock换成NSRecursiveLock，便可解决问题。NSRecursiveLock类定义的锁可以在同一线程多次lock，而不会造成死锁。
递归锁会跟踪它被多少次lock。每次成功的lock都必须平衡调用unlock操作。
只有所有的锁住和解锁操作都平衡的时候，锁才真正被释放给其他线程获得。**

```
//创建锁
    _rsLock = [[NSRecursiveLock alloc] init];
    
   //线程1
    dispatch_async(self.concurrentQueue, ^{
        static void(^TestMethod)(int);
        TestMethod = ^(int value)
        {
            [_rsLock lock];
            if (value > 0)
            {
                [NSThread sleepForTimeInterval:1];
                TestMethod(value--);
            }
            [_rsLock unlock];
        };
        
        TestMethod(5);
    });
```

NSConditionLock 条件锁

```
 //主线程中
    NSConditionLock *theLock = [[NSConditionLock alloc] init];
    
    //线程1
    dispatch_async(self.concurrentQueue, ^{
        for (int i=0;i<=3;i++)
        {
            [theLock lock];
            NSLog(@"thread1:%d",i);
            sleep(1);
            [theLock unlockWithCondition:i];
        }
    });
    
    //线程2
    dispatch_async(self.concurrentQueue, ^{
        [theLock lockWhenCondition:2];
        NSLog(@"thread2");
        [theLock unlock];
    });
```
![](https://upload-images.jianshu.io/upload_images/2208956-3737813c40d45c9f.png?imageMogr2/auto-orient/)

在线程1中的加锁使用了lock，是不需要条件的，所以顺利的就锁住了。
unlockWithCondition:在开锁的同时设置了一个整型的条件 2 。
线程2则需要一把被标识为2的钥匙，所以当线程1循环到 i = 2 时，线程2的任务才执行。

NSConditionLock也跟其它的锁一样，是需要lock与unlock对应的，只是lock,lockWhenCondition:与unlock，unlockWithCondition:是可以随意组合的，当然这是与你的需求相关的。

+ pthread_mutex 互斥锁

```
__block pthread_mutex_t mutex;
    pthread_mutex_init(&mutex, NULL);
    
    //线程1
    dispatch_async(self.concurrentQueue), ^{
        pthread_mutex_lock(&mutex);
        NSLog(@"任务1");
        sleep(2);
        pthread_mutex_unlock(&mutex);
    });
    
    //线程2
    dispatch_async(self.concurrentQueue), ^{
        sleep(1);
        pthread_mutex_lock(&mutex);
        NSLog(@"任务2");
        pthread_mutex_unlock(&mutex);
    });
```

+ dispatch_semaphore 信号量实现加锁

GCD中也已经提供了一种信号机制，使用它我们也可以来构建一把”锁”(从本质意义上讲，信号量与锁是有区别，

```
 // 创建信号量
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    //线程1
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
         NSLog(@"任务1");
        sleep(10);
        dispatch_semaphore_signal(semaphore);
    });
    
    //线程2
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(1);
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"任务2");
        dispatch_semaphore_signal(semaphore);
    });
```

+ OSSpinLock

```
//设置票的数量为5
    _tickets = 5;
    //创建锁
    _pinLock = OS_SPINLOCK_INIT;
    //线程1
    dispatch_async(self.concurrentQueue, ^{
        [self saleTickets];
    });
    //线程2
    dispatch_async(self.concurrentQueue, ^{
        [self saleTickets];
    });

- (void)saleTickets {
    
        while (1) {
            [NSThread sleepForTimeInterval:1];
            //加锁
            OSSpinLockLock(&_pinLock);
            
            if (_tickets > 0) {
                _tickets--;
                NSLog(@"剩余票数= %ld, Thread:%@",_tickets,[NSThread currentThread]);
                
            } else {
                NSLog(@"票卖完了  Thread:%@",[NSThread currentThread]);
                break;
            }
            //解锁
            OSSpinLockUnlock(&_pinLock);
        }

}
```
[不建议使用](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)


