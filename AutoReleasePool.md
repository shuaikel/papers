#### AutoreleasePool对象什么时候释放：

> 新建一个XCode项目，将项目调整成MRC,Target -> Build Sttings -> All -> 搜索‘automatic’ -> 把 Objective-C Automatic Reference Counting 设置为 NO
> 
> 


在MRC中，需要使用retain、release、autorelease手动管理内存，如下代码：

```
int main(int argc, const char * argv[]) {
	@autoreleasepool {
	NSLog(@"****A***");
	Person *p = [[Person alloc]init];
	[p release];
	NSLog(@"***B***");
	}
	NSLog(@"***C***");
	return 0;
}
```

打印结果：

![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%BA%95%E5%B1%82/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/AutoreleasePool.png)

如果使用autorelease，就需要用到自动缓存池，代码如下：

```
int main(int argc, const char * argv[]) {
	@autoreleasepool {
	NSLog(@"****A***");
	Person *p = [[[Person alloc]init] autorelease];
	NSLog(@"***B***");
	}
	NSLog(@"***C***");
	return 0;
}
```
 打印结果：
 
 ![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%BA%95%E5%B1%82/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/AutoreleasePool1.png)
 
 
 **AutoreleasePool具体做了什么呢？ 我们首先查看**AutoreleasePool的实现原理
 
 通过:
 
``` 
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m 命令将 main.m 转成 C++ 
代码
```
 
我们可以找到AutoreleasePool的实现：
  

```

int main(int argc, const char * argv[]) 
{

/* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
	Person *p = ((Person *(*)(id, SEL))(void *)objc_msgSend)((id)((Person *(*)(id, SEL))(void *)objc_msgSend)((id)((Person *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("Person"), sel_registerName("alloc")), sel_registerName("init")), sel_registerName("autorelease"));
}
	return 0;
}

```
会发现@autoreleasepool会被转成：

```
__AtAutoreleasePool __autoreleasepool;

```

而\_\_AtAutoreleasePool我们全局查看发现它是一个结构体：

```
struct __AtAutoreleasePool {
	__AtAutoreleasePool() {atautoreleasepoolobj = objc_autoreleasePoolPush();}
	
	~__AtAutoreleasePool() {objc_autoreleasePoolPop(atautoreleasepoolobj);}
	void * atautoreleasepoolobj;
};
```

下面这段代码构造函数，在创建结构体的时候调用：

```
__AtAutoreleasePool() { // 构造函数，在创建结构体的时候调用
	atautoreleasepoolobj = objc_autoreleasePoolPush();
}
```

下面这段代码析构函数，在结构体销毁的时候调用：

```
~__AtAutoreleasePool() { // 析构函数，在结构体销毁的时候调用
	objc_autoreleasePoolPop(atautoreleasepoolobj);
}

```

```
@autoreleasepool {
	Person *p = [[[Person alloc]init] autorelease];
}
```

上面这段代码其实就是这个样子

```
atautoreleasepoolobj = objc_autoreleasePoolPush();
Person *person = [[[Person alloc] init] autorelease];
objc_autoreleasePoolPop(atautoreleasepoolobj);
```


**AutoreleasePoolPage**

对于objc_autoreleasePoolPush和objc_autoreleasePoolPop 的实现我们可以在runtime源码中查找相关实现

```
objc_autoreleasePoolPush(void)
{
	return AutoreleasePoolPage::push();
}

void
objc_autoreleasePoolPop(void *ctxt)
{
	AutoreleasePoolPage::pop(ctxt);
}
```


我们可以发现：push（）函数和pop（ctxt）函数都是有AutoreleasePoolPage类来调用的。


对于AutoreleasePoolPage类，我们查看成员变量，对于一些静态常亮我们就不过多的探究，我们就来查看一下成员变量。

```
class AutoreleasePoolPage 
{
	magic_t const magic;//用于数据校验
	id *next;//栈顶地址
	pthread_t const thread;//所在的线程
	AutoreleasePoolPage * const parent;//父对象
	AutoreleasePoolPage *child;//子对象
	uint32_t const depth;//page的序号？
	uint32_t hiwat;
	// ...
}
```

+ 1、 每个AutoreleasePoolPage对象占用4096字节内存，除了用来存放它内部的成员变量，剩下的空间用来存放autorelease对象的地址

+ 2、 所有的AutoreleasePoolPage对象通过双向链表的形式连接在一起


+ 3、 调用push方法会将一个POOL_BOUNDARY入栈，并且返回其存放的内存地址

+ 4、调用pop方法时传入一个POOL_BOUNDARY的内存地址，会从最后一个入栈的对象开始发送release消息，直到遇到这个POOL_BOUNDARY

+ 5、id *next指向了下一个能存放autorelease对象地址的区域

+ 6、AutoreleasePoolPage 空间被占满时，会以链表的形式新建链接一个 AutoreleasePoolPage 对象，然后将新的autorelease对象的地址存在child指针

#### push()函数实现

```
static inline void *push() 
{
	id *dest;
	if (DebugPoolAllocation) {
	// Each autorelease pool starts on a new pool page.
	dest = autoreleaseNewPage(POOL_BOUNDARY);
	} else {
	dest = autoreleaseFast(POOL_BOUNDARY);
	}
	assert(dest == EMPTY_POOL_PLACEHOLDER || *dest == POOL_BOUNDARY);
	return dest;
}

1、 在DebugPoolAllocation线程池满了以后，会调用autoreleaseNewPage(POOL_BOUNDARY)来创建一个新的线程池。

2、线程池没有满的时候调用autoreleaseFast函数，以栈的形式压入线程池中。

```

```
static inline id *autoreleaseFast(id obj)
{
	AutoreleasePoolPage *page = hotPage();
	if (page && !page->full()) {
	return page->add(obj);
	} else if (page) {
	return autoreleaseFullPage(obj, page);
	} else {
	return autoreleaseNoPage(obj);
	}
}

1、有 hotPage 并且当前 page 不满，调用 page->add(obj) 方法将对象添加至
 AutoreleasePoolPage 的栈中
 
2、有 hotPage 并且当前 page 已满，调用 autoreleaseFullPage 初始化一个新的页，调用 page-
>add(obj) 方法将对象添加至 AutoreleasePoolPage 的栈中

3、无 hotPage，调用 autoreleaseNoPage 创建一个 hotPage，调用 page->add(obj) 方法将对象
添加至 AutoreleasePoolPage 的栈中

```

#### pop函数（）函数

```
// 简化后
static inline void pop(void *token) 
{   
	AutoreleasePoolPage *page;
	id *stop;
	page = pageForPointer(token);
	stop = (id *)token;
// 1.根据 token，也就是上文的占位 POOL_BOUNDARY 释放 `autoreleased` 对象
	page->releaseUntil(stop);

// hysteresis: keep one empty child if page is more than half full
// 2.释放 `Autoreleased` 对象后，销毁多余的 page。
	if (page->lessThanHalfFull()) {
	page->child->kill();
	}else if (page->child->child) {
	page->child->child->kill();
	}
}
```

来到releaseUntil(...)内部：

```
// 简化后
void releaseUntil(id *stop) 
{
	// 1.
	while (this->next != stop) {
		AutoreleasePoolPage *page = hotPage();
		// 2.
		while (page->empty()) {
		page = page->parent;
		setHotPage(page);
		}
		// 3.
		if (obj != POOL_BOUNDARY) {
		objc_release(obj);
		}
	}
	// 4.
	setHotPage(this);
}

```
+ 外部循环挨个遍历 autoreleased 对象，直到遍历到 stop 这个 POOL_BOUNDARY 。

+ 如果当前 hatPage 没有 POOL_BOUNDARY，将 hatPage 设置为父节点。

+ 给当前 autoreleased 对象发送release消息。

+ 再次配置 hatPage。


```
	RunLoop的状态枚举：
	
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
	kCFRunLoopEntry = (1UL << 0),              // 1
	kCFRunLoopBeforeTimers = (1UL << 1),       // 2
	kCFRunLoopBeforeSources = (1UL << 2),      // 4
	kCFRunLoopBeforeWaiting = (1UL << 5),      // 32
	kCFRunLoopAfterWaiting = (1UL << 6),       // 64
	kCFRunLoopExit = (1UL << 7),               // 128
	kCFRunLoopAllActivities = 0x0FFFFFFFU
};

+ 0x1 （等于1）对应的是kCFRunLoopEntry，第一个 Observer 监视的即将进入Loop时，其回调内会调
用 _objc_autoreleasePoolPush() 创建一个自动释放池。其 order 是 -2147483647，优先级最高
保证创建缓存池发生在其他所有回调之前。


+ 0xa0（16进制等于160，等于32+128） 对应的是 kCFRunLoopBeforeWaiting&kCFRunLoopExit
第二个 Observer 监视了两个事件： 准备进入休眠时调用 _objc_autoreleasePoolPop() 和 
_objc_autoreleasePoolPush() 释放旧的池并创建新池；即将退出Loop时调用
_objc_autoreleasePoolPop()来释放自动释放池。这个 Observer 的 order 是 2147483647，优先
级最低，保证其释放缓存池发生在其他所有回调之后。

```


#### 具体步骤

+ iOS在主线程的Runloop中注册了2个Observer

+ 第1个Observer监听了kCFRunLoopEntry事件，会调用objc_autoreleasePoolPush()

+ 第2个Observer监听了kCFRunLoopBeforeWaiting事件，会调用objc_autoreleasePoolPop()、objc_autoreleasePoolPush() 监听了kCFRunLoopBeforeExit事件，会调用objc_autoreleasePoolPop()

autoreleased 对象是在 runloop 的即将进入休眠时进行释放的












