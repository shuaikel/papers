#GCD：
######Grand Central Dispatch（GCD）是Apple开发的一个多核编程的较新的解决办法，它主要用于优化应用程序以支持多核处理器以及其他对称多处理系统，它是一个在线程池模式的基础上执行的并发任务，在Mac OS X 10.6雪豹中首次推出，也可在iOS 4及以上版本使用。


##使用GCD的好处
+ GCD可用于多核的并行运算
+ GCD 会自动利用更多的 CPU 内核（比如双核、四核）
+ GCD 会自动管理线程的生命周期（创建线程、调度任务、销毁线程）
+ 程序员只需要告诉 GCD 想要执行什么任务，不需要编写任何线程管理代码

##GCD的任务和队列

#####任务：就是执行操作的意思，换句话说就是你在线程中执行的那段代码，在GCD中是放在block中的。执行任务有两种方式:同步执行（sync）和异步执行（async）。两者的主要区别是：（是否等待队列的任务执行结束，以及是否具备开启新线程的能力）


+ 同步执行（Sync）：

	+ 同步添加任务到指定的队列中，在添加的任务执行结束之前，会一直等待，直到队列里面的任务完成之后再继续执行。
	+ 只能在当前线程中执行任务，不具备开启新线程的能力。

+ 异步执行（Async）:
	+ 异步添加任务到指定的队列中，它不会做任何等待，可以继续执行任务。
	+ 可以在新的线程中执行任务，具备开启新线程的能力。

> ###### 注意：异步执行（Async）虽然具备开启新线程的能力，但是并不一定会开启新线程。这跟任务所指定的队列类型有关

#####队列(Dispatch Queue)：这里的队列指执行任务的等待队列，即用来存放任务的队列。队列是一种特殊的线性表，采用FIFO的原则，即新任务总是被插入到队列的末尾，而读取任务的时候总是从队列的头部开始读取。每读取一个任务，则从队列中释放一个任务。
![](https://user-gold-cdn.xitu.io/2018/2/24/161c5e4c0c8b1684?imageslim)

在GCD中有两种队列：串行队列和并发队列。两者都符合FIFO（先进先出）的原则。两者的主要区别是：执行顺序不同，以及开启线程数的不同。

+ 串行队列（Serial Dispatch Queue）：
	+ 每次只有一个任务被执行，让任务一个接着一个执行。（只开启一个线程，一个任务执行完毕后，在执行下一个任务）
+ 并发队列（Concurrent Dispatch Queue）：
	+ 可以让多个任务并发（同时执行），（可以开启多个线程，并且同时执行任务）
> 注意：并发队列 的并发功能只有在异步（Dispatch_async）函数下才有效

两者具体区别如下图所示：
![](https://user-gold-cdn.xitu.io/2018/2/24/161c5e4c21aa3095?imageslim) 

![](https://user-gold-cdn.xitu.io/2018/2/24/161c624e8be89b51?imageslim)

## GCD的使用步骤
GCD的使用步骤

+ 创建一个队列（串行队列或并发队列）

+ 将任务追加到任务的等待队列中，然后系统就会根据任务类型执行任务（同步执行或者异步执行）

> 队列的创建方法/获取方法

+ 可以使用dispatch\_queue\_creat来创建队列，需要传入两个参数，第一个参数表示队列的唯一标识符，用于DEBUG，可为空，Dispatch Queue的名称推荐使用应用程序ID这种逆序全程域名；第二个参数用来识别是串行队列还是并发队列。DISPATCH\_QUEUE\_SERIAL标识串行队列，DISPATCH\_QUEUE\_CONCURRENT标识并发队列。

+ 对于串行队列，GCD提供了一种特殊的串行队列：主队列（Main Dispatch Queue）
	+ 所有放在主队列中的任务，都会放到主线程中执行。
	+ 可使用dispatch\_get\_main\_queue()获得主队列。
	
+ 对于并发队列，GCD默认提供了全局并发队列（Global Dispatch Queue）
	+ 可以使用dispatch\_get\_global\_queue来获取。需要传入两个参数。第一个参数表示队列优先级，一般用DISPATCH\_QUEUE\_PRIORITY\_DEFAULT。第二个参数暂时没有用，传0即可。  


> 任务的创建方法

GCD提供了同步执行任务的创建方法
dispatch_sync 以及异步执行任务创建方法 dispatch\_async

| 区别 | 并发队列 | 串行队列 | 主队列 |
| ------| ------ | ------ | ------ |
| 同步(sync) | 没有开启新线程，串行执行任务 | 没有开启新线程，串行执行任务 | 没有开启新线程，串行执行任务|
| 异步(async) | 有开启新线程，并发执行任务 | 有开启新线程(1条)，串行执行任务 | 没有开启新线程，串行执行任务 |

> GCD线程间的通信
> 


##GCD的其他方法：
+ GCD的栅栏方法：dispatch_barrier\_async

	> 有时我们需要异步执行两组操作，而且第一组操作执行完之后，才能开始执行第二组操作。这样我们就需要一个相当于栅栏一样的一个方法将两组异步执行的操作组给分割起来，当然这里的操作组里可以包含一个或多个任务。这就需要用到dispatch\_barrier\_async 方法在两个操作组之间形成栅栏。
	> dispatch\_barrier\_async函数会等待前边追加到并发队列中的任务全部执行完毕之后，再将指定的任务追加到该异步队列中。然后在dispatch\_barrier\_async函数追加的任务全部执行完毕之后，异步队列才恢复为一般动作，接着追加任务到该异步队列并开始执行。具体如下图所示：

	![](https://user-gold-cdn.xitu.io/2018/2/24/161c6258a0c04a71?imageslim)

+ GCD的延时执行方法：dispatch_after
	> 当我们需要在指定时间之后执行某个任务。可以用GCD的dispatch_after函数来实现，需要注意的是：dispatch\_after函数并不是在指定时间之后才开始执行处理，而是在指定时间之后才将任务追加到主队列中，严格来说，这个时间并不是绝对准确的，但想要大致延迟执行任务，dispatch\_after函数是很有效的。
	
+ GCD一次性代码（只执行一次）：dispatch_once
	> 我们在创建单例、或者整个程序运行过程中只执行一次的代码时，我们就用到了GCD的dispatch\_once函数。使用dispatch\_once函数能保证某段代码在程序运行过程中只被运行一次，并且即使在多线程的环境下，dispatch\_once也可以保证线程安全。
	
+ GCD快速迭代方法：dispatch_apply
	> 通常我们会用for循环遍历，但是GCD给我们提供了快速迭代的函数dispatch\_apply。dispatch\_apply按照指定的次数将指定的任务追加到指定的队列中，并等待全部队列执行结束。

```
/**
 * 快速迭代方法 dispatch_apply
 */
- (void)apply {
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    NSLog(@"apply---begin");
    dispatch_apply(6, queue, ^(size_t index) {
        NSLog(@"%zd---%@",index, [NSThread currentThread]);
    });
    NSLog(@"apply---end");
}

```

> 输出结果为：
> 输出结果：
2018-02-23 22:03:18.475499+0800 YSC-GCD-demo[20470:5176805] apply---begin

>2018-02-23 22:03:18.476672+0800 YSC-GCD-demo[20470:5177035] 1---<NSThread: 0x60000027b8c0>{number = 3, name = (null)}

>2018-02-23 22:03:18.476693+0800 YSC-GCD-demo[20470:5176805] 0---<NSThread: 0x604000070640>{number = 1, name = main}

>2018-02-23 22:03:18.476704+0800 YSC-GCD-demo[20470:5177037] 2---<NSThread: 0x604000276800>{number = 4, name = (null)}

>2018-02-23 22:03:18.476735+0800 YSC-GCD-demo[20470:5177036] 3---<NSThread: 0x60000027b800>{number = 5, name = (null)}

>2018-02-23 22:03:18.476867+0800 YSC-GCD-demo[20470:5177035] 4---<NSThread: 0x60000027b8c0>{number = 3, name = (null)}

>2018-02-23 22:03:18.476867+0800 YSC-GCD-demo[20470:5176805] 5---<NSThread: 0x604000070640>{number = 1, name = main}

>2018-02-23 22:03:18.477038+0800 YSC-GCD-demo[20470:5176805] apply---end

从dispatch\_apply 相关代码执行结果中可以看出：

	0~5打印顺序不定，最后打印了 apply---end

因为是在并发队列中异步执行任务，所以各个任务执行的时间长短不定，最后结束顺序也不定，但是apply---end一定在最后执行，这是因为dispatch\_apply函数会等待全部任务执行完毕。


+ GCD的队列组：dispatch_group
当我们需要异步执行2个耗时任务，然后当2个耗时任务都执行完毕后再回到主线程执行任务，这时候我们可以用到GCD的队列组。

	+ 调用队列组的dispatch\_group\_async先把任务放到队列中，然后将队列放入队列组中，或者使用队列组的dispatch\_group\_enter、dispatch\_group\_leave组合来实现dispatch\_group\_async。
	
	+ 调用队列组的dispatch\_group\_notify 回到指定线程执行任务，或者使用dispatch\_group\_wait回到当前线程继续向下执行（会阻塞当前线程）

> + dispatch\_group\_notify
	
监听group中任务的完成状态，当所有的任务都执行完成后，追加任务到group中，并执行任务。

```
/**
 * 队列组 dispatch_group_notify
 */
- (void)groupNotify {
    NSLog(@"currentThread---%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"group---begin");
    
    dispatch_group_t group =  dispatch_group_create();
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 追加任务1
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        }
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 追加任务2
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"2---%@",[NSThread currentThread]);      // 打印当前线程
        }
    });
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 等前面的异步任务1、任务2都执行完毕后，回到主线程执行下边任务
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"3---%@",[NSThread currentThread]);      // 打印当前线程
        }
        NSLog(@"group---end");
    });
}

```

> 输出结果：
> 2018-02-23 22:05:03.790035+0800 YSC-GCD-demo[20494:5183349] currentThread---<NSThread: 0x604000072040>{number = 1, name = main}

> 2018-02-23 22:05:03.790237+0800 YSC-GCD-demo[20494:5183349] group---begin

> 2018-02-23 22:05:05.792721+0800 YSC-GCD-demo[20494:5183654] 1---<NSThread: 0x60000026f280>{number = 4, name = (null)}

> 2018-02-23 22:05:05.792725+0800 YSC-GCD-demo[20494:5183656] 2---<NSThread: 0x60000026f240>{number = 3, name = (null)}

> 2018-02-23 22:05:07.797408+0800 YSC-GCD-demo[20494:5183656] 2---<NSThread: 0x60000026f240>{number = 3, name = (null)}

> 2018-02-23 22:05:07.797408+0800 YSC-GCD-demo[20494:5183654] 1---<NSThread: 0x60000026f280>{number = 4, name = (null)}

> 2018-02-23 22:05:09.798717+0800 YSC-GCD-demo[20494:5183349] 3---<NSThread: 0x604000072040>{number = 1, name = main}

> 2018-02-23 22:05:11.799827+0800 YSC-GCD-demo[20494:5183349] 3---<NSThread: 0x604000072040>{number = 1, name = main}

> 2018-02-23 22:05:11.799977+0800 YSC-GCD-demo[20494:5183349] group---end

从dispatch\_group\_notify相关代码运行输出结果可以看出： 当所有任务都执行完成之后，才执行dispatch\_group\_notify block 中的任务。

+ dispatch\_group\_wait
	+ 暂停当前线程（阻塞当前线程），等待指定group中的任务执行完成后，才会往下继续执行。
	
```
/**
 * 队列组 dispatch_group_wait
 */
- (void)groupWait {
    NSLog(@"currentThread---%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"group---begin");
    
    dispatch_group_t group =  dispatch_group_create();
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 追加任务1
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        }
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 追加任务2
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"2---%@",[NSThread currentThread]);      // 打印当前线程
        }
    });
    
    // 等待上面的任务全部完成后，会往下继续执行（会阻塞当前线程）
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    
    NSLog(@"group---end");
}
```

> 输出结果:
> 
> 2018-02-23 22:10:16.939258+0800 YSC-GCD-demo[20538:5198871] currentThread---<NSThread: 0x600000066780>{number = 1, name = main}
> 
> 2018-02-23 22:10:16.939455+0800 YSC-GCD-demo[20538:5198871] group---begin
> 
> 2018-02-23 22:10:18.943862+0800 YSC-GCD-demo[20538:5199137] 2---<NSThread: 0x600000464b80>{number = 4, name = (null)}
> 
> 2018-02-23 22:10:18.943861+0800 YSC-GCD-demo[20538:5199138] 1---<NSThread: 0x604000076640>{number = 3, name = (null)}
> 
> 2018-02-23 22:10:20.947787+0800 YSC-GCD-demo[20538:5199137] 2---<NSThread: 0x600000464b80>{number = 4, name = (null)}
> 
> 2018-02-23 22:10:20.947790+0800 YSC-GCD-demo[20538:5199138] 1---<NSThread: 0x604000076640>{number = 3, name = (null)}
> 
> 2018-02-23 22:10:20.948134+0800 YSC-GCD-demo[20538:5198871] group---end

从dispatch\_group\_wait相关代码运行输出结果可以看出： 当所有任务执行完成之后，才执行 dispatch\_group\_wait 之后的操作。但是，使用dispatch\_group\_wait 会阻塞当前线程。

+ dispatch\_group\_enter、dispatch\_group\_leave
	+ dispatch\_group\_enter 标志着一个任务追加到 group，执行一次，相当于 group 中未执行完毕任务数+1
	+ dispatch\_group\_leave 标志着一个任务离开了 group，执行一次，相当于 group 中未执行完毕任务数-1。
	+ 当 group 中未执行完毕任务数为0的时候，才会使dispatch\_group\_wait解除阻塞，以及执行追加到dispatch\_group\_notify中的任务。

```
/**
 * 队列组 dispatch_group_enter、dispatch_group_leave
 */
- (void)groupEnterAndLeave
{
    NSLog(@"currentThread---%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"group---begin");
    
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_enter(group);
    dispatch_async(queue, ^{
        // 追加任务1
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        }
        dispatch_group_leave(group);
    });
    
    dispatch_group_enter(group);
    dispatch_async(queue, ^{
        // 追加任务2
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"2---%@",[NSThread currentThread]);      // 打印当前线程
        }
        dispatch_group_leave(group);
    });
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 等前面的异步操作都执行完毕后，回到主线程.
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"3---%@",[NSThread currentThread]);      // 打印当前线程
        }
        NSLog(@"group---end");
    });
    
//    // 等待上面的任务全部完成后，会往下继续执行（会阻塞当前线程）
//    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
//
//    NSLog(@"group---end");
}

```
> 输出结果:
> 
> 2018-02-23 22:14:17.997667+0800 YSC-GCD-demo[20592:5214830] currentThread---<NSThread: 0x604000066600>{number = 1, name = main}
> 
> 2018-02-23 22:14:17.997839+0800 YSC-GCD-demo[20592:5214830] group---begin
> 
> 2018-02-23 22:14:20.000298+0800 YSC-GCD-demo[20592:5215094] 1---<NSThread: 0x600000277c80>{number = 4, name = (null)}
> 
> 2018-02-23 22:14:20.000305+0800 YSC-GCD-demo[20592:5215095] 2---<NSThread: 0x600000277c40>{number = 3, name = (null)}
> 
> 2018-02-23 22:14:22.001323+0800 YSC-GCD-demo[20592:5215094] 1---<NSThread: 0x600000277c80>{number = 4, name = (null)}
> 
> 2018-02-23 22:14:22.001339+0800 YSC-GCD-demo[20592:5215095] 2---<NSThread: 0x600000277c40>{number = 3, name = (null)}
>
>2018-02-23 22:14:24.002321+0800 YSC-GCD-demo[20592:5214830] 3---<NSThread: 0x604000066600>{number = 1, name = main}
>
>2018-02-23 22:14:26.002852+0800 YSC-GCD-demo[20592:5214830] 3---<NSThread: 0x604000066600>{number = 1, name = main}
>
>2018-02-23 22:14:26.003116+0800 YSC-GCD-demo[20592:5214830] group---end

从dispatch\_group\_enter、dispatch\_group\_leave相关代码运行结果中可以看出：当所有任务执行完成之后，才执行 dispatch\_group\_notify 中的任务。这里的dispatch\_group\_enter、dispatch\_group\_leave组合，其实等同于dispatch\_group\_async。



+ GCD信号量：dispatch\_semaphore

GCD中的信号量是指Dispatch Semaphore,是持有计数的信号，类似于过高速路收费站的栏杆，可以通过时，打开栏杆，不可以通过时，关闭栏杆，在Dispatch Semaphore中使用计数来完成这个功能，计数为0时等待，不可通过，计数为1或大于1时，计数减1且不等待，可通过。Dispatch Semaphore 提供了三个函数:

> + dispatch\_semaphore\_create: 创建一个Semaphore并初始化信号的总量
> 
> + dispatch\_semaphore\_signal: 发送一个信号，让信号量加1
> 
> + dispatch\_semphore\_wait: 可以使总信号量减1，当信号量为0是会一直等待（阻塞所有线程），否则就可以正常执行

注意：信号量的使用前提是：想清楚你需要处理那个线程等待（阻塞），又要哪个线程继续执行，然后使用信号量。

Dispatch Semaphore 在实际开发中主要用于：

+ 保持线程同步，将异步执行任务转换为同步执行任务
+ 保证线程安全，为线程加锁

+ 使用Dispatch Semaphore实现线程同步

我们在开发中，会遇到这样的需求：异步执行耗时任务，并使用异步执行的结果进行一些额外的操作。换句话说，相当于，将异步执行的任务转换为同步执行任务，比如说：AFNetworing中AFURLSessionManager.m里面的tasksForKeyPath：方法,通过引入信号量的方式，等待异步执行任务结果，获取到tasks,然后在返回该tasks

```
- (NSArray *)tasksForKeyPath:(NSString *)keyPath {
    __block NSArray *tasks = nil;
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    [self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        if ([keyPath isEqualToString:NSStringFromSelector(@selector(dataTasks))]) {
            tasks = dataTasks;
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(uploadTasks))]) {
            tasks = uploadTasks;
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(downloadTasks))]) {
            tasks = downloadTasks;
        } else if ([keyPath isEqualToString:NSStringFromSelector(@selector(tasks))]) {
            tasks = [@[dataTasks, uploadTasks, downloadTasks] valueForKeyPath:@"@unionOfArrays.self"];
        }

        dispatch_semaphore_signal(semaphore);
    }];

    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

    return tasks;
}

```

我们来利用Dispatch Semaphore实现线程同步，将异步执行任务转换为同步执行任务。

```
/**
 * semaphore 线程同步
 */
- (void)semaphoreSync {
    
    NSLog(@"currentThread---%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"semaphore---begin");
    
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    
    __block int number = 0;
    dispatch_async(queue, ^{
        // 追加任务1
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        
        number = 100;
        
        dispatch_semaphore_signal(semaphore);
    });
    
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"semaphore---end,number = %zd",number);
}

```

> 输出结果为：
> 2018-02-23 22:22:26.521665+0800 YSC-GCD-demo[20642:5246341] currentThread---<NSThread: 0x60400006bc80>{number = 1, name = main}
>
>2018-02-23 22:22:26.521869+0800 YSC-GCD-demo[20642:5246341] semaphore---begin
>
>2018-02-23 22:22:28.526841+0800 YSC-GCD-demo[20642:5246638] 1---<NSThread: 0x600000272300>{number = 3, name = (null)}
>
>2018-02-23 22:22:28.527030+0800 YSC-GCD-demo[20642:5246341] semaphore---end,number = 100


从Dispatch Semaphore实现线程同步的代码可以看出:

> semaphore---end 是在执行完  number = 100; 之后才打印的。而且输出结果 number 为 100。
这是因为异步执行不会做任何等待，可以继续执行任务。异步执行将任务1追加到队列之后，不做等待，接着执行dispatch_semaphore_wait方法。此时 semaphore == 0，当前线程进入等待状态。然后，异步任务1开始执行。任务1执行到dispatch_semaphore_signal之后，总信号量，此时 semaphore == 1，dispatch_semaphore_wait方法使总信号量减1，正在被阻塞的线程（主线程）恢复继续执行。最后打印semaphore---end,number = 100。这样就实现了线程同步，将异步执行任务转换为同步执行任务。

+ Dispatch Semaphore 线程安全和线程同步（为线程加锁）

> + 线程安全：如果你的代码所在的线程中有多个线程在同时运行，而这些线程可能会同时运行这段代码，如果每次运行结果和单线程运行结果一样，而且其他的变量的值也和这个预期是一样的，那就是线程安全的。
> 
> 若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作（更改变量），一般都需要考虑线程同步，否则的话就可能影响线程安全。
> 
> + 线程同步：可理解为线程A 和 线程B 一块配合，A执行到一定程度的时候需要依靠B的某个结果，于是停下来，示意B运行；B执行完后，将结果返回给A；A在继续操作








	

