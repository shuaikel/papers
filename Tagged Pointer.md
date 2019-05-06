#### Tagged Pointer

+ 从64bit开始，iOS引入了Tagged Pointer技术，用于优化NSNumber、NSData、NSString等小对象的存储

+ 在没有使用Tagged Pointer之前， NSNumber等对象需要动态分配内存、维护引用计数等，NSNumber指针存储的是堆中NSNumber对象的地址值

+ 使用Tagged Pointer之后，NSNumber指针里面存储的数据变成了：Tag + Data，也就是将数据直接存储在了指针中
当指针不够存储数据时，才会使用动态分配内存的方式来存储数据

+ objc_msgSend能识别Tagged Pointer，比如NSNumber的intValue方法，直接从指针提取数据，节省了以前的调用开销


![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%BA%95%E5%B1%82/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/TaggedPointer3.png)

#### 判断是否是TaggedPointer


我们点击isTaggedPointer方法

```
_objc_isTaggedPointer(const void * _Nullable ptr) 
{
return ((uintptr_t)ptr & _OBJC_TAG_MASK) == _OBJC_TAG_MASK;
}
```


+ define _OBJC_TAG_MASK 1UL

```
#if TARGET_OS_OSX && __x86_64__
#   define OBJC_MSB_TAGGED_POINTERS 0
#else
#   define OBJC_MSB_TAGGED_POINTERS 1
#endif


#if OBJC_MSB_TAGGED_POINTERS
#   define _OBJC_TAG_MASK (1UL<<63)
#else
#   define _OBJC_TAG_MASK 1UL
#endif
```

在判断是否有TaggedPointer的时候，在iOS平台和MAC平台上还是不太一样：


+ 1. iOS平台，需要把1向左移动63位，也就是最高有效位1（第64bit）；
+ 2. 在mac平台，最低有效位是1






