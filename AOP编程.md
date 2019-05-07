### 什么是AOP

> AOP: Aspect Oriented Programming, 译为面向切面编程。
> 

在不修改源码的情况下，通过运行时给程序添加统一功能的技术。

其有两层含义：

+ 第一：不修改源代码，即尽可能的解耦
+ 第二：添加统一的功能，即我们能实现的是添加统一的单一的功能，在某处使用AOP,我们只能实现一项单一的功能，

在iOS中实现AOP的核心技术是Runtime,使用Runtime的Method Swizzling黑魔法，我们可以在运行时hook函数，让函数执行一些副操作，为嵌入不同函数的功能做相同的操作，每类功能相同的操作可以抽取出一个切面。

+ OOP针对业务处理过程的实体及其属性和行为进行抽象封装，以获得更加清晰高效的逻辑单元划分。
+ AOP则是针对业务处理过程中的切面进行提取，它所面对的是处理过程中的某个步骤或阶段，以获得逻辑过程中各部分之间低耦合性的隔离效果。

使用runtime相关的方法method_exchangeImplementations即可交替两个select对应的函数IMP指针，具体应用实例是在某一个时刻，hook并替换掉方法的实现，从而从切面对控制器的生命周期进行干涉。但这种方法要求在编译时必须写好替换的函数，如果你需要在运行时获取一个类的method，并动态地进行重定向和包装参数，这个方法就显得局限了。

### AOP的技术实现

```
	id objc_msgSend(id self ,SEL op , ...){
		if(!self)return nil;
		IMP imp = class_getMethodImplementation(self->isa, SEL op);
		imp(self , op ,...); // 调用这个函数，伪代码
	}
	
	// 查找IMP
	IMP IMP class_getMethodImplementation(Class cls, SEL sel) {
		if(!cls || !sel)return nil;
		IMP imp = lookUpImpOrNil(cls, sel);
		if (!imp) return _objc_msgForward; //_objc_msgForward 用于消息转发
		return imp;
	}
```

当我们对一个类调用它无法识别的selector，会抛出unrecognized selector的异常，但是从runtime的伪代码中可以看到，当一个类根据selector找不到对应的IMP函数指针时，会返回\_objc\_msgForward这个函数指针，而并不是直接就抛出异常，而Aspects正是利用\_objc\_msgForward去进行方法重定向的。


#### Aspects的思路

当对一个类调用无法识别的消息selector时，其实并不是马上就会触发异常，而是会走进消息转发的流程。如下图：

![](https://upload-images.jianshu.io/upload_images/1252676-652eb55373c6b860.png?imageMogr2/auto-orient/)

#### 核心代码

```

// 核心方法 1.Hook forwardInvocation 到自己的方法
// 2.交换原方法的实现为_objc_msgForward 使其直接进入消息转发模式

static void aspect_prepareClassAndHookSelector(NSObject *self, SEL selector, NSError **error) {
    NSCParameterAssert(selector);
    // 1  swizzling forwardInvocation
    Class klass = aspect_hookClass(self, error);
    
    // // 被 hook 的 selector
    Method targetMethod = class_getInstanceMethod(klass, selector);
    IMP targetMethodIMP = method_getImplementation(targetMethod);
    // 判断需要被Hook的方法是否应指向 _objc_msgForward 进入消息转发模式
    if (!aspect_isMsgForwardIMP(targetMethodIMP)) {
        // Make a method alias for the existing method implementation, it not already copied.
        // 让一个新的子类方法名指向原先方法的实现，处理回调
        const char *typeEncoding = method_getTypeEncoding(targetMethod);
        SEL aliasSelector = aspect_aliasForSelector(selector);
        if (![klass instancesRespondToSelector:aliasSelector]) {
            __unused BOOL addedAlias = class_addMethod(klass, aliasSelector, method_getImplementation(targetMethod), typeEncoding);
            NSCAssert(addedAlias, @"Original implementation for %@ is already copied to %@ on %@", NSStringFromSelector(selector), NSStringFromSelector(aliasSelector), klass);
        }
 
        // We use forwardInvocation to hook in.
        // 把 selector 指向 _objc_msgForward 函数
        // 用 _objc_msgForward 函数指针代替 selector 的 imp,然后执行这个 imp  进入消息转发模式
        class_replaceMethod(klass, selector, aspect_getMsgForwardIMP(self, selector), typeEncoding);
        AspectLog(@"Aspects: Installed hook for -[%@ %@].", klass, NSStringFromSelector(selector));
    }
}
```

这里的核心代码，主要分为两部分

1、aspect_hookClass

```
// 动态生成子类 把子类的方法替换成自己的
static Class aspect_hookClass(NSObject *self, NSError **error) {
    NSCParameterAssert(self);
    // 如果Self是实例变量 class都是返回isa指针 都是AspectsViewController
	Class statedClass = self.class; //  1、对象 返回isa  2、class 返回本类
	Class baseClass = object_getClass(self); // 返回isa
	NSString *className = NSStringFromClass(baseClass);
 
    // Already subclassed
    // 是否有 _Aspects_ 后缀
	if ([className hasSuffix:AspectsSubclassSuffix]) {
		return baseClass;
 
        // We swizzle a class object, not a single object.
	}else if (class_isMetaClass(baseClass)) {
        return aspect_swizzleClassInPlace((Class)self);
        // Probably a KVO'ed class. Swizzle in place. Also swizzle meta classes in place.
    }else if (statedClass != baseClass) {
        return aspect_swizzleClassInPlace(baseClass);
    }
 
    // Default case. Create dynamic subclass.
    // 动态生成一个当前对象的子类，并将当前对象与子类关联,然后替换子类的 forwardInvocation 方法
	const char *subclassName = [className stringByAppendingString:AspectsSubclassSuffix].UTF8String;
	Class subclass = objc_getClass(subclassName);
 
	if (subclass == nil) {
        // 创建类AspectsViewController的子类   AspectsViewController_Aspects_  // 生成 baseClass 对象的子类
		subclass = objc_allocateClassPair(baseClass, subclassName, 0);
		if (subclass == nil) {
            NSString *errrorDesc = [NSString stringWithFormat:@"objc_allocateClassPair failed to allocate class %s.", subclassName];
            AspectError(AspectErrorFailedToAllocateClassPair, errrorDesc);
            return nil;
        }
        // 把子类的 forwardInvocation 指向 __ASPECTS_ARE_BEING_CALLED__ // 替换子类的 forwardInvocation 方法
		aspect_swizzleForwardInvocation(subclass);
        
        // 修改了 subclass 以及其 subclass metaclass 的 class 方法,使他返回当前对象的 class 隐藏对外的Class 类似KVO
		aspect_hookedGetClass(subclass, statedClass);
		aspect_hookedGetClass(object_getClass(subclass), statedClass);
		objc_registerClassPair(subclass);
	}
    
    // 将当前对象 isa 指针指向了 subclass
    // 将当前 self 设置为子类，这里其实只是更改了 self 的 isa 指针而已
	object_setClass(self, subclass);
	return subclass;
}
 
static NSString *const AspectsForwardInvocationSelectorName = @"__aspects_forwardInvocation:";
// 根据新创建的Self 子类AspectsViewController_Aspects_  //swizzling forwardinvation 方法
static void aspect_swizzleForwardInvocation(Class klass) {
    NSCParameterAssert(klass);
    // If there is no method, replace will act like class_addMethod.
    // 由于新创建的class没有实现 forwardInvocation  因此 这里的 originalImplementation就是空的
    // 使用 __ASPECTS_ARE_BEING_CALLED__ 替换子类的 forwardInvocation 方法实现
    // 由于子类本身并没有实现 forwardInvocation ，
    // 所以返回的 originalImplementation 将为空值，所以子类也不会生成 AspectsForwardInvocationSelectorName 这个方法
    IMP originalImplementation = class_replaceMethod(klass, @selector(forwardInvocation:), (IMP)__ASPECTS_ARE_BEING_CALLED__, "v@:@");
    if (originalImplementation) {
        class_addMethod(klass, NSSelectorFromString(AspectsForwardInvocationSelectorName), originalImplementation, "v@:@");
    }
    AspectLog(@"Aspects: %@ is now aspect aware.", NSStringFromClass(klass));
}
```

第一部分主要动态生成一个子类，然后把子类的forwardInvocation函数动态替换成**__ASPECTES_ARE_BEING_CALLED__**，然后通过**object_setClass**把self的isa指针指向subClass,这样后期调用的时候，会直接进入到动态创建的这个子类。



2. 交换方法的实现，使其直接进去**__msg_forward__**转发流程。

```
if (!aspect_isMsgForwardIMP(targetMethodIMP)) {
        // Make a method alias for the existing method implementation, it not already copied.
        // 让一个新的子类方法名指向原先方法的实现，处理回调
        const char *typeEncoding = method_getTypeEncoding(targetMethod);
        SEL aliasSelector = aspect_aliasForSelector(selector);
        if (![klass instancesRespondToSelector:aliasSelector]) {
            __unused BOOL addedAlias = class_addMethod(klass, aliasSelector, method_getImplementation(targetMethod), typeEncoding);
            NSCAssert(addedAlias, @"Original implementation for %@ is already copied to %@ on %@", NSStringFromSelector(selector), NSStringFromSelector(aliasSelector), klass);
        }
 
        // We use forwardInvocation to hook in.
        // 把 selector 指向 _objc_msgForward 函数
        // 用 _objc_msgForward 函数指针代替 selector 的 imp,然后执行这个 imp  进入消息转发模式
        class_replaceMethod(klass, selector, aspect_getMsgForwardIMP(self, selector), typeEncoding);
        AspectLog(@"Aspects: Installed hook for -[%@ %@].", klass, NSStringFromSelector(selector));
    }
```

第二部分**aspect_isMsgForwardIMP**是会提前判断原方法是否有被Hook走，有的话就不会再操作(这里和JSPatch会冲突),然后有两部操作，第一步吧动态生成的带有前缀的方法的IMP指向原方法的实现，第二步把原来函数的IMP指针指向objc_msgForward直接进入消息转发流程


最终Hook到以下函数，执行完之后回调到原来的方法实现，到此所有流程就结束了

```
	// This is the swizzled forwardInvocation: method.
static void __ASPECTS_ARE_BEING_CALLED__(__unsafe_unretained NSObject *self, SEL selector, NSInvocation *invocation) {
    NSCParameterAssert(self);
    NSCParameterAssert(invocation);
    SEL originalSelector = invocation.selector;
	SEL aliasSelector = aspect_aliasForSelector(invocation.selector);
    invocation.selector = aliasSelector;
    // 拿出对象的关联Aspects
    AspectsContainer *objectContainer = objc_getAssociatedObject(self, aliasSelector);
    // 拿出类关联的Aspects
    AspectsContainer *classContainer = aspect_getContainerForClass(object_getClass(self), aliasSelector);
    AspectInfo *info = [[AspectInfo alloc] initWithInstance:self invocation:invocation];
    NSArray *aspectsToRemove = nil;
 
    // Before hooks.
    aspect_invoke(classContainer.beforeAspects, info);
    aspect_invoke(objectContainer.beforeAspects, info);
 
    // Instead hooks.
    BOOL respondsToAlias = YES;
    if (objectContainer.insteadAspects.count || classContainer.insteadAspects.count) {
        aspect_invoke(classContainer.insteadAspects, info);
        aspect_invoke(objectContainer.insteadAspects, info);
    }else {
        Class klass = object_getClass(invocation.target);
        do {
            if ((respondsToAlias = [klass instancesRespondToSelector:aliasSelector])) {
                [invocation invoke];
                break;
            }
        }while (!respondsToAlias && (klass = class_getSuperclass(klass)));
    }
 
    // After hooks.
    aspect_invoke(classContainer.afterAspects, info);
    aspect_invoke(objectContainer.afterAspects, info);
 
    // If no hooks are installed, call original implementation (usually to throw an exception)
    if (!respondsToAlias) {
        invocation.selector = originalSelector;
        SEL originalForwardInvocationSEL = NSSelectorFromString(AspectsForwardInvocationSelectorName);
        if ([self respondsToSelector:originalForwardInvocationSEL]) {
            ((void( *)(id, SEL, NSInvocation *))objc_msgSend)(self, originalForwardInvocationSEL, invocation);
        }else {
            [self doesNotRecognizeSelector:invocation.selector];
        }
    }
 
    // Remove any hooks that are queued for deregistration.
    [aspectsToRemove makeObjectsPerformSelector:@selector(remove)];
}
```

该函数swizzle后，实现新IMP统一处理的核心方法，完成以下几件事情：

+ 处理调用逻辑，有before、instead、after、remove四种option

+ 将block转换成一个NSInvocation对象以供调用

+ 从AspectsContainer根据aliasSelector取出对象, 并组装一个AspectInfo, 带有原函数的调用参数和各项属性, 传给外部的调用者 (在这是block)

+ 调用完成后销毁带有removeOption的hook逻辑, 将原selector挂钩到原IMP上, 删除别名selector


### 思路总结

1. 找到被 hook 的 originalSelector 的 方法实现

2. 新建一个 aliasSelector 指向原来的 originalSelector 的方法实现

3. 动态创建一个 originalSelector 所在实例的子类，然后 hook 子类的 forwardInvocation: 方法并将方法的实现替换成**ASPECTS_ARE_BEING_CALLED**方法

4. originalSelector指向 **_objc_msgForward**方法实现

5. 实例的 originalSelector 的方法执行的时候，实际上是指向**objc_msgForward**，而 objc_msgForward的方法实现被替换成**ASPECTS\_ARE\_BEING\_CALLED**的方法实现，也就是说 originalSelector的方法执行之后，实际上执行的是**\_\_ASPECTS\_ARE\_BEING\_CALLED**的方法实现。而aliasSelector的作用就是用来保存 originalSelector 的方法实现，当 hook 代码执行完成之后，可以回到 originalSelector 的原始方法实现上继续执行


