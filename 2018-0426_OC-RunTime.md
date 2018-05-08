###Runtime 简介

[链接地址](https://halfrost.com/objc_runtime_isa_class/)

[mianshiti](https://juejin.im/post/5adaed6a518825673123c757?utm_source=gold_browser_extension)

[mianshiwentizonjie](https://juejin.im/post/5ad541e0f265da23994f032c?utm_source=gold_browser_extension)


Runtime 又叫运行时，是一套底层的C语言API，是iOS系统的核心之一。开发者在编码过程中，可以给任意一个对象发送消息，在编译阶段只是确定了要向接收者发送这条消息，而接收者将要如何响应和处理这条消息，那就要看运行时来决定了。

C语言中，在编译期，函数的调用就会决定调用哪个函数。
而OC的函数，属于 动态调用过程，在编译期并不能决定真正调用哪个函数，只有在真正运行时才会根据函数的名称找到对应的函数来调用。

Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态创建类和对象、进行消息传递和转发。

![](https://ob6mci30g.qnssl.com/Blog/ArticleImage/23_3.png)

1. 通过Objective-C 源代码

> 一般情况下开发者只需要编写OC代码即可，Runtime系统自动在幕后把我们写的源代码在编译阶段转换成运行时代码，在运行时确定对应的数据结构和调用具体哪个方法。

2. 通过Foundation 框架的NSObject类定义的方法

> 在OC的世界中，除了NSProxy类之外，所有的类都是NSObject的子类，在Foundation框架下，NSObject和NSProxy两个基类，定义了类层次结构中该类下方所有类的公共接口和行为。NSProxy是专门用于实现代理对象的类，这两个类都遵循了NSObject协议，在NSObject协议中，声明了所有OC对象的公共方法。
> 
> 在NSObject协议中，有以下5个方法，是可以从Runtime中获取信息，让对象进行自我检查。
> 
```
- (Class)class OBJC_SWIFT_UNAVAILABLE("use 'anObject.dynamicType' instead");
- (BOOL)isKindOfClass:(Class)aClass;
- (BOOL)isMemberOfClass:(Class)aClass;
- (BOOL)conformsToProtocol:(Protocol *)aProtocol;
- (BOOL)respondsToSelector:(SEL)aSelector;
```

从上述源码中，我们可以看到，Objective-C对象都是C语言结构体实现，