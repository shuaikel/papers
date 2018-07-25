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
	
实现：当一个对象被weak修饰时，系统会以对象的地址为key，将对象加入由系统维护的Hash-Map中，当对象的引用计数为0时，找到相应的对象，释放对象的内存空间，同时将指向对象的指针置为nil；
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
class_ro_t 是class_rw_t的结构体成员变量， class_ro_t 存储对象的成员变量、以及
成员变量的size。类名。 class_rw_t 包含 class_ro_t类型成员，方法列表，协议列
表，属性列表
```

+ iskindOfClass：class 与 isMemberOfClass：class的区别

```
iskindOfClass 是检查对象是否是那个类或者其继承类实例化的对象
isMemberOfClass 是检查对象是否是那个类但不包括继承类实例化的对象。
```

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
因为离屏渲染需要创建一个屏幕外的缓冲区，然后从当屏缓冲区切换到屏幕外的缓冲区，然后在完成渲染；其中创建屏幕外缓冲区和上下文最消耗性能，而绘制其实不是性能损耗的最主要原因。

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
给Appdelegate添加分类
```

+ 反射是什么？可以举出几个应用场景么？（知道多少说多少）
+ 有哪些场景是NSOperation比GCD更容易实现的？（或是NSOperation优于GCD的几点，知道多少说多少）
```
当要控制线程的数量，以及需要建立线程间的依赖的时候，以及需要手动管理线程的挂起以及执行的时候，可以使用NSOperation。
```
+ App 启动优化策略？最好结合启动流程来说（main()函数的执行前后都分别说一下，知道多少说多少）
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

+ App 网络层有哪些优化策略？
+ TCP为什么要三次握手，四次挥手？
+ 对称加密和非对称加密的区别？分别有哪些算法的实现？
+ HTTPS的握手流程？为什么密钥的传递需要使用非对称加密？双向认证了解么？
+ HTTPS是如何实现验证身份和验证完整性的？
+ 如何用Charles抓HTTPS的包？其中原理和流程是什么？
+ 什么是中间人攻击？如何避免？

---
+ 了解编译的过程么？分为哪几个步骤？
+ 静态链接了解么？静态库和动态库的区别？
+ 内存的几大区域，各自的职能分别是什么？
+ static和const有什么区别？
+ 了解内联函数么？
+ 什么时候会出现死锁？如何避免？
+ 说一说你对线程安全的理解？
+ 列举你知道的线程同步策略？
+ 有哪几种锁？各自的原理？它们之间的区别是什么？最好可以结合使用场景来说

---

+ 除了单例，观察者设计模式以外，还知道哪些设计模式？分别介绍一下
+ 最喜欢哪个设计模式？为什么？
+ iOS SDK 里面有哪些设计模式的实践？
+ **设计模式是为了解决什么问题的？
+ **设计模式的成员构成以及工作机制是什么？
+ **设计模式的优缺点是什么？

---

+ MVC和MVVM的区别？MVVM和MVP的区别？
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






