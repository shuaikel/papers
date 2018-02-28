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

