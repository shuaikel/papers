
+ 分类和扩展有什么区别？可以分别用来做什么？分类有哪些局限性？分类的结构体里面有哪些成员？

```
	1.分类有分类名，而类扩展没有类名
	2.分类可以单独文件创建，而类扩展必须写在类文件中。
	3.分类中添加属性只能生成属性的声明，并不会生成对应的访问方法和成员变量，但类扩展可以
	
	
	分类的结构体类主要有：name（分类名）、cls（分类所属的类）、method_list（方法列表）、property_list（属性列表）、protocol_list(协议列表)
```

+ 讲一下atomic的实现机制；为什么不能保证绝对的线程安全（最好可以结合场景来说）？

```
实现机制是：为对象属性的setter、getter方法加锁，
比如多个线程对属性进行访问，其中一个线程对对象进行了setter操作，使得对象的引用计数加一，在这个过程中有一个线程对被访问对象进行了release操作，此时对象的引用计数为0，对象被释放，
还有比如数组的非getter、setter方法，也可能导致线程取到的数据为脏数据。
```

+ 被weak修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable么？里面的结构可以画出来么？

```
weak修饰的对象被释放时指向对象的指针也会被置为nil，可以避免野指针错误。
	
实现：当一个对象被weak修饰时，系统会以对象的地址为key，将对象加入由系统维护的CFMutableDictionary中，当对象的引用计数为0时，就会去这个全局的字典里面，将weak指针置为nil；
具体的实现： Firday QA 上介绍了一种类似KVO的实现方式，当对象存在weak指针时，我们可以将这个实例指向一个新创建的子类，然后修改这个子类的release方法，在release方法中，去从全局的CFMutableDictionary字典中找到所有的weak对象（这里应该是找到弱引用本对象的地方，将其置为nil）

Class subclass = objc_allocateClassPair(class, newNameC, 0); 

Method release = class_getInstanceMethod(class, @selector(release)); 

Method dealloc = class_getInstanceMethod(class, @selector(dealloc)); 

class_addMethod(subclass, @selector(release), 
(IMP)CustomSubclassRelease, method_getTypeEncoding(release)); 

class_addMethod(subclass, @selector(dealloc), (IMP)CustomSubclassDealloc, method_getTypeEncoding(dealloc)); 

objc_registerClassPair(subclass);


#### sideTable
```

+ 关联对象有什么应用，系统如何管理关联对象？其被释放的时候需要手动将其指针置空么？

```
关联对象可以动态的给类添加成员变量，系统通过Map来管理关联对象，在被释放的时候不需要
手动将指针置为nil。

```

+ KVO的底层实现？如何取消系统默认的KVO并手动触发（给KVO的触发设定条件：改变的值符合某个条件时再触发KVO）？

```
KVO	的底层实现是在注册监听时，系统会动态创建一个继承被监听类的
XX_NSNotification_class,重写类的class方法，返回被观察类，重写被观察属性的set
方法，在set方法里触发KVO的监听的方法，

[XXXX willChangeValueForkey:""];
[XXXX didChangeValueForKey:""];

```

+ Autoreleasepool所使用的数据结构是什么？AutoreleasePoolPage结构体了解么？

```
Autoreleasepool所使用的数据结构是__AutoreleasePool
AutoreleasePoolPage结构体双向列表，里面的成员有：
1. page Size 默认一页的大小是4096字节，
2. magic（用来数据校验），
3. id *next；（栈顶地址）、
4. pthread_t const trhread;(所在的线程)、
5. AutoreleasePoolPage *parent；（指向父页）、
6. Autoreleasepool *child；（指向子页）
```

+ 讲一下对象，类对象，元类，跟元类结构体的组成以及他们是如何相关联的？为什么对象方法没有保存的对象结构体里，而是保存在类对象的结构体里？

```
对象的结构体组成：isa 指针
类对象的结构体组成：{
	isa_t isa；
	Class superclass
	
	cache_t cache； // 方法调用缓存列表
	class_rw_t; // 方法列表
}

因为一个类的类对象在内存中只存在一份。将对象的方法保存在类对象的结构体中，可以避免初始化很多个对象的时候导致内存问题。

```

+ class\_ro\_t 和  class\_rw\_t 的区别？

```
class_rw_t是可读可写的，而class_ro_t可读不可写。
最初类的方法、属性、成员变量属性协议等等都是存放在class_ro_t中的，当程序运行的时候
需要将分类中的列表跟类初始的列表合并在一起时，就会将class_ro_t中的列表和分类中的列表合并起来存储在class_rw_t中，也就是说class_rw_t中有部分列表是从class_ro_t里面拿出来的，并最终和分类的方法合并。可以通过源码realizeClass查看到。通过源码我们可以发现，类的初始信息本来是存储在class_ro_t中的，并且ro本来是指向cls->data()的，也就是说bits.data()得到的ro，但是在运行过程中创建了class_rw_t，并将cls->data指向了rw,同时将初始信息ro赋值给rw中的ro，最后在通过setData（rw）设置data，那么此时bits.data()得到的就是rw，之后再去检查分类，同时将分类中的方法，属性，协议列表存储在class_rw_t的方法、属性及协议列表中。


<!--class_ro_t 是class_rw_t的结构体成员变量， class_ro_t 存储对象的成员变量、以及
成员变量的size。类名。 class_rw_t 包含 class_ro_t类型成员，方法列表，协议列
表，属性列表-->
```

+ class_rw_t 是如何存储方法的

```
我们知道method_array_t、property_array_t、protocol_array_t中以method_array_t为例，method_array_t中最终存储的是method_t,method_t是对方法、函数的封装，每一个方法对象就是一个method_t，

method_t 结构体定义：
struct method_t{
	SEL name; //函数名
	const char *types; // 编码 （返回值类型，参数类型）
	IMP imp; // 指向函数的指针
}

method_t结构体中有三个成员变量，
SEL： 代表方法、函数名，一般叫做选择器，底层结构跟char*类似typedef struct
objc_selector *SEL; 可以把SEL看做是方法名字符串。SEL可以通过@selector()和sel_registerName()获得，也可以通过sel_getName()和NSStringFromSelector()将SEL	转成字符串。不同类中相同名字的方法，所对应的方法选择器是相同的，
SEL仅仅代表方法的名字，并且不同类中相同的方法名的SEL是全局唯一的。

types：包含了函数返回值，参数编码的字符串，通过字符串拼接的方式将返回值和参数拼接成一个字符串，来代表函数返回值及参数。我们可以通过代码查看一下types是如何代表函数返回值及参数的，通过强制类型转换，类型转换编码


IMP：代表函数的具体实现，存储的内容是函数地址，也就是说当找到imp的时候就可以找到函数实现，进而对函数进行调用。当多次继承的子类想要调用基类的方法时，就需要通过superclass指针一层一层找到基类，在基类方法列表中找到对应的方法进行调用，如果多次调用基类方法，就需要多次遍历每一层父类的方法列表，这是非常消耗性能的，apple通过方法缓存的形式解决了这一问题

在类对象结构体中，成员变量cache就是用来对方法进行缓存的。
cache_t cache； 用来缓存曾经调用过的方法，可以提高方法的查找速度。
每当调用这个方法的时候，会先去cache中查找是否有缓存的方法，如果没有缓存，在去类对象方法列表中查找，以此类推直到找到方法之后，就会将方法直接存储在cache中，下一次在调用这个方法的时候，就会在类对象的cache中找到这个方法，直接调用。

cache_t如何进行缓存：
struct cache_t{
	struct bucket_t * _buckets; // 散列表
	mask_t _mask; // 散列表的长度 -1
	mask_t _occupoied; // 已经缓存的方法数量
}

bucket_t 是以数组的方式存储方法列表，看一下bucket_t内部结构
struct bucket_t {
	private:
		cache_key_t key;// SEL作为key
		IMP _imp;   // 函数的内存地址
}

从源码中可以看到bucket_t中存储这SEL和_imp，通过key->value的形式

```

+ iskindOfClass：class 与 isMemberOfClass：class的区别

```
iskindOfClass 是检查对象是否是那个类或者其继承类实例化的对象
isMemberOfClass 是检查对象是否是那个类但不包括继承类实例化的对象。
```

+ ios中动态解析方法流程图：

![](https://user-gold-cdn.xitu.io/2018/7/2/16456e432e5b2266?imageslim)

+ ios 消息转发流程图

```
当通过对象的isa指针找不到方法的实现，而又没有对方法进行动态解析的时候，会进行消息转发，会调用_objc_msgForward_impcache函数，其内部是实现也是根据函数返回的对象调用objc_msgSend，重新走一遍消息发送，动态解析，消息转发的过程，最终找到方法进行调用。如果消息转发函数forwardingTargetForSelector方法返回nil或者没有实现的话，就会调用methodSignatureForSelector方法，用来返回一个方法签名，这也是最后正确跳转方法的机会。

[如果methodSignatureForSelector方法返回正确的方法签名就会调用
forwardInvocation方法，forwardInvocation方法内部提供一个
NSInvocation类型的参数，NSIvocation封装了一个方法的调用，包括方法的
调用者，方法名，以及方法的参数，在forwardInvocation函数内修改方法调
用对象即可]

[如果methodSignatureForSelector返回的为nil，就会来到doseNotRecognizeSelector：]方法内部，程序crash提示无法识别选择器。

NSInvocation : methodSignatureForSelector 方法中返回的方法签名，在forwardInvocation中被包装成NSInvocation对象，NSInvocation提供了获取和修改方法名、参数、返回值等方法，也就是说在forwardInvocation函数中我们可以对方法进行最后的修改。

```
##### 消息转发流程图
![](https://user-gold-cdn.xitu.io/2018/7/2/16456e432f0a99c7?imageslim)




+ iOS 中内省的几个方法？class 和 objc_getClass方法有什么区别?

```

ios中内省的方法有很多： 可以有效的避免错误的进行消息转发，错误的假设对象相等问题，

+ iskindOfClass：class
+ isMemberOfClass：class
+ respondToSelector：selector
+ conformsToProtocol：protocol
+ isEqual：方法首先会检查指针的等同性，然后是类的等同性，最后调用对象的比较器进行比较，


对于用户自定义的类 class ，objc_getClass 方法返回的是类对象（也就是类本身），
而对于系统类，[obj class] 和 objc_getClass(obj) 由于类簇的存在，方法可能返回的对象不一致，但class 和 object_getClass(obj) 返回的结果一致。

```

+ 在运行时创建类的方法objc_allocateClassPair的方法名尾部为什么是pair（成对的意思）？

```
objc_allocateClassPair 方法名后面是pair的原因是，需要同时创建类和元类
（class、meta-class）
```

+ 一个int变量被__block修饰与否的区别？

```
变量一般都存储在堆区，但当变量被__block修饰时，系统会将变量的存储方式由堆区转为栈
区。
```

+ 为什么在block外部使用__weak修饰的同时需要在内部使用__strong修饰？
RunLoop的作用是什么？它的内部工作机制了解么？（最好结合线程和内存管理来说）


```
用__weak修饰的对象需要在内部__strong修饰的原因是为了防止，在block执行的过程中，对象被提前释放的情况。

Runloop的作用是 keep your thread busy where there is work to do and put your thread to sleep when there is none。

+ 保持程序的持续运行，程序启动的时候主线程运行，主线程启动的时候，主线程对应的runloop会被开启，runloop保证了主线程不会被销毁，也就保证了程序的持续运行。
+ 处理App中的各种事件（比如：触摸事件，定时器事件，Selector事件）
+ 节省CPU资源，提高程序性能，Runloop在运行时，当接收到Input sources 或者 Timer source时，

内部工作机制：通过GNUStep源码可以看到，线程和Runloop是一一对应的，其关系保存在一个Dictionary里，所以我们创建子线程Runloop时，只需在子线程中获取当前线程的Runloop对象即可，如果不获取，那么子线程就不会创建与之相关联的Runloop，当Runloop创建时，从__CFRunloop的源码中我们可以看到，其中包含Runloop运行模式的CFRunLoopModeRef结构体，一个Runloop包含若干个Mode，每个Mode又包含若干个Source0/Source1/Timer/Observer，而RunLoop启动时只能选择其中一个Mode作为currentMode；在runloop开启后，会开启事件监听，当有事件唤醒时，根据事件源的类型，通过不同的函数回调处理，当Runloop的model为空或者mode里面没有source/timer/observer时，Runloop会通知Observer即将退出。App启动后，系统在主线程Runloop里面默认注册了两个Observer，其回调都是_wrapRunLoopWithAutoreleasePoolHandler(),
第一个Observer监视的事件是Entry（即将进入Loop），其回调内会调用_objc_autoreleasePoolPush()创建自动释放池，其Order是-2147483647，优先级最高，保证创建释放池发生在所有回调之前
第二个Observer监视了两个事件：BeforeWaiting（准备进入休眠）时调用_objc_autoreleasePoolPop()和_objc_autoreleasePoolPop()来释放自动释放池，这个Observer的order是2147483647，优先级最低，保证其释放池发生在所有其他回调之后                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           

```

> 当调用NSObject的performSelector：afterDelay：后，实际上其内部会创建一个Timer并添加到当前的RunLoop中，所以如果当前线程没有RunLoop，则这个方法会失效
> 
> 当调用performSelector：onThread：时，实际上其会创建一个Timer加入到对应的线程，同样的，如果对应线程没有RunLoop该方法也会失效。



+ 哪些场景可以触发离屏渲染？（知道多少说多少）

```
离屏渲染：在使用圆角、阴影和遮罩等视图功能时，图层属性的混合体被指定为在未预合成之前不能直接在屏幕中渲染，所以就需要在屏幕外的上下文中进行渲染，即为离屏渲染

iOS屏幕渲染的类型：CPU计算好显示内容提交到GPU，GPU渲染完成后将渲染结果放入帧缓冲区，随后视频控制器就会按照VSync信号逐行读取帧缓冲区的数据，经过可能的数模转换传递给显示器显示。

：离屏渲染造成卡顿的原因：
因为离屏渲染需要创建一个屏幕外的缓冲区，然后从当屏缓冲区切换到屏幕外的缓冲区，然后在渲染完成后切换环境大到帧缓冲器；其中创建屏幕外缓冲区和上下文切换最消耗性能，而绘制其实不是性能损耗的最主要原因。

使用Core graphics绘制API的确会触发离屏渲染，但不是GPU的离屏渲染，使用Core Grphics的绘制API是在CPU上执行，触发的是CPU版本的离屏渲染。

CPU计算好显示内容提交到GPU，GPU渲染完成后将渲染结果放入帧缓冲区，随后视频控制器会按照VSync信号逐行读取帧缓冲区的数据，经过可能的数模转换传递给显示器显示；

+ 屏幕渲染的类型：
	+ GPU的屏幕渲染：on-screen Rendering；即当前屏幕渲染，指的是渲染操作在当前用于显示的屏幕缓冲区中进行
	+ off-screen Rendering；即离屏渲染，指的是GPU在当前屏幕缓冲区之外开辟了一个新的缓冲区进行渲染操作。
	+ CPU中的离屏渲染（特殊离屏渲染，即不在GPU中的渲染）；如果我们重写了drawRect方法，并且使用任何Core Graphics的技术进行了绘制操作，就涉及了CPU渲染。

+ 可能导致离屏渲染的原因：
 	1. shouldRasterize(光栅化)
 	2. masks(遮罩)
 	3. shadows(阴影)
 	4. edge antialiasing (抗锯齿)
 	5. group opacity (不透明)
 	6. 复杂形状设置圆角
 	7. 渐变

```

+ block 里面Strong self后，block不是也会持有self吗？而self又持有block，那不是又循环引用了？

```
在block里面strong引用，保证了持有引用的周期只在block被执行时，闭包函数返回后就释放了，而直接用强引用，持有引用的周期则是block的生命周期，就会循环引用了。
```

+ block的使用在什么情况下。不需要使用weak self

```
当block本身不被self持有，而被别的对象持有，同时不产生循环引用的时候，就不需要使用weak self了，最常见的代码就是UIView的动画代码，我们在使用UIView的animateWithDuration:animations动画的时候，并不需要使用weak self，因为引用持有的关系是：UIView 的某个动画对象持有了block，block持有了self，因为self并不持有block，所以就没有循环引用的产生，也就不需要使用weak self
```

---

+ AppDelegate如何瘦身？

```

在iOS开发中，Appdelegate很容易出现代码臃肿、调用顺序混乱、逻辑复杂的问题，作为UIApplication的代理类，是一个常驻内存的单例，它承载了很多的功能：
	+ app的启动代码
	+ 响应app的状态，比如app切换到后台和前台等状态
	+ 响应外部传递给app的通知，比如说push，low-memory warnings
	+ 决定了app的状态是否应该保存或者恢复
	+ 响应不是发送给特定View或者VC，而是发送给app本身的事件
	+ 用来保存一些不属于特定vc的数据。

+ 瘦身的方法大致分为两种：（1）从AppDelegate本身入手，通过这种方式减少AppDelegate的代码行数，比如：FRDModuleManager豆瓣开源的轻量级模块管理工具，它通过减少AppDelegate的代码量来把很多职责拆分到各个模块中去，或者通过代理的方式实现事件的分发、或者通过给appdelegate设置分类也可以解决（2）在架构层面就解决

给Appdelegate添加分类
```

+ 反射是什么？可以举出几个应用场景么？（知道多少说多少）

```
反射是计算机程序在运行时检查、内省、修改自身结构和行为的一种能力。


eg：根据后台推送过来的数据，进行动态页面的跳转，跳转到页面后根据返回的数据执行对应的操作；

这个时候我们可以使用反射机制：我们可以用反射机制动态的创建类并执行方法，当然也可以通过runtime来实现这个功能，这个时候就需要和后台配合，我们首先需要和后台商量好返回的数据结构，以及数据格式、类型等，返回后我们按照和后台约定的格式，根据后台返回的信息，直接进行反射和调用即可。

```

+ 有哪些场景是NSOperation比GCD更容易实现的？（或是NSOperation优于GCD的几点，知道多少说多少）
```
当要控制线程的数量，以及需要建立线程间的依赖的时候，以及需要手动管理线程的挂起以及执行的时候，可以使用NSOperation。
```
+ App 启动优化策略？最好结合启动流程来说（main()函数的执行前后都分别说一下，知道多少说多少）

```
t(App总启动时间) = t1（main（）之前的加载时间）+t2(main()之后加载的时间)
t1 = 系统dylib（动态链接库）和自身App可执行文件的加载
t2 = main方法执行之后到Appdelegate类中：-（BOOL）Application：（UIApplication*）Application didFinishLaunchingWithOptions：（NSDictionary*）launchOptions；方法执行结束前这段时间，主要是构建第一个界面，并完成渲染展示。

main（）调用之前的加载过程：App开始启动之后，系统首先可加载可执行文件（自身App的所有.o文件的集合，）然后加载动态链接库dyld，dyld是一个专门用来加载动态链接库的库。执行从dyld开始，dyld从可执行文件的依赖开始，递归加载所有的依赖动态链接库。

动态链接库包括：iOS中用到的所有系统framework，加载OC Runtime方法的libobjc，系统级别的libSystem，例如libdispatch（GCD）和libsystem_blocks（Block）

其实无论对于系统的动态链接库还是对于App本身的可执行文件来讲，他们都是image（镜像），而每个App都是以image（镜像）为单位进行加载的。
除了我们App本身的可执行文件的，系统中所有的framework比如UIKit、Foundation等都是以动态链接库的方式集成进App中的。


动态链接库加载的具体流程：

1. load dylibs image 读取库镜像文件
2. Rebase image 
3. Bind image
4. Objc setup
5. initializers

总的来概括就是：
1. dyld开始将程序二进制文件初始化
2. 交由ImageLoader读取Image，其中包含了我们的类，方法等各种符号。
3. 由于runtime向dyld绑定了回调，当image加载到内存后，dyld会通知runtime进行处理
4. runtime接手后调用mapimages做解析和处理，接下来loadimages中调用callloadmethods方法，遍历所有加载进来的Class，按继承层级依次调用Class的+load方法和其Category的+load方法。

至此，可执行文件和动态库所有的符号（Class、Protocol、Selector、IMP。。。）都已经按格式加载到内存中，被runtime所管理，在这之后，runtime的那些方法（动态添加的class，swizzle等等才会生效。）

在main（）之前我们能做的优化的点有：
（1）减少不必要的framework，因为动态链接比较耗时
（2）check framework应当设为optional和required，如果该framework在当前App的所有iOS系统版本都存在，那么就设为required，否则就设为optional，因为optional会有一些额外的检查
(3)合并或者删减一些OC类，关于清理项目中没用到的类，使用工具AppCode代码检查功能。
	+ 删除一些无用的静态变量
	+ 删除没有被调用到或者已经废弃的方法
	+ 将不必须在+load方法中做的事情延迟到+initialize中
	+ 尽量不要用C++虚函数

	
main（）函数发生了什么：
main()函数的执行会首先
（1）创建应用程序UIApplication对象，
（2）指定应用程序UIApplication代理
（3）创建并开启主运行循环
（4）加载应用程序配置信息info.plist文件，
	1> 判断Main storyboard file base name中没有指定Main，即需要加载的StoryBoard文件
	2> 如果指定了，就加载Main.storyBoard
	3> 如果没有指定的话，就会黑屏


main（）函数调用之后的加载时间：
在main（）被调用之后，App的主要工作就是初始化必要的服务，显示首页内容等，而我们的优化也是围绕如何能够快速展现首页来开展。App通常在AppDelegate类的-（BOOL）Application：（UIApplication*）application didFininshLaunchingWithOptions：（NSDictionary*）launchOptions;方法中创建首页需要显示的View，然后在当前runloop的末尾，主动调用CA::Transation::commit完成视图的渲染

而视图的渲染主要涉及三个阶段：
1. 准备阶段 这里主要是图片的解码
2. 布局阶段 首页所有UIView的-（void）layoiutSubView()运行
3. 绘制阶段 首页所有UIView的- (void) drawRect:(CGRect)rect运行

因此对于main（）函数调用之前我们可以优化的点有：
1. 不使用xib，直接使用代码加载首页视图
2. 避免读取大容量的Plist文件
3. 删除启动时各业务方的log
4. 梳理启动时发送的网络请求，是否可以统一在异步线程请求

```

+ App 无痕埋点的思路了解么？你认为理想的无痕埋点系统应该具备哪些特点？（知道多少说多少）
+ 你知道有哪些情况会导致app崩溃，分别可以用什么方法拦截并化解？（知道多少说多少）

```
1. 数组越界，给集合类添加分类，在取值或者存值得时候检查是否越界或者值是否为nil；
2. 空指针以及野指针错误；
3. 内存占用过大
4. 
```

+ 你知道有哪些情况会导致app卡顿，分别可以用什么方法来避免？（知道多少说多少）


---

+ HTTP和HTTPS的区别?

```
（1） HTTP 是超文本传输协议，属于明文传输协议，HTTPS则是具有安全性的基于ssl加密的传输协议
（2） HTTPS和HTTPS使用的连接方式不同，而且用的端口也不一样，前者是80，后者是443
（3）HTTP是简单的无状态连接。HTTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议比HTTP协议安全。
（4）HTTPS内容经过对称加密，每个连接生成一个唯一的加密密钥
（5）HTTPS内容传输经过完整性校验
```

+ App 网络层有哪些优化策略？
+ TCP为什么要三次握手，四次挥手？

```
在《计算机网络》一书中有提到，三次握手的目的是为了“防止已经失效的连接请
求报文段突然又传到服务端，而产生错误”，这种情况是：一端（client）A发出
去的第一个连接请求报文并没有丢失，而是因为某些未知的原因在某个节点上发生
滞留，导致延迟到连接释放以后的某个时间才到达另外一端（server）B，本来
这是一个早已失效的报文段，但是B收到此失效的报文之后，会误以为是A再次发
出的一个新的连接请求，于是B端向A又发出确认报文，表示同意连接，如果不采
用“三次握手”，那么只要B端发出确认报文就会认为新的连接已经建立了，但是A
端并没有发出建立连接的请求，因此不回去向B端发送数据，B端没有收到数据就
会一直等待，这样B端就会白白浪费很多资源，如果采用“三次握手”的话就不会出
现这种情况，B端收到一个过时失效的报文段之后，向A端发出确认，此时A并没有
要求建立连接，所以就不会向B端发出确认，这个时候B端也能够知道连接没有建
立。


： 问题的本质是：信道是不可靠的，但是我们要建立可靠的连接发送可靠的数
据，也就是数据传输需要时可靠的，这个时候三次握手是一个理论上的最小值，并
不是说是tcp协议要求的，而是为了满足在不可靠的信道上传输可靠的数据所要求
的。

：四次挥手：TCP四次挥手的原因本质原因是：tcp是全双工的，要实现可靠的链
接关闭，A发出结束报文FIN，收到B确认后A知道自己没有数据需要发送了，B知
道A不在发送数据了，自己也不会接收数据了，此时，A->B方向的连接已经中断，
但B->A方向的连接还存在，此时B可以向A发送数据，A还是可以接收数据；只有
当B发出FIN报文的时候此时两边才会真正的断开连接，

```
![](https://upload-images.jianshu.io/upload_images/616078-5093ce7f19f61840.png!blog?imageMogr2/auto-orient/strip%7CimageView2/2/w/561)

![](https://upload-images.jianshu.io/upload_images/616078-5aeaf2475546f960.png!blog?imageMogr2/auto-orient/)
> 
> 1.** FIN_WAIT_1 ** 表示在等待另一方的FIN报文，和FIN_WAIT_2的区别是，FIN_WAIT_1表示socket现在要主动关闭连接，在发送完FIN报文后socket进入FIN_WAIT_1状态，当收到另一方发送FIN的ACK之后立即进入FIN_WAIT_2状态；
> 
> 2.** FIN_WAIT_2 ** 同上，此时需要做的事情是可能还会接收数据，然后等待另一方的FIN；
> 
3.** TIME_WAIT ** 存在主动关闭的一方，表示收到了对方的FIN报文，并发送出了ACK报文，就等2MSL(Max Segment Lifetime))后即可回到CLOSED可用状态了，需要等一段时间时原因是网络是不可靠的，不能保证这个ACK发送成功了，如果失败了，对端会超时重传FIN；
>
4.** CLOSING ** 表示在发送FIN之后，没有收到对方的ACK，而是收到了对方的FIN，这中情况很少见，只有在两端几乎同时关闭同一个socket的时候才会出现CLOSING状态；
>
5.** CLOSE_WAIT ** 表示收到对方的FIN之后，回给对方ACK，此时处于CLOSE_WAIT状态，等待关闭，要看自己是否还有数据要发送；
>
6.** LAST_ACK ** 表示收到对方的FIN之后，回给对方ACK，然后自己也要关闭发送FIN，等待另一方的ACK时候的状态；
>
7.** CLOSED ** 这个状态表示连接已经断开。


+ 对称加密和非对称加密的区别？分别有哪些算法的实现？

```
1.对称加密中加密和解密使用的是同样的密钥，所以速度快，但是由于需要将密
钥在网络传输，所以安全性不高。
2.非对称加密使用了一对密钥，公钥和私钥，所以安全性高，但加密和解密速度慢。

常见的对称加密算法有：DES、3DES、Blowfish、IDEA、RC4、RC6和AEC

常见的非对称加密算法有：RSA、ECC（移动设备用）、Diffie-Hellman、EI Gamal、DSA（数字签名用）

Hash算法（摘要算法）
Hash算法特别的地方在于它的一种单向算法，用户可以通过hash算法对目标信息生成一段特定长度的唯一hash值，却不能通过这个hash值重新获得目标信息，因此Hash算法常用语在不可还原的密码存储、信息完整性校验。

```

+ HTTPS的握手流程？为什么密钥的传递需要使用非对称加密？双向认证了解么？

![](/Users/shuaike/Desktop/20160908134036615.png)

![](http://zhaox.github.io/assets/images/HttpsFlow.png)

```

```

+ HTTPS是如何实现验证身份和验证完整性的？

```
1. 认证服务器，浏览器内置一个受信任的CA机构列表，并保存这些CA机构的证书，第一阶段服务器会提供经CA机构认证颁发的服务器证书，如果认证该服务器证书的CA机构，存在于浏览器的受信任的CA机构列表中，并且服务器中的信息与当前正在访问的网站（域名等）一致，那么浏览器就认为服务端是可信的，并从服务器证书中取得服务器公钥。用于后续流程，否则，浏览器将提示用户，根据用户的选择，决定是否继续，我们可以管理这个受信任的CA机构列表，添加我们想要信任的CA机构，或者移除我们不信任的CA机构。（在客户端拿到这份证书之后，就会用CA提供的公钥来对数字证书里面的数字签名进行解密得到消息摘要，然后对数字证书里面服务端的公钥和个人信息进行Hash得到另一份消息摘要，然后把两份消息摘要进行比对，如果一样，则证明这些东西确实是服务端的，否则就不是）
2. 协商会话密钥，客户端在认证完服务器，获得服务器的公钥之后，利用该公钥与服务器进行加密通信，协商出两个会话密钥，分别是用于加密客户端往服务端发送数据的客户端会话密钥，用于加密服务端往客户端发送数据的服务端会话密钥。在已有服务器公钥，可以加密通信的前提下，还要协商两个对称密钥的原因，是因为非对称加密相对复杂度更高，在数据传输过程中，使用对称加密，可以节省计算资源，另外，会话密钥是随机生成的，每次协商都会有不一样的结果，所以安全性也比较高。
3. 加密通信。此时客户端服务器双方都有了本次通讯的会话密钥，之后传输的所有http数据，都通过会话密钥加密，这样网路上的其它用户，将很难窃取和篡改客户端和服务端之间传输的数据，从而保证了数据的私密性和完整性。
```

+ 如何用Charles抓HTTPS的包？其中原理和流程是什么？

```
Charles 本身是一个协议代理工具，如果只是普通的HTTP请求，因为数据本省没有经过再次加密，因此作为代理可以知道所有客户端发送到服务端的请求内容以及服务端返回给客户端的数据内容，这也就是抓包工具能够将数据内容直接展示出来的原因，对于HTTPS请求，由于数据都经过了加密，代理如果什么都不做的话是无法获取到其中的内容的，为了实现这个过程的数据获取，Charles需要做的事情就是对客户端伪装成服务端，对服务端伪装成客户端。
具体就是：
1. 截取真是客户端的HTTPS请求，伪装成客户端向真实服务端发送HTTPS请求
2. 接受真实服务器响应，用Charles自己的证书伪装成服务端向真实客户端发送数据内容。

+ 当校验服务端的CA证书时，由于系统只会允许可信的CA签发的数字证书能够访问，私有CA签发的数字证书时无法访问，所以在验证Charles证书的时候，需要将私有CA签发的数字证书安装到手机并且作为受信任的证书保存，

```

+ 什么是中间人攻击？如何避免？
```
当数据传输发生在
```

+ HTTPS数字证书生成
![](https://user-gold-cdn.xitu.io/2018/7/30/164e97bc98d892b3?imageslim)

```
当客户端拿到这份数字证书之后，就会用CA提供的公钥来对数字证书里面的数字签名进行解密得到消息摘要，然后对数字证书里面服务端的公钥和个人信息进行Hash得到另外一份消息摘要，然后把两份消息摘要进行比对，如果一样，则证明消息没有被串改。
```

+ HTTPS是如何保证数据的安全?

```

```

---
+ 了解编译的过程么？分为哪几个步骤？
```
ios app编译的大致流程：
如果项目有第三方依赖库，首先会build依赖库，然后在build主target
1. Compile 各个.m文件
2. copy静态资源，包括img、string、font
3. compile xib，编译xib，生成nib文件
4. compile storyboard，编译storyboard文件，生成.storyboard文件，打开包内容，是nib+plist
5. compile asset catalogs; 生成Asset.car 文件
6. process info.plist 处理info.plist
7. link storyboard 链接storyboard
8. Run custom script 执行脚本
9. Touch app 生成.app文件
10. sign app 对app进行签名
11. validate app 校验app


```
+ 静态链接了解么？静态库和动态库的区别？

```

静态链接：在编译链接可执行文件的时候，就会把程序所需要的二进制代码都包含到可执行文件中去。
动态链接：在编译链接可执行文件的时候，通过记录一系列符号和参数，在程序运行或加载的时候再将这些信息传递给操作系统，操作系统负责将需要的动态库加载到内存中，然后程序在运行到指定的代码时，去共享执行内存中已经加载的动态库可执行代码，最终达到运行时连接的目的。

库是共享程序代码恶方式：一般分为静态库和动态库
ios 静态库：链接时完整地拷贝到可执行文件中。
ios 动态库：链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用
ios 中的静态库形式：.a 和 .framework
ios 中的动态库形式：.dylib 和 .framework

```

+ .a 和 .framework 有什么区别

```
.a 是一个纯二进制文件，.framwwork中除了有二进制文件之外还有资源文件
.a 文件不能直接使用，至少需要.h文件的配合，.framework文件可以直接使用

```

+ 内存的几大区域，各自的职能分别是什么？
+ static和const有什么区别？

```
静态变量：
static : 从面向对象的角度出发，当需要一个一个数据对象为整类而非某个对象服务时，同时又力求不破坏类的封装性，既要求此成员隐藏在类的内部，又要求对外不可见的时候，就可以使用static。在java或C语言中，某个类中声明一个static静态变量，其它类中想使用它或者修改，可以直接使用它的类名拿到这个静态变量的对象，便可以在其他类中任意修改这个变量的数值。然而在Object-c语法中，声明后的static静态变量在其他类中是不能通过类名直接访问的，。他的作用域只能在声明的这个.m文件中，不过可以调用这个类的方法间接的修改这个静态变量的值，标记的变量存储在全局变量区，生命周期和程序相同，只在声明的类中可见，在声明的类中结束之后，再次使用还是之前的值。

const ： 共享一块内存空间，就算项目中N处用到，也不会分配N块内存空间，可以根据const修饰的位置设定能否修改，在编译阶段会执行类型检查。

四种写法：
static const NSString *Coder = @“****”;
const NSString *Coder = @"****";
NSString const *Coder = @"****";
NSString * const Coder = @"****";

全局常量：不管你定义在那个文件夹，外部都能访问到
局部常量： 用static修饰后，不能提供外界访问

const 右边的总不能修改
const NSString *Coder = @"xxx";

"*Coder"不能被修改， "Coder"能被修改

2.NSString const *Coder = @"xxx";

"*Coder"不能被修改， "Coder"能被修改

3.NSString * const Coder = @"xxx";

"Coder"不能被修改，"*Coder"能被修改

extern：全局变量，也称之为外部变量，是在方法外部定义的变量，它不属于那个方法，而是属于整个源程序，作用域是整个源程序，如果全局变量和局部变量重名，则在局部变量作用域内，群局变量被屏蔽。所以编程时，尽量不要使用全局变量。

```

+ 了解内联函数么？
+ 什么时候会出现死锁？如何避免？
+ 说一说你对线程安全的理解？
+ 列举你知道的线程同步策略？
+ 有哪几种锁？各自的原理？它们之间的区别是什么？最好可以结合使用场景来说

---

+ 除了单例，观察者设计模式以外，还知道哪些设计模式？分别介绍一下
+ 最喜欢哪个设计模式？为什么？
+ iOS SDK 里面有哪些设计模式的实践？
```
 (1) 单例 ：在整个应用生命周期内，对象在内存中唯一，
 (2）通知
（3）代理
（4）观察者
 (5) 类簇
（6）工厂模式
 (7)
```

+ 写iOS SDK的注意事项：
```
注意事项1. 所有的类名都应该加前缀,以避免命名冲突
注意事项2. 所有的category方法都应该加前缀，以避免命名冲突
注意事项3. 不要将第三方库打包进SDK，
注意事项4. 做基本的检查和测试，SDK对外公布前应该进行基本的编译检查，不应该有编译器警告存在。
注意事项5. 文档完整并且正确
注意事项6. 图片资源，图片资源一般都是把文件单独放在一个.bundle文件中，一般.bundle的名字和.a 或 .framework的名字相同，
注意事项7. 如果静态库很复杂，需要暴露的.h比较多的话，就可以在静态库的内部创建一个.h文件（一般这个.h文件的名字和静态库的名字相同）,然后把所有的需要暴露出来的.h文件都集中在和这个.h文件中，这样就只需要暴露这个.h文件就可以了。
```
![](/Users/shuaike/Desktop/20170913170403346.png)
```
在制作framework或者lib的时候，如果使用了category， 把category打包成静态库是没有问题的，但是在用这个静态库的工程中，
调用category中的方法时会有找不到该方法的【运行时】错误（selector not recognized），
解决办法是：在使用静态库的工程中配置other linker flags的值为-ObjC
```

+ **设计模式是为了解决什么问题的？
+ **设计模式的成员构成以及工作机制是什么？
+ **设计模式的优缺点是什么？

---

+ MVC和MVVM的区别？MVVM和MVP的区别？

```

```

+ 面向对象的几个设计原则了解么？最好可以结合场景来说。
+ 可以说几个重构的技巧么？你觉得重构适合什么时候来做？
+ 你觉得框架和设计模式的区别是什么？
+ 看过哪些第三方框架的源码，它们是怎么设计的？设计好的地方在哪里，不好的地方在哪里，如何改进？（这道题的后三个问题的难度已经很高了，如果不是太N的公司不建议深究）

---
+ 链表和数组的区别是什么？插入和查询的时间复杂度分别是多少？
+ 哈希表是如何实现的？如何解决地址冲突？
+ 排序题：冒泡排序，选择排序，插入排序，快速排序（二路，三路）能写出那些？
+ 链表题：如何检测链表中是否有环？如何删除链表中等于某个值的所有节点？
+ 数组题：如何在有序数组中找出和等于给定值的两个元素？如何合并两个有序的数组之后保持有序？
+ 二叉树题：如何反转二叉树？如何验证两个二叉树是完全相等的？

----






