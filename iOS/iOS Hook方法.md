+ iOS的OC类中可以通过category添加扩展方法，却无法添加实例属性？

> <span>原因是因为属性方法中对属性变量的读写是在编译时直接通过对对象指针的偏移值来获取的，也就是访问在编译时就死定了，无法在运行时被改变，如果分类能够添加实例变量则会影响到对象实例的内存布局方式，从而造成访问异常，所以分类无法添加实例变量属性</span>
> 


运行时Hook所有Block方法调用的技术实现

+ 1.方法调用的几种Hook机制

> iOS系统中一共有：C函数、Block、OC类方法三种形式的方法调用，Hook一个方法调用的目的一般是为了监控拦截或者统计一些系统的行为，Hook的机制有很多种，通常良好的Hook方法都是以AOP的形式来实现的。
> 

> 当我们想Hook一个OC类的某些具体的方法时可以通过Method Swizzling技术来实现，当我们想Hook动态库中导出的某个C函数时可以通过修改导入函数地址表中的信息来实现（可以通过开源库fishhook来完成）、当我们想Hook所有OC类的方法时则可以通过objc_msgSend

+ 2. Block的内部实现原理和实现机制简介

源码中定义的每个Block在编译时都会转化成一个和OC类对象布局相似的对象，每个Block也存在着isa这个数据成员，根据isa指向的不同，Block分为\_\_NSStackBlock、\_\_NSMallocBlock、\_\_NSGlobalBlock 三种类型，也就是说某种程度上Block对象也是一种OC对象，每个Block对象在内存中的布局，也就是Block对象的存储结构被定义如下（代码出自苹果开源的[libclosure](https://opensource.apple.com/source/libclosure/)）：


+ 3.实现Block对象Hook的方法和原理

```
	一个OC类对象的实例通过引用计数来管理对象的生命周期，在MRC时代当对象进行赋值和拷贝时需要通过调
	用retain方法来实现引用计数的增加，而在ARC时代对象进行赋值和拷贝操作时就不再需要显示调用
	retain方法了，而是系统内部在编译时会自动插入相应的代码实现引用计数的添加和减少，不管如何只要
	是对OC对象执行赋值拷贝操作，最终内部都会调用retain方法。
```

<h5 style="color:#f02b2b">Block对象也是一种OC对象</h5>

<span style="letter-space:1;color:red">每当一个Block对象在需要进行赋值或者拷贝操作时，也会激发对retain方法的调用，因为Block对象赋值操作一般是发生在Block方法之前，因此我们可以通过Method Swizzling时就不需要对NSObject的retainer方法执行替换，而只要对上述三个类的retain执行替换即可</span>

Block技术不仅可以用在OC语言中，LLVM对C语言进行的扩展也能使用Block,比如gcd库中大量的使用了Block,在C语言中如果对一个Block进行赋值或者拷贝系统需要通过C库函数：

```
//函数声明在Block.h头文件汇总
// Create a heap based copy of a Block or simply add a reference to an existing one.

// This must be paired with Block_release to recover memory, even when running
// under Objective-C Garbage Collection.

BLOCK_EXPORT void *_Block_copy(const void *aBlock)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_2);
    

```

来实现，这个函数定义在libsystem_blocks.dylib库中，并且库实现已经开源：[](libclosure)。因此可以借助fishhook库来对\_\_Block\_copy这个函数进行替换处理，然后在替换的函数中将一个Block的原始的invoke函数替换成统一的Hook函数。


[原文链接](https://juejin.im/post/5ca0ca6e51882567e32fc44b)













