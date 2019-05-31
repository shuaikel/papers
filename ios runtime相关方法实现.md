<p style="color:#ee464d;font-size:18px;">1、运行时动态创建类</p>

```
- (Class)makeSubclassWithClass:(Class)origClass {
    NSString *className = NSStringFromClass(origClass);
    NSString *aopClassName = [kAOPFeedsViewPrefix stringByAppendingString:className];
    Class aopClass = NSClassFromString(aopClassName);

    if (aopClass) {
        return aopClass;
    }
    aopClass = objc_allocateClassPair(origClass, aopClassName.UTF8String, 0);
	
	// 添加方法、或者变量，在objc_registerClassPair之前
    [self setupAopClass:aopClass];

    objc_registerClassPair(aopClass);
    return aopClass;
}
```

<p style="color:#ee464d;font-size:18px;">2、交换两个方法的实现</p>

```
BOOL imyaop_swizzleMethod(Class clazz, SEL origSel_, SEL altSel_) {
    if (!clazz) {
        return NO;
    }
    Method origMethod = class_getInstanceMethod(clazz, origSel_);
    if (!origMethod) {
        return NO;
    }
    Method altMethod = class_getInstanceMethod(clazz, altSel_);
    if (!altMethod) {
        return NO;
    }

    class_addMethod(clazz,
                    origSel_,
                    class_getMethodImplementation(clazz, origSel_),
                    method_getTypeEncoding(origMethod));
    class_addMethod(clazz,
                    altSel_,
                    class_getMethodImplementation(clazz, altSel_),
                    method_getTypeEncoding(altMethod));

    method_exchangeImplementations(class_getInstanceMethod(clazz, origSel_), class_getInstanceMethod(clazz, altSel_));

    return YES;
}
```
<p style="color:#ee464d;font-size:18px;">3、动态关联对象</p>

```
static const void *kIMYAOPTableUtilsKey = &kIMYAOPTableUtilsKey;

- (IMYAOPTableViewUtils *)aop_utils {
    IMYAOPTableViewUtils *aopUtils = objc_getAssociatedObject(self, kIMYAOPTableUtilsKey);
    if (!aopUtils) {
        @synchronized(self) {
            aopUtils = objc_getAssociatedObject(self, kIMYAOPTableUtilsKey);
            if (!aopUtils) {
                objc_setAssociatedObject(self, kIMYAOPTableUtilsKey, aopUtils, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
            }
        }
    }
    return aopUtils;
}
```


<p style="color:#ee464d;font-size:18px;">4、通过消息转发调用父类的方法</p>

```
((void (*)(void *, SEL, id))(void *)objc_msgSendSuper)(&objcSuper,
 @selector(setDelegate:), self);
```

<p style="color:#ee464d;font-size:18px;">5、动态给一个类添加实例、类方法</p>

```
- (void)addOverriteMethod:(SEL)seletor toMethod:(SEL)toSeletor aopClass:(Class)aopClass {
    // 指定函数实现的类
    Class implClass = [self implAopViewClass];
    Method method = class_getInstanceMethod(implClass, toSeletor);
    if (method == NULL) {
        method = class_getInstanceMethod(implClass, seletor);
    }
    const char *types = method_getTypeEncoding(method);
    IMP imp = method_getImplementation(method);
    class_addMethod(aopClass, seletor, imp, types); // 类方法挂载在metaclass对象
}
```

<p style="color:#ee464d;font-size:18px;">6、改变方法的实现</p>

```
 * @param m The method for which to set an implementation.
 * @param imp The implemention to set to this method.
 method_setImplementation(<#Method  _Nonnull m#>, <#IMP  _Nonnull imp#>)
```




