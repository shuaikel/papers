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

新版本的objc_class 定义:

![](/Users/shuaike/Desktop/FBD8B798-A73F-4B0B-A83E-11DEF0A2AEFA.png)

```

```

objc\_object 定义：

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/04F54C3E-C08D-4E35-8DF8-1FBFAB834F29.png)


class\_rw\_t 定义：

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/0B131962-9CAA-4E00-A84E-6D54E2E3B656.png)

可以看到，在class\_rw\_t 中存储着类的

+ 方法列表，
+ 属性列表，
+ 协议列表

而class_rw_t 是通过bits调用data得到的，我们来到data方法内部实现，我们可以看到，data函数内部仅仅是对bits进行&FAST_DATA_MASK操作。

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/4BCD0267-CC9F-4865-BE67-123783DCCB96.png)

而成员变量信息则存储在class_ro_t内部，
class_ro_t 定义：

![](/Users/shuaike/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/718401263/QQ/Temp.db/B8130D07-F3BA-4FDD-BEF0-E905CC5F5E02.png)


