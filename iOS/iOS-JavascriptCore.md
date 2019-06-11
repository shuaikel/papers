+ JavascriptCore在iOS7中，apple用Objective-C进行了封装。JavascriptCore允许我们在不使用UIWebView和WKWebView的情况下去执行Javascript代码，而Javascript作为web开发的脚本语言，在跨平台的应用上，有着很重要的作用，许多跨平台的方案，比如JSPatch、React-Native都是基于JavascriptCore作为iOS端的实现方式。

##### JavascriptCore的库组成

在JavaScriptCore库文件中，可以看到其中导入了5个头文件

```
#import "JSContext.h"
#import "JSValue.h"
#import "JSManagedValue.h"
#import "JSVirtualMachine.h"
#import "JSExport.h"
```
对比这五个类，简单总结如下：

+ JSContext是JavaScriptCore的运行上下文，是执行JavascriptCore代码和注册native方法接口的执行环镜。

+ JSValue是JSContext执行后的返回结果，可以是任何Javascript类型，JSValue提供很多的方法进行Javascript和Objective-C类型的转换，但是需要注意的是JSValue是一个强引用类型，Javascript内部通过垃圾回收机制进行内存管理。Javascript和Objective-C类型的转换可以参考JSValue提供的方法。
+ JSManagedValue是JSValue的封装，用它可以解决Javascript和native代码之间循环引用的问题。
+ JSVirtualMachine管理Javascript运行时和管理Javascript暴露的native对象的内存，一般情况下是不需要直接和这个类交流的，但是如果你需要并行执行Javascript代码，JSContext就需要运行在不同的JSVirtualMachine之上，
+ JSExport是一个协议，通过实现它可以完成把一个native对象暴露给Javascript


#####Objective-C使用Javascript

> JavaScript没有类的概念，可以通过原型继承的方式实现继承，Objective-C可以调用Javascript的函数和Javascript属性。这些函数和属性需要现已NSString的形式加载到JSContext环境上去，执行成功后，如果有返回值，返回一个JSValue对象，一般情况下，Javascript文件都会存在本地，打包进你的程序中去，当然也可以从网络中获取Javascript文件，加载到context中去，这就是我们讲的热更新
>
>

###### 函数和属性

```
现有如下Javascript代码
// jscore.js
var jsGlobalVar = "JS Demo";
function min(a,b){
	return a-b;
};

那么这个min方法就可以在Objective-C中以类似Key-Value的形式被检索到，这个value就是一个JSValue，然后调用JSValue实例的callWithArguments传入Objective-C类型的参数，在Javascript环境中进行相应的类型转换。

//JSCoreViewController.m
  NSString *jsCode = [self readFile]; // jscore.js
    [self.context evaluateScript:jsCode]; //将上述min函数加载到context环境中去 
    JSValue *min = [self.context[@"min"] callWithArguments:@[@2,@4]]; // self.context[@"min"] 对应的就是js中的 min函数或者min属性
    JSValue *jsVar = self.context[@"jsGlobleVar"];
     NSLog(@"jsGlobleVar-----%@",[jsVar toString]);// "JS Demo"
    NSLog(@"min+++++%d",[min toInt32]); //-2

```

##### Javascript调用Objective-C

Javascript调用Objective-C的block
如下代码，向context注册了key为multi的JSValue对象，
```
 self.context[@"multi"] = ^(NSIntger a, NSInteger b){
 	return a*b;
 };
 
 相当于Javascript中定义一个如下的函数
 function multi(a,b){
 	return a*b;
 }
 
那么在context调用这个block的方式和Objective-C中调用Javascript的形式是一致的

JSValue *multi = [self.context[@"multi"] callWithArguments:@[@3,@4]];
// 或者
multi = [self.context evaluateScript:@"multi(10,3)"];
```

##### 自定义类型和方法

除了使用JSContext下标方法暴露Javascript对象之外，还可以使用JSExport协议把Objective-C的自定义类型转换为JSValue，并暴露给Javascript对象，在Javascript的环境中操作这个Objective-C对象。
首先需要定义一个遵循JSExport的子协议，在子协议中规定了哪些属性和方法可以在Javascript环境中可用。由于Javascript中函数参数没有类型，所以所有的Objective-C方法都会以驼峰命名的方式被调用，比如

```
-(void)minuse:(NSInteger)a b:(NSInteger)b c:(NSInteger)c;将以minusBC(a,b,c)的方式被调用。也可以通过JSExportAs宏重定义在Javascript中被调用的方法
```

总结一下：JSExport主要功能有

1. 将Objective-C的函数和属性传给Javascript
2. @property---> JavaScript的getter、setter
3. Objective-C的实例方法----> Javascript函数 Objective-C
4. Objective-C类方法-----> 在全局类对象上的Javascript函数


+ 传入一个Objective-C对象

先定义如下的Objective-C协议，并创建一个遵循这个协议的类（这里使用JSExportModel来定义）

```
@protocol JSModelProtocol<JSExport>

-(void)minuse:(NSInteger)a b:(NSInteger)b c:(NSInteger)c;
@property (assign , nonatomic)NSInteger sum;
@end
```
接下来就可以将遵循这个协议的类JSExportModel的实例对象加到执行环境中去，并调用在Javascript中已经被加载到context中的useOCObject方法。

```
- (void)excuteOCCodeInJS{
	JSExportModel *model = [JSExportModel new];
	self.context[@"model"] = model;
	[self.context[@"useOCObject"] callWithArguments:nil];
	NSLog(@"%ld",(long)model.intV); // 14
}
```

useOCObject的Javascript代码如下：
```
function useOCObject(){
	model.minuseBC(100,12,12);
	model.intV = 14;
}
```
这里调用了Objective-C的对象的方法，并对传入的对象的属性进行了修改。


##### 传入一个类对象
因为Javascript没有类的概念，所有传入到Javascript环境中的Objective-C类就变为了一个Constructor对象，如果你需要在Javascript环境中生成一个对象，你需要使这个Objective-C类有个类方法来生成实例对象，即在之前的协议中，我们先定义一个类方法用来生成实例对象。

```
// 在协议中声明，来暴露给JavaScript
+ (instancetype) createWithIntV:(NSInteger)value;
```
然后在Objective-C中进行如下调用
```
-(void)excuteOCClassInJS{
    self.context[@"Model"] = [JSExportModel class];
   JSValue *returned = [self.context[@"useOCClass"] callWithArguments:nil];
   JSExportModel *m= [returned toObjectOfClass:[JSExportModel class]];
    NSLog(@"%ld",(long)m.intV); //12
}
```

JavaScript代码如下所示：
```
function useOCClass(){
	var m = Model.createWithIntV(12);
	m.minuseBC(10,1,1); // 调用Objective-C方法
	return m
}
可以看到我们，上述Objective-C代码打印出了12
```

##### 返回一个Javascript函数
在Javascript中函数也是变量，是一等公民，也可以被返回，在Javascript文件中有个JavascriptFunc函数，这个函数如下所示：

```
function callback(){
	// 这里打印的东西可以在safari浏览器的开发者选项中打开
	console.log("method-----")
}

function jsFunc(){
	Obj.jsValue = callback // 直接对变量赋值
	return callback; // 将function 以callback的形式返回
}

这个函数就是将callback函数返回给Objective-C对象，上述第一种是将函数对象直接赋值给Objective-C对象，第二种直接返回函数对象，可以在Objective-C中用如下方式调用函数：

- (void)jsReturnBlock{
	self.obj.jsValue = [self.context[@"jsFunc"] callWithArguments:nil];
	
	[self.obj.jsValue callWithArguments:nil];
	
	self.context[@"Obj"] = self.obj;
	
	[self.context[@"jsFunc"] callWithArguments:nil];
	
	[self.obj.jsValue callWithArguments:nil];
}

```
##### 内存管理

我们知道Objective-C使用ARC进行内存管理，JavaScriptCore(virtualMechine)内部通过垃圾回收机制进行内存管理，所有的引用都是强引用，JavascriptCore内部保证了大多数的内存管理都是自动进行的，不需要我们进行额外的内存管理，但是还有两种情况下的内存管理需要注意：

1. 在Objective-C对象中存储Javascript的值。
2. 将Javascript区域（主要是函数）加到Objective-C对象上去。

```
self.jaValue = [JSValue new];
self.context[@"block"] = ^(){
	JSValue *value = self.jsValue;
	NSLog(@"%@",value);
};
```
这里和普通的block对象造成的循环引用类似，self.context引用self，而本身self保有context。这种情况下，其实编译器也会告诉你这里产生了循环引用，最好的方法就是将这个jaValue作为block的参数传到Javascript环境中去。

还有一种情况就是在block中需要使用JS中其他对象或者函数，需要从当前的JSContext中获取，这时候就造成了循环引用，推荐的方式是通过+[JSContext currentContext];来获取当前的context，即

```
self.jsValue = [JSValue new];

self.context[@"block"] = ^(){
	// 循环引用
	// context = self.context;
	JSContext *context = [JSContext currentContext];
	NSLog(@"%@",value);
};
```

还有一种情况是你必须要用Objective-C对象保存一个Javascript的值或者函数，你必须使用JSManagedValue去弱引用Javascript对象，JSManagedValue对象加入了垃圾回收的引用引用，那么这个函数的意思就是Javascript的垃圾回收机制观察引用Objective-C对象ower，如果存在，那么它就不回收这个对象，否则就回收这个对象。


##### 线程
JSVirtualMachine保证了Javascript代码执行的线程安全，锁都是JSVirtualMachine来控制，使用不同的JSVirtualMachine来实现串行或并发。

##### JavascriptCore 和 UIWebView

在UIWebView中能够调用Javascript代码，其基本上还是UIWebView将Javascript代码在其内部的JSContext上执行，可以在UIWebView加载完成后调用，获取到这个_context。

```
-(void)webViewDidFinishLoad:(UIWebView *)webView{
     _context = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    [_context evaluateScript:@"showWebViewAlert()"];

}
```

##### JavascriptCore 和 WKWebView

WKWebView 和 JavascriptCore之间的通信，可以通过WKWebView的代理来实现，在js代码中，可以通过发送如下消息：
```
window.webkit.messageHandlers.AppModel.postMessage({body: 'call js alert in js'});
```
WKWebView 只需要在启动的时候绑定了AppModel就可以接收到JS发送过来的消息了，具体如下

```
WKUserContentController *contentVC = [WKUserContentController new];

// 这样就可以通过代理得到js发送来的消息了
[contentVC addScriptMessageHandler:self name:@"AppModel"];
config.userContentController = contentVC;
```
这样就可以WKScriptMessageHandler收到消息了

```
- (void)userContentCpntroller:(WKUserContentConroller *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
	// 收到消息
	NSLog(@"%@",message.body);
}
```


+ 如何设计出优雅的多版本api接口，这个问题在于通过什么方式传递api的版本号数据，主要有这么几种：

1. api版本号放在url路径中 www.xxx.com/v1.api.xxx
2. api版本号放在url参数中 api.www.com/api?version=v1&..
3. api版本号放在请求的header中
4. api版本号放在二级域名中。v1.api.xxx.com

远程门面模式：
对于一个应用来说，可能会涉及到一些第三方的接口，那么这些第三方的api接口为了满足api的多版本设计（第三方的api接口也常常会有所变动），可以考虑使用远程门面模式，即app不直接访问第三方的api，而是通过服务端api间接访问第三方api。

这样做的好处有：
1. 第三方的api可以纳入版本管理，并且在第三方接口变动时，可以通过修改api来应对，免去修改app导致重新审核的问题。
2. 保证接口的稳定性和统一性（比如api的接口是rest json），但第三方的接口可能是xml，soap等，这样可以通过服务端统一封装接口，而app统一调用api的rest json接口
3. 简化客户端逻辑和代码，这个符合客户端尽可能轻的开发准则

服务定位器：
多版本api策略对于客户端来说，api请求的地址可以直接写在app代码中，也可以通过服务定位器，去动态获取服务的url地址，使用服务定位器可以使多版本api策略更加灵活，可以考虑在每次启动app的时候调用服务定位器的接口。














