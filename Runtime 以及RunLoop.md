###### runtime概念
--
__Objective\-C__是基于C的，它为C添加了面向对象的特性，它将很多静态语言在编译和链接时期做的事情放到了__runtime__运行时来处理，可以说runtime使我们Objective\-C幕后工作者。

> + runtime（运行时），是一套纯C（C和汇编）的API，对于C语言，函数的调用在编译的时候就会决定调用哪个函数，OC的函数调用称为消息发送，属于动态调用过程，在编译的时候并不能决定真正调用哪个函数，只有真正运行的时候才会根据函数的名称找到对应的函数实现。事实证明：在编译阶段，OC可以__调用任何函数__，即使这个函数并未实现，只要声明过就不会报错，只有当运行的时候才会报错，这是因为OC是运行时动态调用的，而C语言__调用未实现的函数__就会报错。


###### runtime概念
--


###### runtime常见作用
--
+	动态交换两个方法的实现

+	动态添加属性

+	实现字典转模型的自动转换

+	发送消息

+	动态添加方法

+	拦截并替换方法

+	实现NSCoding的自动归档和解档

###### runtime交换方法的实现
> 应用场景：当第三方框架或者系统原生方法功能不能满足的时候，我们可以保持系统的原生功能的基础上，添加额外的功能。
>	> eg: 加载一张图片直接用[UIImage imageNamed:@"name"];是无法知道到底有没有加载成功，给系统的imageNamed添加额外的功能

>	> + 方案1：继承系统的类，重写方法
>	> + 方案2：使用runtime，交换方法，当使用runtime交换方法实现的时候，我们需要注意的是在调用方法之前，交换两个方法地址的指向，一般我们将方法交换的代码写在分类的load方法里，
>

###### runtime给分类动态添加属性
>	给一个类声明属性，其实本质就是给这个类添加关联，并不是直接把这个值得内存空间添加到类的内存空间。
>
>应用场景：当给系统的类添加属性的时候，可以使用runtime动态添加属性方法，
>
>eg：给系统NSObject添加一个分类，分类中是不能够添加成员属性的，虽然我们用了@property，但是仅仅会自动生成__get__和__set__方法的声明，并没有带下划线的属性和方法实现的生成，但是我们可以通过runtime就可以做到添加方法的实现。
>

```
@interface NSObject (Property)

// @property分类:只会生成get,set方法声明,不会生成实现,也不会生成下划线成员属性
@property NSString *name;
@property NSString *height;
@end

@implementation NSObject (Property)

- (void)setName:(NSString *)name {
    
    // objc_setAssociatedObject（将某个值跟某个对象关联起来，将某个值存储到某个对象中）
    // object:给哪个对象添加属性
    // key:属性名称
    // value:属性值
    // policy:保存策略
    objc_setAssociatedObject(self, @"name", name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (NSString *)name {
    return objc_getAssociatedObject(self, @"name");
}

// 调用
NSObject *objc = [[NSObject alloc] init];
objc.name = @"123";
NSLog(@"runtime动态添加属性name==%@",objc.name);

// 打印输出
runtime动态添加属性--name == 123
```
总结：其实，给属性赋值的本质，就是让属性与一个对象产生关联，所以要给NSObject的分类的name属性让name和NSObject产生关联，

--
###### 字典转模型

--

###### 动态添加方法
> 应用场景：如果一个类方法非常多，加载到内存的时候也比较耗费资源，需要给每个方法生成映射表，可以使用动态给某个类，添加方法解决。（消息转发）


##### 动态变量控制
> 动态改变类对象中成员变量的值。

--
##### 实现NSCoding的自动归档和解档
> 如果你实现过自定义模型数据持久化的过程，当一个模型有许多个属性，那么我们需要对每个属性都实现一遍encodeObject 和 decodeObjectForKey方法，如果这样的模型又有很多个，这其实是一个十分麻烦的事情。
> 


```
#import "Movie.h"
#import <objc/runtime.h>
@implementation Movie

- (void)encodeWithCoder:(NSCoder *)encoder

{
    unsigned int count = 0;
    Ivar *ivars = class_copyIvarList([Movie class], &count);

    for (int i = 0; i<count; i++) {
        // 取出i位置对应的成员变量
        Ivar ivar = ivars[i];
        // 查看成员变量
        const char *name = ivar_getName(ivar);
        // 归档
        NSString *key = [NSString stringWithUTF8String:name];
        id value = [self valueForKey:key];
        [encoder encodeObject:value forKey:key];
    }
    free(ivars);
}

- (id)initWithCoder:(NSCoder *)decoder
{
    if (self = [super init]) {
        unsigned int count = 0;
        Ivar *ivars = class_copyIvarList([Movie class], &count);
        for (int i = 0; i<count; i++) {
        // 取出i位置对应的成员变量
        Ivar ivar = ivars[i];
        // 查看成员变量
        const char *name = ivar\_getName(ivar);
       // 归档
       NSString *key = [NSString stringWithUTF8String:name];
      id value = [decoder decodeObjectForKey:key];
       // 设置到成员变量身上
        [self setValue:value forKey:key];

        }
        free(ivars);
    } 
    return self;
}
@end
```
> 这样的方式实现，不管有多少个属性，写这几行代码可以了。
> 也可以将方法抽出写成宏，

```
#import "Movie.h"
#import <objc/runtime.h>

#define encodeRuntime(A) \
\
unsigned int count = 0;\
Ivar *ivars = class_copyIvarList([A class], &count);\
for (int i = 0; i<count; i++) {\
Ivar ivar = ivars[i];\
const char *name = ivar_getName(ivar);\
NSString *key = [NSString stringWithUTF8String:name];\
id value = [self valueForKey:key];\
[encoder encodeObject:value forKey:key];\
}\
free(ivars);\
\

#define initCoderRuntime(A) \
\
if (self = [super init]) {\
unsigned int count = 0;\
Ivar *ivars = class_copyIvarList([A class], &count);\
for (int i = 0; i<count; i++) {\
Ivar ivar = ivars[i];\
const char *name = ivar_getName(ivar);\
NSString *key = [NSString stringWithUTF8String:name];\
id value = [decoder decodeObjectForKey:key];\
[self setValue:value forKey:key];\
}\
free(ivars);\
}\
return self;\
\

@implementation Movie

- (void)encodeWithCoder:(NSCoder *)encoder

{
    encodeRuntime(Movie)
}

- (id)initWithCoder:(NSCoder *)decoder
{
    initCoderRuntime(Movie)
}
@end
```

##### runtime下Class的各项操作：
+ 获取属性列表

__unsigned int count;__

```
objc_property_t *propertyList = class_copyPropertyList([self class], &count);
 for (unsigned int i=0; i<count; i++) {
     const char *propertyName = property_getName(propertyList[i]);
     NSLog(@"property---->%@", [NSString stringWithUTF8String:propertyName]);
 }
```

+ 获取方法列表

```
Method *methodList = class_copyMethodList([self class], &count);
 for (unsigned int i; i<count; i++) {
     Method method = methodList[i];
     NSLog(@"method---->%@", NSStringFromSelector(method_getName(method)));
 }
```

+ 获取协议列表

```
__unsafe_unretained Protocol **protocolList = class_copyProtocolList([self class], &count);
  for (unsigned int i; i<count; i++) {
      Protocol *myProtocal = protocolList[i];
      const char *protocolName = protocol_getName(myProtocal);
      NSLog(@"protocol---->%@", [NSString stringWithUTF8String:protocolName]);
  }
```

+ 获取成员变量列表

```
Ivar *ivarList = class_copyIvarList([self class], &count);
  for (unsigned int i; i<count; i++) {
      Ivar myIvar = ivarList[i];
      const char *ivarName = ivar_getName(myIvar);
      NSLog(@"Ivar---->%@", [NSString stringWithUTF8String:ivarName]);
  }
```

> 现有一个Person类，和person创建的xiaoming对象，有test1和test2两个方法
+ 获取类方法

```
Class PersonClass = object_getClass([Person class]);
SEL oriSEL = @selector(test1);
Method oriMethod = _class_getMethod(xiaomingClass, oriSEL);
```

+ 获取实例方法

```
Class PersonClass = object_getClass([xiaoming class]);
SEL oriSEL = @selector(test2);
Method cusMethod = class_getInstanceMethod(xiaomingClass, oriSEL);
```

+ 添加方法

```
BOOL addSucc = class_addMethod(xiaomingClass, oriSEL, method_getImplementation(cusMethod), method_getTypeEncoding(cusMethod));
```
+ 替换原方法的实现

```
class_replaceMethod(toolClass, cusSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
```
+ 交换两个方法的实现

```
method_exchangeImplementations(oriMethod, cusMethod);
```

```

// 得到类的所有方法
    Method *allMethods = class_copyMethodList([Person class], &count);
// 得到所有成员变量
    Ivar *allVariables = class_copyIvarList([Person class], &count);
// 得到所有属性
    objc_property_t *properties = class_copyPropertyList([Person class], &count);
// 根据名字得到类变量的Ivar指针，但是这个在OC中好像毫无意义
Ivar oneCVIvar = class_getClassVariable([Person class], name);
// 根据名字得到实例变量的Ivar指针
    Ivar oneIVIvar = class_getInstanceVariable([Person class], name);
// 找到后可以直接对私有变量赋值
    object_setIvar(_per, oneIVIvar, @"Mike");//强制修改name属性
/* 动态添加方法：
     第一个参数表示Class cls 类型；
     第二个参数表示待调用的方法名称；
     第三个参数(IMP)myAddingFunction，IMP是一个函数指针，这里表示指定具体实现方法myAddingFunction；
     第四个参数表方法的参数，0代表没有参数；
     */
    class_addMethod([_per class], @selector(sayHi), (IMP)myAddingFunction, 0);
// 交换两个方法
    method_exchangeImplementations(method1, method2);
// 关联两个对象
objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
/*
 id object                     :表示关联者，是一个对象，变量名理所当然也是object
 const void *key               :获取被关联者的索引key
 id value                      :被关联者，这里是一个block
 objc_AssociationPolicy policy : 关联时采用的协议，有assign，retain，copy等协议，一般使用OBJC_ASSOCIATION_RETAIN_NONATOMIC
*/
```

###### runtime相关参数的概念
1. __objc\_msgSend__

	> 这个是最基本的用于发送消息的函数。
	> 其实编译器会根据情况在objc\_msgSend,objc\_msgSend\_stret,objc_msgSendSuper,或objc\_msgSendSuper\_start四个方法中选择一个来调用。如果消息是传递给超类，那么会调用名字带有Super的函数，如果消息返回值是数据结构而不是简单值，那么会调用名字带有stret函数。

2. __SEL__
> objc\_msgSend函数第二个参数类型为SEL，它是selector在OBJC中的表示类型，selector是方法选择器，可以理解为区分方法的ID，而这个ID的数据结构是SEL：
> 
> __typedef struct objc\_selector *SEL;__
> 其实它就是个映射到方法的C字符串，你可以用Objc编译器命令@selector或者runtime系统的sel_registerName函数来获得一个SEL类型的方法选择器。


3. __id__
> objc\_msgSend第一个参数类型为id，大家对它都不陌生，它是一个指向类实例的指针：
> 
> __typedef struct objc\_object *id;__
> 
> __struct objc\_object { Class isa; };__
> 
> objc_object结构体包含一个isa指针，根据isa指针就可以顺藤摸瓜找到对象所属的类。

4. runtime.h里Class的定义

```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;//每个Class都有一个isa指针
    
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;//父类
    const char *name                                         OBJC2_UNAVAILABLE;//类名
    long version                                             OBJC2_UNAVAILABLE;//类版本
    long info                                                OBJC2_UNAVAILABLE;//!*!供运行期使用的一些位标识。如：CLS_CLASS (0x1L)表示该类为普通class; CLS_META(0x2L)表示该类为metaclass等(runtime.h中有详细列出)
    long instance_size                                       OBJC2_UNAVAILABLE;//实例大小
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;//存储每个实例变量的内存地址
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;//!*!根据info的信息确定是类还是实例，运行什么函数方法等
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;//缓存
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;//协议
#endif
    
} OBJC2_UNAVAILABLE;
```
> 可以看到运行时一个类还关联了它的超类指针，类名，成员变量，方法，缓存，还有附属的协议。
> 
> 在objc\_class结构体中：methodLists是指向objc\_method\_list指针的指针，也就是说可以动态修改*methodLists的值来添加成员方法，这也是Category实现的原理。
> 

###### 交换方法实现的几种实现方式
+ 利用 method_exchangeImplementations 交换两个方法的实现

+ 利用 class_replaceMethod 替换方法的实现

+ 利用 method_setImplementation 来直接设置某个方法的IMP。

eg:

```
@implementation Son : NSObject
- (id)init
{
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```
答案：都输出Son，

>	+ super 仅仅是一个编译器指示器，只是告诉编译器调用这个方法时，去调用分类的方法，但是消息的接收者还是self。


| 作者 | Runtime模块推荐阅读博文|
|-----|-----|
|[西木]()|[完整总结]( http://www.jianshu.com/p/6b905584f536)|
||[http://www.jianshu.com/p/9e1bc8d890f9](http://www.jianshu.com/p/9e1bc8d890f9)|
||[ http://www.jianshu.com/p/46dd81402f63]( http://www.jianshu.com/p/46dd81402f63)|
||消息机制 [http://www.jianshu.com/p/f6300eb3ec3d](http://www.jianshu.com/p/f6300eb3ec3d)|
||Runtime在实际开发中的应用 [http://www.jianshu.com/p/851b21870d91](http://www.jianshu.com/p/851b21870d91)|

--
###### 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？
> 不能向编译后得到的类中增加实例变量，但是可以向运行时创建的类中添加实例变量
> 
> 因为编译后的类已经注册在runtime中，类结构体中的objc\_ivar\_list实例变量的链表和instance\_size实例变量的内存大小已经确定，runtime会调用class_\setvarlayout 或 class\_setWeekIvarLayout来处理strong weak引用，所以不能向存在的类中添加实例变量。
> 
> 运行时创建的类是可以添加实例变量，调用class\_addIvar函数，但是这个函数调用必须在调用objc\_allocateClassPair之后，objc\_registerClassPair之前，原因同上
> 

###### runtime如何实现weak变量自动置为nil？
> runtime对注册的类，会进行布局，对于weak对象会放入一个hash表中，用weak指向的对象内存地址作为key，当此对象的引用计数为0的时候会dealloc，假如weak指向的对象内存地址是a，那么就会以a为键，在这个weak表中搜索，找到所有以a为键的weak对象，从而置为nil；


--

# 2. runloop
![](/Users/luleios/Desktop/1519808823434.jpg)

--
+ Runloops 是线程相关底层基础的一部分，它的本质和字面意思一样运行着的循环（事件处理的循环），作用：接受循环事件和安排线程的工作，目的：让线程在有任务的时候忙于工作，而没任务的时候让线程处于休眠状态。

> + 保持程序的持续运行（如：程序一启动就会开启一个主线程（主线程中的runloop是自动创建并运行），runloop保证主线程不会被销毁、也就保证了程序的持续运行）
> 
> +	处理App中的各种事件（如：touches触摸事件、NSTimer定时器事件、Selector事件（选择器performSelector））
> 
> + 节省CPU资源，提高程序性能
> 
> +	 负责渲染屏幕上所有UI


+ Runloops 的管理并非完全自动，你仍然需要设置线程代码在合适的时候启动runloop来帮助你处理输入事件，iOS中Cocoa和CoreFoundation框架中各有一套完整的关于runloop对象的操作api，在主线程中runloop是自动创建并运行的（子线程需要手动开启）

###### 附：CFRunLoop.c源码

```
#【用DefaultMode启动，具体实现查看 CFRunLoopRunSpecific Line2704】
#【RunLoop的主函数，是一个死循环 dowhile】
void CFRunLoopRun(void) {	/* DOES CALLOUT */
    int32_t result;
    do {
        /*
         参数一：CFRunLoopRunSpecific   具体处理runloop的运行情况
         参数二：CFRunLoopGetCurrent()  当前runloop对象
         参数三：kCFRunLoopDefaultMode  runloop的运行模式的名称
         参数四：1.0e10                 runloop默认的运行时间，即超时为10的九次方
         参数五：returnAfterSourceHandled 回调处理
         */
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
        
        //【判断】：如果runloop没有停止 且 没有结束则继续循环，相反侧退出。
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}
#【直观表现】
RunLoop 其实内部就是do-while循环，在这个循环内部不断地处理各种任务（`比如Source、Timer、Observer`），
通过判断result的值实现的。所以 可以看成是一个死循环。
如果没有RunLoop，UIApplicationMain 函数执行完毕之后将直接返回，就是说程序一启动然后就结束；
```

###### 创建runloop

```
# NOTE: 获得runloop实现 (创建runloop)
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    if (pthread_equal(t, kNilPthreadT)) {// ✔️【主线程相关联的RunLoop创建】,如果为空，默认是主线程
        t = pthread_main_thread_np();
    }
    __CFLock(&loopsLock);
    if (!__CFRunLoops) { // 如果 RunLoop 不存在
        __CFUnlock(&loopsLock);
        // 创建字典
        CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        // 创建主线程对应的runloop
        CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
        // 使用字典保存（KEY:线程 -- Value:线程对应的runloop）, 以保证一一对应关系。
        CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
        if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
            CFRelease(dict);
        }
        CFRelease(mainLoop);
        __CFLock(&loopsLock);
    }
    
    // ✔️【创建与子线程相关联的RunLoop】,从字典中获取 子线程的runloop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFUnlock(&loopsLock);
    if (!loop) {
        // 如果子线程的runloop不存在,那么就为该线程创建一个对应的runloop
        CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFLock(&loopsLock);
        loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
        // 把当前子线程和对应的runloop保存到字典中
        if (!loop) {
            CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
            loop = newLoop;
        }
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFUnlock(&loopsLock);
        CFRelease(newLoop);
    }
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```
： 有以上源码可得：

	> + 每条线程都有唯一的一个与之对应的RunLoop对象。
	> + 主线程的RunLoop已经自动创建，子线程的RunLoop需要主动创建。
	> + RunLoop对象是利用字典来进行存储，而且Key：线程-Value：线程对应的runLoop。
￼￼
###### 如何创建子线程对应的RunLoop：
+ 子线程创建runloop，不是通过[alloc init]方法创建，而是直接通过调用currentRunLoop方法来创建。

	> + currentRunLoop本身是懒加载的，当第一次调用currentRunLoop方法获得该子线程对应的RunLoop的时候，它会先去判断这个线程的runloop是否存在，如果不存在就会自己创建并且返回，如果存在直接返回。
	
RunLoop内部实现流程：
![](http://upload-images.jianshu.io/upload_images/2230763-013c96b25cc33a2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### runloop相关类
###### Core Foundation中关于RunLoop的5个类
	+ 	CFRunloopRef【RunLoop本身】
	+ 	CFRunloopModeRef【Runloop的运行模式】
	+ 	CFRunloopSourceRef【Runloop要处理的事件源】
	+  CFRunloopTimerRef【Timer事件】
	+  CFRunloopObserverRef【Runloop的观察者（监听者）】
###### CFRunloop的5个相关类图解：

![](http://upload-images.jianshu.io/upload_images/2230763-b082dfb38ca44c87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 图解直观得知：
+ 一条线程对应一个Runloop，Runloop总是运行在某种特定的CFRunLoopModeRef（运行模式）下。
+ 每个RunLoop都可以包含若干个Mode，每个Mode又包含Source源/Timer事件/Observer观察者。
+ 在RunLoop中有多个运行模式，如果需要切换Mode，只能是退出CurrentMode切换到指定的Mode进入，目的是保证不同Mode下的Source/Timer/Observer互不影响。
+ Runloop有效，mode里面至少要有一个timer（定时器事件）或者Source（源）。

######源码：
```
struct __CFRunLoop {
     CFRuntimeBase _base;
     pthread_mutex_t _lock;          /* locked for accessing mode list */
     __CFPort _wakeUpPort;           // used for CFRunLoopWakeUp 
     Boolean _unused;
     volatile _per_run_data *_perRunData;              // reset for runs of the run loop
     pthread_t _pthread;
     uint32_t _winthread;
     CFMutableSetRef _commonModes;
     CFMutableSetRef _commonModeItems;
     CFRunLoopModeRef _currentMode;
     CFMutableSetRef _modes;
     struct _block_item *_blocks_head;
     struct _block_item *_blocks_tail;
     CFAbsoluteTime _runTime;
     CFAbsoluteTime _sleepTime;
     CFTypeRef _counterpart;
 };
 struct __CFRunLoopMode {
     CFRuntimeBase _base;
     pthread_mutex_t _lock;  /* must have the run loop locked before locking this */
     CFStringRef _name; // mode名
     Boolean _stopped;
     char _padding[3];
     CFMutableSetRef _sources0; // source0 源
     CFMutableSetRef _sources1; // source1 源
     CFMutableArrayRef _observers; // observer 源
     CFMutableArrayRef _timers; // timer 源
     CFMutableDictionaryRef _portToV1SourceMap;// mach port 到 mode的映射,为了在runloop主逻辑中过滤runloop自己的port消息。
     __CFPortSet _portSet;// 记录了所有当前mode中需要监听的port，作为调用监听消息函数的参数。
     CFIndex _observerMask;
 #if USE_DISPATCH_SOURCE_FOR_TIMERS
     dispatch_source_t _timerSource;
     dispatch_queue_t _queue;
     Boolean _timerFired; // set to true by the source when a timer has fired
     Boolean _dispatchTimerArmed;
 #endif
 #if USE_MK_TIMER_TOO
     mach_port_t _timerPort;// 使用 mk timer， 用到的mach port，和source1类似，都依赖于mach port
     Boolean _mkTimerArmed;
 #endif
 #if DEPLOYMENT_TARGET_WINDOWS
     DWORD _msgQMask;
     void (*_msgPump)(void);
 #endif
     uint64_t _timerSoftDeadline; /* TSR timer触发的理想时间*/
     uint64_t _timerHardDeadline; /* TSR timer触发的实际时间，理想时间加上tolerance（偏差*/
 };
```

###### CFRunloopModeRef 代表RunLoop的运行模式；系统默认提供了5个Mode
+ kCFRunloopDefaultMode（NSDefaultRunLoopMode）：App的默认Mode，通常主线程是在这个Mode下运行

+ UITrackingRunLoopMode：界面跟踪Mode，用于ScrollView追踪触摸滑动，保证界面滑动时不受其它Mode影响。

+ UIInitiationRunLoopMode：接受系统事件的内部Mode，通常用不到。

+ GSEventReceiveRunLoopMode：接受系统事件的内部Mode，通常用不到。

+ kCFRunLoopCommonModes（NSRunLoopCommonModes）：这个并不是某种具体的Mode，可以说是一个占位的Mode（一种模式组合）

+ CFRunloop对外暴露的Mode接口：

```
# CFRunLoop
CF_EXPORT void CFRunLoopAddCommonMode(CFRunLoopRef rl, CFRunLoopMode mode);
CF_EXPORT CFRunLoopRunResult CFRunLoopRunInMode(CFRunLoopMode mode, CFTimeInterval seconds, Boolean returnAfterSourceHandled);
# NSRunLoop.h
FOUNDATION_EXPORT NSRunLoopMode const NSDefaultRunLoopMode;// (默认):同一时间只能执行一个任务
FOUNDATION_EXPORT NSRunLoopMode const NSRunLoopCommonModes NS_AVAILABLE(10_5, 2_0); // (公用):可以分配一定的时间处理定时器
```

> 对照上面的源码：关于CommonModes：
	
> 【关于 \_commonModes】：一个mode可以标记为common属性（用于CFRunLoopAddComon），然后它就会保存在\_commonModes,主线程CommonModes默认已有两个mode，CFRunLoopDefaultMode和UITrackingRunLoopMode，当然你也可以通过CFRunLoopCommonMode（）方法将自定义的mode放到KCFRunLoopCommonModes组合。
> 
> 【关于_commonModeItems】：_commonModeItems里面存放的source，observer，timer等，在每次runloop运行的时候都会被同步到具有Common标记的Modes里，如：
```
[[NSRunLoop currentRunLoop] addTimer:_timer forMode:NSRunLoopCommonModes];
就是把timer放到commonModeItems 里。
```
[更多关于RunLoop的资料](http://iphonedevwiki.net/index.php/CFRunLoop)

###### CFRunloopSourceRef 事件源 \ 输入源，有两种分类模式
![](/Users/luleios/Desktop/runloop.jpg)
> 输入源以异步的方式向您的线程传递事件，事件的来源取决于输入源的类型，它通常是两类中的一类，基于端口的输入源监视你的应用程序的Mach端口（内核端口），自定义输入源监视自定义事件，就您的运行循环而言，系统通常会实现两种类型的输入源，您可以原样使用它们，两个来源之间唯一的区别是它们如何发出信号，基于端口的源由内核自动发送信号，自定义源必须从另一个线程手动发送信号。
> 
> 按照函数调用栈的分类source0 和 source1
> 
> 	+ Source0：非基于端口Port事件；（用于用户主动触发事件，如：点击按钮或点击屏幕）
> 	+ Source1：基于端口Port的事件；（通过内核和其他线程相互发送消息，与内核相关）
> 	+ Source1事件在处理时会分发一些操作给Source0处理。


###### Timer
> CFRunLoopTimerRef是基于时间的触发器
> 
> 基本上说的就是NSTimer（CADisplayLink也是加到RunLoop），它受RunLoop的影响。
> 
> 而与NSTimer相比，GCD定时器不会受到RunLoop的影响。
> 
> 定时器源在未来的预设时间将事件同步传递给您的线程，定时器是线程通知自己做事的一种方式，例如，搜索字段可以使用定时器在用户点击的连续击键之间经过一段时间后启动自动搜索，
> 
> 虽然它生成基于时间的通知，但计时器不是实时机制，与输入源一样，定时器与运行循环的特定模式相关联，如果定时器未处于运行循环当前正在监视的模式下，则只有在定时器支持的其中一种模式下运行循环时才会触发定时器，同样，如果在运行循环处理执行处理程序（handle router）时触发定时器，则定时器将等待下一次通过运行循环来调用其处理程序例程，如果运行循环根本没有运行，则定时器不会启动。
> 

###### Observer
> 相对来说CFRunLoopObserver理解并不复杂，它相当于消息循环中的一个监听器，随时通知外部当前RunLoop的运行状态（它包含一个函数指针_callout_将当前状态及时告诉观察者），具体的Observer状态如下：
> 
```
/* jianshu:白开水ln Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),           //即将进入Runloop
    kCFRunLoopBeforeTimers = (1UL << 1),    //即将处理NSTimer
    kCFRunLoopBeforeSources = (1UL << 2),   //即将处理Sources
    kCFRunLoopBeforeWaiting = (1UL << 5),   //即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),    //从休眠装填中唤醒
    kCFRunLoopExit = (1UL << 7),            //退出runloop
    kCFRunLoopAllActivities = 0x0FFFFFFFU   //所有状态改变
};
```

###### RunLoop休眠
>	其实对于Event Loop而言，RunLoop最核心的事情就是保证线程在没有消息时休眠以避免占用系统资源，有消息时能够及时唤醒，RunLoop的这个机制完全依靠系统内核来完成，具体来说就是苹果操作系统核心组件Darwin中的Mach来完成

![](/Users/luleios/Desktop/2230763-ad034c393cfe948a.png)

GCD定时器的优点：

1. 与NSTimer相比，GCD定时器不会受RunLoop影响。
2. GCD定时器是绝对准确的

```
    /**
     创建GCD中的定时器

     @param DISPATCH_SOURCE_TYPE_TIMER source类型，DISPATCH_SOURCE_TYPE_TIMER表示是定时器
     @param 0 描述信息，线程ID
     @param 0 更详细的描述信息
     @param dispatchQueue ：队列，决定GCD定时器中的任务在哪个线程执行
     @return 返回GCD定时器对象
     */
    dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, <#dispatchQueue#>);
    
    // 设置定时器（起始时间，时间间隔，精准度）
    /**
    timer：定时器对象
    DISPATCH_TIME_NOW：起始时间，从现在开始
    intervalInSeconds：时间间隔，GCD中时间单位为纳秒
    leewayInSeconds：精准度，绝对精准0
    *
    */
    dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, <#intervalInSeconds#> * NSEC_PER_SEC, <#leewayInSeconds#> * NSEC_PER_SEC);
    /**
    设置定时器执行的任务
    */
    dispatch_source_set_event_handler(timer, ^{
        <#code to be executed when timer fires#>
    });
    // 启动执行
    dispatch_resume(timer);
```

--
RunLoop 应用、场景

1. NSTimer
2. ImageView显示：控制方法在特定的模式下可用
3. PerformSelector
4. 常驻线程：在子线程中开启一个RunLoop
5. AutoReleasePool自动释放池
6. UI更新

--

> AutoReleasePool 自动释放池

> : AutoAutoReleasePool 创建：
> >（1）有多种创建AutoReleasePool的方法，
> >
	> 在线程创建的时候，系统会自动创建。
	> 在MRC环境下，可以通过NSAutoreleasePool手动创建。
	> 通过@autoreleasepool创建
> > (2) 自动释放池的销毁
> > 
	 > 在每一次runloop结束时，系统会将栈顶的pool销毁，
	 > 在MRC环境下，release pool时，autoreleasepool会被销毁
	 
>> * 如果一个变量在自动释放池之外创建，如下，需要通过__autoreleaseing修饰符将其加入到自动释放池。
>> 
>> autoreleasepool的作用：系统通过@autoreleasepool{}这种方式来为我们创建自动释放池的，一个线程对应一个runloop，系统会为每一个runloop隐式的创建一个自动释放池，所有的autoreleasepool构成一个栈式结构，在每个runloop结束时，当前栈顶的autoreleasepool会被销毁，会对其中的每一个对象做一次release（严格来说，是对对象进行了几次autorelease就会做几次release，不一定是一次），
>> 
>> 特别需要注意：使用容器的block版本的枚举器的时候，系统会自动添加一个autoreleasePool

```
[array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
// 这里被一个局部@autoreleasepool包围着
}];
```

>> 
>> 
>> 自动释放池的应用场景:  

```
 MRC：
 	+ 对象作为函数返回值:
 	当一个对象要作为函数返回值得时候，因为要遵循谁申请谁释放的原则，
 	所以应该在返回之前释放，但要是返回之前释放了，就会造成野指针错
 	误，为了解决这个问题，可以使用autoreleasepool延迟释放的特性，
 	将其在返回之前做一次autorelease，将其加入到自动释放池中，这样就
 	可以保证正常返回，同时也可以做到在一次runloop之后，系统会自动帮我们释放他。
 	
 	+ 临时生成大量对象，一定要将自动释放池放在for循环里面，要释放在外
 	面，就会因为大量对象得不到及时释放，而造成内存紧张，最后程序意外退出
```
>> 
>> 
：AutoreleasePool是另一个与RunLoop相关讨论较多的话题，其实从RunLoop源代码分析，AutoreleasePool与RunLoop并没有直接的关系，之所以将两个话题放到一起讨论最主要的原因是因为iOS应用启动后会注册两个Observer管理和维护AutoreleasePool，可以在程序刚刚启动的时候打印currentRunloop可以看到系统默认注册了很多Observer，其中两个Observer的callout都是_wrapRunLoopWithAutoreleasePoolHandler，这两个和自动释放池相关的两个监听。

+ 第一个Observer会监听RunLoop的进入，它会回调objc\_autoreleasePoolPush()向当前的AutoreleasePoolPage增加一个哨兵对象标志创建自动释放池，这个Observer的order是-2147483647优先级最高，确保发生在所有回调操作之前。
+ 第二个Observer会监听RunLoop的进入休眠和即将退出RunLoop两种状态，在即将进入休眠时会调用objc\_autoreleasePoolPop（）和objc\_autoreleasePoolPush()根据情况从最新加入的对象一直往前清理到遇到哨兵对象，而在即将退出RunLoop时会调用objc\_autoreleasePoolPop（）释放自动释放池内对象。这个observer的order是2147483647，优先级最低，确保发生在所有回调操作之后。
+ 主线程的其他操作通常均在这个AutoReleasePool之内，以尽可能减少内存维护操作

###### UI更新
> 在应用程序启动之后，主线程RunLoop会默认注册一个callout为___ZN2CA11Transaction17observer\_callbackEP19\_\_CFRunLoopObservermPv__的Observer，这个

