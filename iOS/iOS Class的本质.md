不管是类对象还是元类对象，类型都是Class，class和meta-class的底层都是objc_class结构体的指针，内存中就是结构体。

我们查看objc_class内部，可以看到这段在底层原理中经常出现的代码：

```
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

```

**objc_class 定义:**

![](/Users/shuaike/Desktop/F993032A-77C5-4CBA-B0E5-BAB8585DEF64.png)


**objc\_object 定义：**

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/04F54C3E-C08D-4E35-8DF8-1FBFAB834F29.png)


**class\_rw\_t 定义：**

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/0B131962-9CAA-4E00-A84E-6D54E2E3B656.png)

可以看到，在class\_rw\_t 中存储着类的

+ 在objc\_class 结构体中的注释中写到class\_data\_bits\_t相当于class\_rw\_t指针加上rr/alloc标志

+ 方法列表，
+ 属性列表，
+ 协议列表

而**class\_rw\_t**是通过bits调用data得到的，我们来到data方法内部实现，我们可以看到，data函数内部仅仅是对bits进行&FAST_DATA_MASK进行位运算操作,只取其中的 [3, 47] 位转换成 class_rw_t * 返回。

在 x86\_64 架构上，Mac OS 只使用了其中的 47 位来为对象分配地址。而且由于地址要按字节在内存中按字节对齐，所以掩码的后三位都是 0。

因为class_rw_t * 指针只存于第【3，47】位，所以可以使用最后三位来存储关于当前类的其他信息：

![](/Users/shuaike/Desktop/1463472850397719.jpg)

```
// class is a Swift class
#define FAST_IS_SWIFT           (1UL<<0)
// class's instances requires raw isa
#define FAST_REQUIRES_RAW_ISA   (1UL<<1)
// class or superclass has .cxx_destruct implementation
//   This bit is aligned with isa_t->hasCxxDtor to save an instruction.
#define FAST_HAS_CXX_DTOR       (1UL<<2)
// data pointer
#define FAST_DATA_MASK          0x00007ffffffffff8UL
```

+ isSwift(): FAST_IS_SWIFT 用于判断 Swift 类

+ hasDefaultRR()：FAST_HAS_DEFAULT_RR 当前类或者父类含有默认的 retain/release/autorelease/retainCount/_tryRetain/_isDeallocating/retainWeakReference/allowsWeakReference 方法

+ requiresRawIsa()：FAST_REQUIRES_RAW_ISA 当前类的实例需要 raw isa

执行 class_data_bits_t 结构体中的 data() 方法或者调用 objc_class 中的 data() 方法会返回同一个 class_rw_t * 指针，因为 objc_class 中的方法只是对 class_data_bits_t 中对应方法的封装。

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/4BCD0267-CC9F-4865-BE67-123783DCCB96.png)

从class\_rw\_t结构体中可以看到其中有一个指向常量的指针ro,其中存储了当前类在编译期就已经确定的属性、方法以及遵循的协议。

**class\_ro\_t 定义：**

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/B8130D07-F3BA-4FDD-BEF0-E905CC5F5E02.png)

在编译期类的结构中的class\_data\_bits\_t \*data指向的是一个class\_ro\_t *指针。

![](/Users/shuaike/Desktop/1463472924453436.jpg)


然后在类加载的过程中在realizeClass方法中：

+ 从 class_data_bits_t 调用 data 方法，将结果从 class_rw_t 强制转换为 class_ro_t 指针

+ 初始化一个 class_rw_t 结构体

+ 设置结构体 ro 的值以及 flag

+ 最后设置正确的 data。

```
/***********************************************************************
* realizeClass
* Performs first-time initialization on class cls, 
* including allocating its read-write data.
* Returns the real class structure for the class. 
* Locking: runtimeLock must be write-locked by the caller
**********************************************************************/
static Class realizeClass(Class cls)
{
    runtimeLock.assertWriting();

    const class_ro_t *ro;
    class_rw_t *rw;
    Class supercls;
    Class metacls;
    bool isMeta;

    if (!cls) return nil;
    if (cls->isRealized()) return cls;
    assert(cls == remapClass(cls));

    // fixme verify class is not in an un-dlopened part of the shared cache?

    ro = (const class_ro_t *)cls->data();
    if (ro->flags & RO_FUTURE) {
        // This was a future class. rw data is already allocated.
        rw = cls->data();
        ro = cls->data()->ro;
        cls->changeInfo(RW_REALIZED|RW_REALIZING, RW_FUTURE);
    } else {
        // Normal class. Allocate writeable class data.
        rw = (class_rw_t *)calloc(sizeof(class_rw_t), 1);
        rw->ro = ro;
        rw->flags = RW_REALIZED|RW_REALIZING;
        cls->setData(rw);
    }

    isMeta = ro->flags & RO_META;

    rw->version = isMeta ? 7 : 0;  // old runtime went up to 6


    // Choose an index for this class.
    // Sets cls->instancesRequireRawIsa if indexes no more indexes are available
    cls->chooseClassArrayIndex();

    if (PrintConnecting) {
        _objc_inform("CLASS: realizing class '%s'%s %p %p #%u", 
                     cls->nameForLogging(), isMeta ? " (meta)" : "", 
                     (void*)cls, ro, cls->classArrayIndex());
    }

    // Realize superclass and metaclass, if they aren't already.
    // This needs to be done after RW_REALIZED is set above, for root classes.
    // This needs to be done after class index is chosen, for root metaclasses.
    supercls = realizeClass(remapClass(cls->superclass));
    metacls = realizeClass(remapClass(cls->ISA()));

#if SUPPORT_NONPOINTER_ISA
    // Disable non-pointer isa for some classes and/or platforms.
    // Set instancesRequireRawIsa.
    bool instancesRequireRawIsa = cls->instancesRequireRawIsa();
    bool rawIsaIsInherited = false;
    static bool hackedDispatch = false;

    if (DisableNonpointerIsa) {
        // Non-pointer isa disabled by environment or app SDK version
        instancesRequireRawIsa = true;
    }
    else if (!hackedDispatch  &&  !(ro->flags & RO_META)  &&  
             0 == strcmp(ro->name, "OS_object")) 
    {
        // hack for libdispatch et al - isa also acts as vtable pointer
        hackedDispatch = true;
        instancesRequireRawIsa = true;
    }
    else if (supercls  &&  supercls->superclass  &&  
             supercls->instancesRequireRawIsa()) 
    {
        // This is also propagated by addSubclass() 
        // but nonpointer isa setup needs it earlier.
        // Special case: instancesRequireRawIsa does not propagate 
        // from root class to root metaclass
        instancesRequireRawIsa = true;
        rawIsaIsInherited = true;
    }
    
    if (instancesRequireRawIsa) {
        cls->setInstancesRequireRawIsa(rawIsaIsInherited);
    }
// SUPPORT_NONPOINTER_ISA
#endif

    // Update superclass and metaclass in case of remapping
    cls->superclass = supercls;
    cls->initClassIsa(metacls);

    // Reconcile instance variable offsets / layout.
    // This may reallocate class_ro_t, updating our ro variable.
    if (supercls  &&  !isMeta) reconcileInstanceVariables(cls, supercls, ro);

    // Set fastInstanceSize if it wasn't set already.
    cls->setInstanceSize(ro->instanceSize);

    // Copy some flags from ro to rw
    if (ro->flags & RO_HAS_CXX_STRUCTORS) {
        cls->setHasCxxDtor();
        if (! (ro->flags & RO_HAS_CXX_DTOR_ONLY)) {
            cls->setHasCxxCtor();
        }
    }

    // Connect this class to its superclass's subclass lists
    if (supercls) {
        addSubclass(supercls, cls);
    } else {
        addRootClass(cls);
    }

    // Attach categories
    methodizeClass(cls);

    return cls;
}
```

下图是realizeClass方法执行过后的类所占用的内存的布局，通过对比可以看到：

![](http://cc.cocimg.com/api/uploads/20160517/1463473009766529.jpg)

但是，在这段代码运行之后class\_rw\_t中的方法、属性以及协议列表均为空，这时需要realizeClass调用methodizeClass方法来将类自己实现的方法（包括分类）、属性和遵循的协议加载到methods、properties和protocols列表中。

总结：

+ 类在内存中的位置是在编译期决定的，在之后修改代码，也不会改变内存中的位置。

+ 类的方法、属性以及协议在编译期存放到了”错误“的位置，直到realizeClass执行之后，才放到了class\_rw\_t指向的只读区域class\_ro\_t,这样我们即可在运行时为class\_rw\_t添加方法，也不会影响类的只读结构。

+ 在class\_ro\_t 中的属性在运行期间就不能改变了，再添加方法时，会修改class\_rw\_t中的methods列表，而不是class\_ro\_t中的baseMthods。

