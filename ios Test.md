
+ 分类和扩展有什么区别？可以分别用来做什么？分类有哪些局限性？分类的结构体里面有哪些成员？

```
1.分类有分类名，而类扩展没有类名
2.分类可以单独文件创建，而类扩展必须写在类文件中。
3.分类中添加属性只能生成属性的声明，并不会生成对应的访问方法和成员变量，但类扩展可以
	
	
分类的结构体类主要有：
name（分类名）、
cls（分类所属的类）、
method_list（方法列表）、
property_list（属性列表）、
protocol_list(协议列表)
```

+ 讲一下atomic的实现机制；为什么不能保证绝对的线程安全（最好可以结合场景来说）？

```
实现机制是：为对象属性的setter、getter方法加锁，
比如多个线程对属性进行访问，其中一个线程对对象进行了setter操作，使得对
象的引用计数加一，在这个过程中有一个线程对被访问对象进行了release操
作，此时对象的引用计数为0，对象被释放，
还有比如数组的非getter、setter方法，也可能导致线程取到的数据为脏数
据。
```

+ 被weak修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable么？里面的结构可以画出来么？

```
weak修饰的对象被释放时指向对象的指针也会被置为nil，可以避免野指针错误。

对象的引用计数存放：
	
实现：weak实现原理大致概括：在全局的Hash表（sideTables）找到对象所属的引用计数管理对象sideTable，然后在通过所指对象的sideTable.find(对象的地址)找到对应的value结构，sideTable结构体表示在下文，然后在sideTable得成员中通过RefCountMap.find(对象的地址)找到找到对象的引用计数结构表示，
1. 当一个对象被weak修饰时，runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
2. 当添加引用时：objc_initWeak函数会调用objc_storeWeak()函数，objc_storeWeak（）的作用是更新指针指向，创建对应的弱引用表。

3. 在释放时，会通过对象的地址，在全局的Hash表（sideTables）找到对象所属的引用计数管理对象sideTable，然后在通过所指对象的sideTable.find(对象的地址)找到对应的value结构，sideTable结构体表示在下文，然后找到维护对象weak指针的结构体weak_table_t weak_table,它包含两个元素，
> 第一个元素是weak_entry_t *weak_entries是一个数组，上面的RefcountMap是要通过find（key）来找到精确的元素的，而weak_entries则是通过循环遍历来找到对应的entry的。
> 第二个元素num_entries是用来维护保证数组始终有一个合适的size，比如数组中元素的数量超过3/4的时候将数组的大小乘于2。

weak_entry_t 的结构包含三部分：
1. referent
被指对象的地址，前面循环遍历查找的时候就是判断目标地址是否和它相等。
2. referrers
可变数组，里面保存着所有指向这个对象的弱引用的地址，当这个对象被释放的时候，referrers里面的指针都会被设置为nil
3. inline_referrers
只有4个元素的数组，默认情况下用它来存储弱引用的指针，当大于4个的时候使用referrers来存储指针

找到维护weak引用的结构weak_entry_t之后，在weak对象release的时候，标记对象为正在释放状态。然后在dealloc里面调用weak_clear_no_lock(weak_table_t *weak_table, id referent_id)通过要销毁对象的地址找到对应的weak_entry_t入口，然后判断对象的弱引用计数是否超过4，如果超过就会将referrers数组内的弱引用都置为nil，如果不超过4个则将inline_referrers数组内的弱引用都置为nil，最后从weak_table中移除该弱引用对象的weak_entry_t结构对象。

4. 
当一个对象被weak修饰时，系统会以对象的地址为key，将对象加入由系统维护的CFMutableDictionary中，当对象的引用计数为0时，就会去这个全局的字典里面，将weak指针置为nil；
具体的实现： Firday QA 上介绍了一种类似KVO的实现方式，当对象存在weak指针时，我们可以将这个实例指向一个新创建的子类，然后修改这个子类的release方法，在release方法中，去从全局的CFMutableDictionary字典中找到所有的weak对象（这里应该是找到弱引用本对象的地方，将其置为nil）

Class subclass = objc_allocateClassPair(class, newNameC, 0); 

Method release = class_getInstanceMethod(class, @selector(release)); 

Method dealloc = class_getInstanceMethod(class, @selector(dealloc)); 

class_addMethod(subclass, @selector(release), 
(IMP)CustomSubclassRelease, method_getTypeEncoding(release)); 

class_addMethod(subclass, @selector(dealloc), (IMP)CustomSubclassDealloc, method_getTypeEncoding(dealloc)); 

objc_registerClassPair(subclass);


#### sideTable
为了管理所有对象的引用计数和weak指针，苹果创建了一个全局的SideTables，虽然名字后面有个”s“，不过他其实是一个全局的Hash表，里面的内容装的都是SideTable结构体而已，它使用对象的内存地址当它的key，管理引用计数和weak指针。

当我们通过SideTables[key]来得到SideTable的时候，SideTable的结构如下：

struct SideTable {
    spinlock_t slock;
    RefcountMap refcnts;
    weak_table_t weak_table;

    SideTable() {
        memset(&weak_table, 0, sizeof(weak_table));
    }

    ~SideTable() {
        _objc_fatal("Do not delete SideTable.");
    }

    void lock() { slock.lock(); }
    void unlock() { slock.unlock(); }
    void forceReset() { slock.forceReset(); }

    // Address-ordered lock discipline for a pair of side tables.

    template<HaveOld, HaveNew>
    static void lockTwo(SideTable *lock1, SideTable *lock2);
    template<HaveOld, HaveNew>
    static void unlockTwo(SideTable *lock1, SideTable *lock2);
};
从SideTable的结构体中我们可以看到：其中主要有以下几个结构体成员：

1. 一把自旋锁，spinlock_t slock;
：自旋锁比较适用于锁的使用者保存锁的时间比较短的情况，正是由于自旋锁使用者一般保持锁时间非常短，因此选择自旋而不是睡眠是非常必要的，自旋锁的效率远高于互斥锁。信号量和读写信号量适合于保持时间较长的情况，它们会导致调用者睡眠，因此只能在进程上下文使用，而自旋锁适合于保持时间非常短的情况，它可以在任何上下文使用。
2. 引用计数器，RefcountMap refcnts;
: RedCountMap 是一个一层结构，可以通过key直接找到对应的key，进而找到对象真正的引用计数的数据类型。
引用计数器的数据类型是：
typedef __darwin_size_t size_t ，再进一步看它的定义其实是unsigned long,在32位和64位操作系统中，它分别占用32和64个bit，
+ （1UL<<0）的意思是将一个“1”放到最右侧的盒子里面。然后将这个“1”向左移动0位
+ （1UL<<1）的意思是将一个“1”放到最右侧的盒子里，然后将这个“1”向左移动一位
分析引用计数器的结构，从低位到高位：
+ (1UL<<0)    WEAKLY_REFERENCED
表示是否有弱引用指向这个对象，如果有的话(值为1)在对象释放的时候需要把所有指向它的弱引用都变成nil(相当于其他语言的NULL)，避免野指针错误
+ (1UL<<1)     DEALLOCATING
表示对象是否正在释放，1位正在释放，0没有
+ REAL COUNT
图中REAL COUNT的部分才是对象真正的引用计数存储区，所以咱们说的引用计数加一或者减一，实际上是对整个unsigned long加四或者减四，因为真正的引用计数是从2^2位开始的。
+ (1UL<<(WORD_BITS-1))    SIDE_TABLE_RC_PINNED
其中WORD_BITS在32位和64位系统的时候分别等于32和64。随着对象的引用计数不断变大。如果这一位都变成1了，就表示引用计数已经最大了不能再增加了。

3. 维护weak指针的结构体weak_table_t weak_table;

/**
 * The global weak references table. Stores object ids as keys,
 * and weak_entry_t structs as their values.
 */
struct weak_table_t {
    weak_entry_t *weak_entries;
    size_t    num_entries;
    uintptr_t mask;
    uintptr_t max_hash_displacement;
};

weak_entry_t的结构体包含三个部分：
struct weak_entry_t {
<!--被指向对象的地址，前面遍历循环查找的时候就是判断目标地址和他相等-->
    DisguisedPtr<objc_object> referent;
    union {
        struct {

<!--可变数组，里面保存着所有指向这个对象的弱引用的地址，当这个对象被释放的时候，referrers里的所有的指针都会被设置为nil-->
            weak_referrer_t *referrers;
            uintptr_t        out_of_line_ness : 2;
            uintptr_t        num_refs : PTR_MINUS_2;
            uintptr_t        mask;
            uintptr_t        max_hash_displacement;
        };
        struct {
            // out_of_line_ness field is low bits of inline_referrers[1]
            <!--只有四个元素的数组，默认情况下使用它来存储弱引用的指针，当大于4个的时候使用referrers来存储指针-->
            weak_referrer_t  inline_referrers[WEAK_INLINE_COUNT];
        };
    };

    bool out_of_line() {
        return (out_of_line_ness == REFERRERS_OUT_OF_LINE);
    }

    weak_entry_t& operator=(const weak_entry_t& other) {
        memcpy(this, &other, sizeof(other));
        return *this;
    }

    weak_entry_t(objc_object *newReferent, objc_object **newReferrer)
        : referent(newReferent)
    {
        inline_referrers[0] = newReferrer;
        for (int i = 1; i < WEAK_INLINE_COUNT; i++) {
            inline_referrers[i] = nil;
        }
    }
};
在weak对象被释放的时候，最重要的函数时：objc_clear_deallocating改函数的动作如下：
+ 从weak表中获取废弃对象的地址为键值得记录
+ 将包含在记录中的所有weak修饰符变量的地址，赋值为nil
+ 将weak表中weak_entry_t记录移除
 
```
![](https://upload-images.jianshu.io/upload_images/1834534-bd0c3c43ec617a25.jpeg?imageMogr2/auto-orient/)

+ ios中 .m和.mm文件的区别

```
.m 源代码文件：可以包含OC和C代码
.mm 文件，除OC和C代码，还可以包含C++代码，仅在你的代码中确实需要使用C++类或者特性的时候才会使用这种扩展名
```

+ 并发和并行的区别

```
当有多个线程在操作时，如果系统只有一个CPU，则它根本不可能真正同时进行一个以上的线程，它只能把CPU运行时间划分为若干个时间段，再将时间段分配给各个线程执行，在一个时间段的线程代码运行时，其他线程处于挂起状态，这种方式我们称之为并发（Concurrent）
当系统有一个以上CPU时，则线程操作有可能非并发，当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行（Parallel）
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

从源码中可以看到bucket_t中存储这SEL和_imp，通过key->value的形式，以SEL为key，函数实现的内存地址_imp为value来存储方法

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
objc_allocateClassPair 方法名后面是pair的原因是，需要同时创建类和元类（class、meta-class）
```

+ 一个int变量被__block修饰与否的区别？

```
变量一般都存储在堆区，但当变量被__block修饰时，系统会将变量的存储方式由堆区转为栈区。
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

+ 瘦身的方法大致分为两种：
（1）从AppDelegate本身入手，通过这种方式减少AppDelegate的代码行数，比如：FRDModuleManager豆瓣开源的轻量级模块管理工具，它通过减少AppDelegate的代码量来把很多职责拆分到各个模块中去，或者通过代理的方式实现事件的分发、或者通过给appdelegate设置分类也可以解决
（2）在架构层面就解决

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

![](http://zhaox.github.io/assets/images/HttpsRole.png)

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

+ Https是如何保证数据传输的安全性？

```
  1
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

```
在计算机系统中，运行的应用程序都是保存在内存中的，不同类型的数据，保存的内存区域不同。

1. 栈区（stack）有编译器自动分配并释放，存放函数的参数值，局部变量等，栈是系统数据结构，对应线程/进程是唯一的。优点是快速高效，缺点是有限制，数据不灵活【先进后出】;
	栈空间分 静态分配和动态分配两种：
	静态分配是编译器完成的，比如自动变量auto的分配。
	动态分配有alloc函数完成
	栈的动态分配无需释放，也就没有释放函数
	
2. 堆区（heap）由程序员分配和释放，如果程序员不释放，程序结束时，可能会由操作系统回收，比如在ios中alloc都是存放在堆中。
3. 全局区（静态区）（static）全局变量和静态变量的存储是放在一起的，初始化的全局变量和静态变量存放在一块区域，未初始化的全局变量和静态变量在相邻的另一块区域，程序结束后由系统释放。
4. 文字常量区，存放常量字符串，程序结束后由系统释放
5. 程序代码区，存放函数的二进制代码

```

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

```
内联函数 static inline
使用inline修饰的函数，在编译的时候，会把代码直接嵌入调用代码中，就相当于用#define宏定义函数那样。
1. 引入内联函数是为了解决函数调用效率的问题
2. 由于函数之间的调用，会从一个内存地址调用到另外一个内存地址，当函数调用完毕之后还会返回原来函数执行的地址，函数调用会有一定的时间开销，引入内联函数就是为了解决这个问题。
3. 具体的原理就是：没有使用内联函数的汇编代码中，会出现call指令，
  > 将下一条指令的所在地址入栈
  
  > 将子程序的起始地址送入PC（于是CPU的下一条指令就会去执行子程序）
```

+ CADisplayLink 和 NSTimer的不同

```
CADisplayLink 在正常情况下会在iOS的屏幕刷新结束后调用，精确度比较高，而NSTimer的精确度稍低，如果NSTimer的触发时间到了，而RunLoop处于阻塞状态，则其触发时间就会
```

+ 内联函数和宏定义的区别

```
1.#define 定义的格式有要求，而使用inline则就像平常写函数那样，只要加上“inline”即可
2. 使用#define宏定义的代码，编译器不会对其进行参数有效性检查，仅仅只是对符号表进行替换。
3. #define宏定义的代码，其返回值不能强制转换成可替换的合适转换类型
4. 在inline加上‘static’修饰符，只是表明该函数在该文件内可见，也就是说，在同一工程内，就算在其他文件中出现同名、同参数的函数也不会引起函数重复定义的错误。

```

+ 什么时候会出现死锁？如何避免？

```
当两个线程之间互相等待对方完成才能执行任务完成的时候，会产生线程死锁的问题。
避免的方法：是查看是否在串行队列中，在线程执行任务的过程中，调用会阻塞这个线程的函数即可。
```

+ 说一说你对线程安全的理解？

```
线程安全：即当有多个线程访问同一块资源时，很容易引发数据错乱和数据安全问题。

```

+ 列举你知道的线程同步策略？

```
+ 可以利用串行队列实现线程同步
+ 可以利用GCD的栈栏方法实现线程同步
+ 可以利用GCD的信号量机制实现线程同步
+ 可以利用线程组的方式实现线程同步
+ 给多线程需要共同访问的资源加锁，

```

+ 有哪几种锁？各自的原理？它们之间的区别是什么？最好可以结合使用场景来说

```
互斥锁：互斥锁是排它的，意思是锁在某个线程获取之后，只有获取锁的线程才能释放这个锁。其他线程必须等到获取锁的线程不再拥有锁之后，才能继续执行。具体的原理是，当线程访问了已经加锁的临界资源时，检测到代码加锁，于是切换至内核态进行进一步的操作，伪代码的大致实现如下：

if (!lock.try_lock()){
	// 切换至内核态
	thread current = this;
	list queue = get_global_wait_list();
	queue.push(current);
	current.sleep(forever);
}

此时线程会进行休眠状态避免继续占用CPU资源，然后等待锁持有者执行完成释放锁，一旦任务完成，会检测是否存在等待执行代码的线程，如果存在，唤醒继续执行任务。

list queue = get_global_wait_list()
if ((t = queue.pop())){
 	t.wakeup()
}
互斥的实现涉及到可能发生的内核态切换、线程休眠、唤醒等，如果临界执行代码足够小而快，互斥的线程锁可能并不是最佳的实践方案。

自旋：自旋的实现要比互斥简单的多，只要标记位的修改被设计为原子操作，就能保证多线程环境下的安全。对比互斥方案，自旋没有线程切换、休眠唤醒的开销，但是空转的代码会导致CPU在等待期间是满负荷执行的，如果加锁的代码不够小而快，甚至会直接影响程序的运行。

信号量：信号量拥有比互斥锁更多的用途，当信号量的value大于0时，所有的线程都能访问临界资源，在线程进入临界区后，value减一，反之亦然，如果信号量初始化为0时，可以看做是等待任务执行完成而非资源保护，value的操作应当是采用原子操作来保证指令的安全性的。信号的性能在自旋和互斥之间，通常的性能表现总是仅次于自旋，这里基于GCD的信号量实现来看，在进入等待时，会根据传入的超时时间出现三种表现：

-DISPATCH_TIME_NOW
-DISPATCH_TIME_FOREVER
根据超时时间的设置，信号量最终会表现为互斥或者自旋的方式实现，这也是为什么评测中信号量性能总是优于互斥低于自旋，虽然信号量的性能不是最优，但是这种结合方案保证了它的作用范围更大。



Barrier：barrier的任务总是保证在执行过程中，并发队列中有且只有barrier的任务在执行。查看libdispatch的源码中得以窥见真容：

void dispatch_barrier_async_f(dispatch_queue_t dq ,void *ctxt , dispatch_function_t func){
	dispatch_continuation_t dc;
	
	dc = fastpath(_dispatch_continuation_alloc_cacheonly());
	if (!dc){
		return _dispatch_barrier_async_f_slow(dq ,ctxt ,func);
	}
	
	dc->do_vtable = (void *)(DISPATCH_OBJ_ASYNC_BIT | DISPATCH_OBJ_BARRIER_BIT);
	dc->dc_func = func;
	dc->dc_ctxt = ctxt;
	
	_dispatch_queue_push(dq ,dc);
}

相比dispatch_async的实现，barrier只是简单的将任务标记为DISPATCH_OBJ_ASYNC_BIT。但在执行队列任务_dispatch_queue_drain会循环获取任务并且判断，barrier任务的真正实现在这个函数中。由于函数实现稍长，笔者只是放上去除额外参数的伪代码

void _dispatch_queue_drain(){
	while((task = queue.next())) {
		return;
	}else if(task.do_vtable & DISPATCH_OBJ_ASYNC_BIT){
		return;
	}else{
		task.execute();
	}
}

当循环取出队列任务执行的时候，检测到当前存在barrier的任务，则停止任务获取，直到当前所有的任务执行完成。并且在barrier执行过程中，不允许执行其他任务。

```

---

+ 除了单例，观察者设计模式以外，还知道哪些设计模式？分别介绍一下

```
1. 适配器模式：适配器模式将一个类的接口适配成用户所期待的，一个适配器通常允许因为接口不兼容而不能一起工作的类能够在一起工作，做饭是将类自己的接口包裹在一个已存在的类中。
：如何使用适配器：以下情况下比较适合使用适配器:
	+ 当你想使用一个已经存在的类，而它的接口不符合你的需求；
	+ 你想创建一个可以复用的类，该类可以与其他不相关的类或者不可预见的类协同工作
	+ 你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口，对象适配器可以适配它的父亲接口
：适配器模式的优缺点：
	+ 优点：降低数据层和视图层（对象）的耦合度，使之使用更加广泛，适应复杂多变的变化
	+ 缺点：降低了可读性，代码量增加，对于开发维护人员增加了理解成本，使代码可读性变低。

2. 策略模式：策略模式定义了一系列的算法，并将每一个算法封装起来，而且使它们还可以相互替换，策略模式让算法独立于使用它的客户而独立变化
：如何使用策略模式：在有多种算法相似的情况下，使用if...else所带来的复杂和难以维护
	+ 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为
	+ 一个系统需要动态地在几种算法中选择一种。
	+ 如果一个对象有很多行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现
	+ 注意事项：如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题。
：策略模式的优缺点：
	+ 简化操作，提高代码的维护性。算法可以自由切换，避免使用多重条件判断，扩展性良好
	+ 缺点：在使用之前就要确定使用某种策略，而不是动态的选择策略，策略类会增多，所有策略类都需要对外暴露
：观察者模式：当对象间存在一对多关系时，则使用观察者模式，比如，当一个对象被修改时，则会自动通知它的依赖对象，观察者模式属于行为型模式。
：如何使用观察者模式：一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作，一个对象（目标对象）的状态发生改变，所有的依赖对象都将得到通知，进行广播通知。
：观察者模式的优缺点：
	 + 优点：观察者和被观察者是抽象耦合的
	 + 建立了一套触发机制
：缺点：
		+ 如果一个被观察者对象有很多的直接或间接的观察者的话，将所有的观察者都通知到会花费很多时间。
		+ 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发他们之间进行循环调用，可能导致系统崩溃。
		+ 观察者模式没有相应的机制让观察者的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

3.工厂模式：这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。
：工厂模式的使用：我们明确地计划不同条件下创建不同实例时。作为一种创建型模式，在任何需要生产复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过new就可以完成创建的对象，无需使用工厂模式，如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。
：工厂模式的优缺点：
	+ 优点：一个调用者想创建一个对象，只要知道其名称就可以了。
	+ 扩展性高：如果想增加一个产品，只要扩展一个工厂类就可以了。
	+ 屏蔽产品的具体实现，调用者只要关心产品的接口。
	+ 工厂类中包含了必要的逻辑判断，根据客户端的选择条件动态实例化相关的类，对于客户端来说，去除了与具体产品的依赖，

缺点：+ 每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。

4.工厂方法模式：工厂方法使用OOP的多态性，将工厂和产品都抽象出一个基类，在基类中定义统一的接口，然后在具体的工厂中创建具体的产品，
工厂方法模式的参与者：
				+ 抽象工厂角色：与程序无关，任何在模式中创建对象的工厂必须实现这个接口。
				+ 具体工厂角色：实现了抽象工厂接口的具体类，含有与引用密切相关的逻辑，并且受到应用程序的调用以创建产品对象。
				+ 抽象产品角色：工厂方法所创建产品对象的超类型，也就是产品对象的共同父类或共同拥有的接口。
				+ 具体产品角色：这个角色实现了抽象产品角色所声明的接口，工厂方法所创建的每个具体产品对象都是某个具体产品角色的实例。
：工厂方法模式的优缺点：降低了工厂类的内聚，满足了类之间的层次关系，又很好的符合了面向对象设计中的单一职责原则，与简单工厂模式把所有的产品都放到工厂类来获取相比，工厂模式每次需要一个新的产品，就需要新建一个具体工厂来生成新的产品，

5.抽象工厂模式：抽象工厂模式是围绕一个超级工厂创建其他工厂，该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。所谓抽象工厂是指一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象，抽象工厂和工厂方法的区别是，工厂方法中具体工厂一般只生产一个或几个控件对象，而抽象工厂中的具体工厂生产的是一族控件对象。

在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显示指定它们的类，每个生成的工厂都能按照工厂模式提供对象。

：抽象工厂中的参与者：
	+ 抽象工厂（Abstract Factory）角色：担任这个角色的是工厂方法模式的核心，它是与应用系统商业逻辑无关的。
	+ 具体工厂（Concrete Factory）角色：这个角色直接在客户端的调用下创建产品的实例。这个角色含有选择合适的产品对象的逻辑，而这个逻辑是与应用系统的商业逻辑紧密相关的。
	+ 抽象产品（Abstract Product）角色：担任这个角色的类是工厂方法模式所创建的对象的父类，或他们共同拥有的接口。
	+ 具体产品（Concrete Product）角色：抽象工厂模式所创建的任何产品对象都是某一个具体产品类的实例，这是客户端最终需求，其内部包含应用系统的商业逻辑。
：抽象工厂的使用场景：
	+ 一个系统不应当依赖于产品类实例如何不创建、组合和表达的细节，这对于所有形态的工厂模式都是重要的。
	+ 这个系统有多余一个的产品族，而系统只消费其中某一个产品族。同属同一产品族的产品是在一起使用的，这一约束必须在系统的设计中提现出来。
	+ 系统提供一个产品类的库，所有的产品以同样的接口出现，从而使客户端不依赖于实现。

抽象工厂模式和工厂方法模式的区别：
+ 工厂方法模式：每个抽象产品派生多个具体的产品类，每个抽象工厂类派生多个具体工厂类，每个具体工厂类负责一个具体产品的实例创建。
+ 抽象工厂模式：每个抽象产品派生出多个具体的产品类，每个抽象工厂派生出多个具体工厂类，每个具体工厂负责多个具体产品的实例创建。

：总结
	：从简单工厂模式到工厂模式，再到抽象工厂模式，可以看到整个模式的一步步演进，简单工厂模式在产品多样之后，整个工厂将会变得臃肿而难以维护，于是我们将简单工厂模式中的工程做了抽象处理，这样每种产品对于一个工厂，虽然这样会增加代码量，但是好处也是显而易见的，单独让一个工厂处理一种产品会让逻辑变得好维护，但是这样还不够，因为增加新的品类，就会产生新的类，对于调用者来说，处理太多具有相同接口的类显然是不合算的，于是，我们使用抽象工厂来解决这个问题，我们封装抽象工厂内部，用以隐藏真正的具体工厂，这样，对于调用者来说，即使内部增加新产品，外部调用也不知道。

6. 代理模式
	：在代理模式中（Proxy Pattern）中，一个类代表另外一个类的功能，这种类型的设计模式属于结构型模式；
	：在代理模式中，我们创建具有现有对象的对象，以便向外界提供功能接口。

代理模式的优缺点：
	：职责清晰、高扩展性、智能化
	缺点：
	：实现代理模式需要额外的工作，有些代理模式的实现非常复杂
7. 单例模式
	：这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。
	注意：
		：单例类只能有一个实例。
		：单例类必须自己创建自己的唯一实例。
		：单例类必须给所有其他对象提供这一实例。
	如何使用单例模式：
		：当你想控制实例数目，节省系统资源的时候。
	单例模式的优缺点：
		优点：在内存中只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例
		：避免了对资源的多重占用比如读写文件操作
		缺点：
			：没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化

7. 生成器模式
	：建造者模式（Build Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种模式的设计模式属于创建型模式，它提供了一种创建对象的最佳方式
	：如何使用生成器模式：
		+ 主要解决在软件系统中，有时候面临真”一个复杂对象“的创建工作，其通常由各个部分的子对象用一定的算法构成，由于需求的变化，这个复杂对象的各个部分经常面临着变化，但是将他们组合在一起的算法却相对稳定
		+ 一些基本部件不会变，而其组合却会经常变化的时候。
	：生成器模式的优缺点：
		+ 建造者独立，易扩展
		+ 便于控制细节风险
	缺点： 产品必须有共同点，范围有限制
		：如内部变化复杂，会有很多的建造类
	：使用场景：
		+ 需要生成的对象具有复杂的内部结构
		+ 需要生成的对象内部属性本身相互依赖。
		注意事项：与工厂模式的区别是：建造者模式更加关注于零件装配的顺序。

```

+ 最喜欢哪个设计模式？为什么？

```
```

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

```
数组在内存中是连续存储的，所以它的索引速度非常快，而且赋值和修改元素也很简单，但是数组也存在一些不足的地方，在数组中的两个数据间插入数据是很麻烦的，而且在声明数组的时候必须指定数组的长度，数组的长度过长，会造成内存浪费，过短会造成数据溢出的错误，
查询的时间复杂度是O(n),插入或删除的时间复杂度是：O(n)

链表是在非连续的内存单元中保存数据，并且通过指针将各个内存单元链接在一起，最后一个节点的指针指向NULL，链表不需要提前分配固定大小内存空间，当需要存储数据的时候分配一块内存并将这块内存插入链表中。
查询的时间复杂度是O(n)，插入或删除的时间复杂度是O（1）。
```

+ 哈希表是如何实现的？如何解决地址冲突？

```
散列表（Hash table也叫哈希表），是根据关键码值（key-Value）而直接进行访问的数据结构，也就是说，它通过把关键码值映射到表中的一个位置来访问记录，以加快查找的速度，这个映射函数叫做散列函数，存放记录的数组叫做散列表。

哈希函数的构造方法：
1. 直接定值法：取关键字或关键字的某个线性函数值为哈希值
2. 数字分析法：假设关键字是以r为基的数，并且哈希表中可能出现的关键字都是事先知道的，则可取关键字的若干数位组成哈希地址。
3. 平方取中法：取关键字的平方后的中间几位为哈希地址
4. 斐波那契散列法
5. 折叠法：将关键字分割成位数相同的几部分（最后一部分的位数可以不同），然后取这几部分的叠加和（舍去进位）作为哈希地址，这方法称为折叠法
6. 除留余数法：取关键字被某个不大于哈希表表长m的数p除后所得余数为哈希地址。即H(key)=key MOD p,(p<=m)，这是一种最简单，也是最常用的构造哈希函数的方法。它不仅可以对关键字直接取模（MOD），也可在折叠、平方取中等运算之后取模。
7. 随机数法：选择一个随机函数，取关键字的随机函数值为它的哈希地址，即H(key)=random(key)，其中random为随机函数。通常，当关键字长度不等时采用此法构造哈希函数较切当。

哈希表的缺陷：哈希表总会不可避免的产生冲突（尤其是在数据量很大的时候），这个时候需要一个方法来解决冲突，以下是几种常见的解决冲突的方法：
1. 开放定址法
2. 在哈希法
3. 链地址法
4. 建立一个公共溢出区

哈希散列表的底层主要是基于数组和链表来实现的，它之所于有相当快的查询速度主要是因为它是通过计算散列码来决定存储的位置的，HashMap中主要通过key的hashCode来计算hash值得，只要hashCode相同，计算出来的hash值就一样，如果存储的对象多了，就有可能不同的对象所算出来的hash值是相同的，这就出现了hash地址冲突的问题，
```

+ 排序题：冒泡排序，选择排序，插入排序，快速排序（二路，三路）能写出那些？

```

void exchangeAction( int a[],int i , int j){
	int temp = a[i];
	a[j] = a[i];
	a[i] = temp;
}

1.冒泡排序：
void sortByMaoPao(int a[]){
	int length = sizeof(a)/sizeof(a[0]);
	for (int i = 0 ; i < length - 1;i++){
		for (int j = 0 ; j < length - 1 - i ; j++){
			if (a[j] > a[i]){
				// 交换a[i] 和 a[j] 的位置
				exchangeAction(*a[i] , *a[j]);
			}
		}
	}
}

2. 选择排序
3. 

```

+ 链表题：如何检测链表中是否有环？如何删除链表中等于某个值的所有节点？

```
1. 设置两个指针，fast和slow，从节点的起点开始遍历，fast指针每次偏移两个节点，slow每次偏移一个节点，如果fast指针不为NULL,且fast->next指针不为NULL，这一直进行循环判断fast == slow，如果结果成立，说明链表中有环，


2. 删除链表中等于某个值的所有节点
pNode removeTargetNode(pNode nodeList , int target){
	pNode p,q,head;
	head = nodeList;
	<!--去除头部节点等于target-->
	while(head->val == target){
		p = head->next;
		free(head);
		head = p;
	}
	p = head;
	q = head->next;
	while(q){
		if (q->val == target){
			// 如果q节点的值等于target
			p->next = q->next;
			q = p->next;
			continue;
		}
		// 如果q节点的值不等于target，则往下继续遍历
		p = q;
		q = p->next;
	}
	return head;
}

```

+ 数组题：如何在有序数组中找出和等于给定值的两个元素？如何合并两个有序的数组之后保持有序？

+ 二叉树题：如何反转二叉树？如何验证两个二叉树是完全相等的？

----






