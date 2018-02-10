+ 解释一下代码的运行结果

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        NSLog(@"before");
        [self performSelector:@selector(performLog) withObject:self afterDelay:0];
        NSLog(@"after");
    });
}

- (void)performLog
{
    NSLog(@"performLog");
}
```
上面的代码只会打印before和after，不会打印performLog，
原因：
> GCD默认的是全局并发队列，在并发执行任务的时候，会从线程池获取可执行任务的线程（如果没有就会阻塞线程）
> performSelector的原理是设置一个Timer到当前线程的Runloop，并且是NSDefaultRunLoopMode;
> 非主线程的Runloop默认是不开启的

+ performSelector(onMainThread aSelector: Selector, with arg: Any?, waitUntilDone wait: Bool)

>该方法的作用是将对象要执行的函数切换到主线程中执行。该方法没有返回值，最多只能带一个参数
>
> Bool值得设定 ,决定了当前要执行的函数是否需要阻塞当前线程，如果为YES，这回阻塞当前线程直到主线程的方法执行完，如果为NO，则不会阻塞当前线程。（Tips：如果当前线程为主线程，该Bool值无效，不然可能造成线程等待引发Crash）。
> 