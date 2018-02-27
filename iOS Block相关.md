
#### 1. block为什么要用Copy修饰

> 1.1 内存堆栈的理解
> 
> > 内存栈区：
> > 由编译器自动分配释放，存放函数的参数值，局部变量的值等，不需要程序员来操心，其操作方式类似于数据结构中的栈
> > 
> > 内存堆区：
> > 一般由程序员分配释放，若程序员不释放，程序结束时可能由OS回收，尽管后边苹果引入ARC机制，但是ARC的机制其实仅仅是帮助程序员添加了retain，release，autorelease代码，并不是说系统就可以自动管理了，它的系统管理的原理还是基于引用计数
> 
> 1.2 block的作用域
> > 首先，block是一个对象，所以block理论上是可以retain/release的。但是block在创建的时候它的内存是默认是分配在栈(stack)上，而不是堆(heap)上的。所以它的作用域仅限创建时候的当前上下文(函数, 方法...)，当你在该作用域外调用该block时，block占用的内存已经释放，无法进行访问，程序就会崩溃，出现野指针错误。

> 1.3 三种block
> > NSGlobalBlock:
> > 
> > 全局的静态block，没有访问外部变量，存储在代码区（存储方法或者函数）。他直到程序结束的时候，才会被被释放。但是我们实际操作中基本上不会使用到不访问外部变量的block。

```
void(^testOneBlock)() = ^(){
  	NSLog(@"我是全局的block");
  };
```
> > NSStackBlock:
> > 
> > 保存在栈中的block，没有用copy去修饰并且访问了外部变量，但是必须要在MRC的模式下控制台才会输出NSStackBlock类型。

```
//需要MRC模式
  int a = 5;
  void(^testTwoBlock)() = ^(){
  	NSLog(@"%d",a);
  };
```

> > NSMallocBlock:
> > 
> > 保存在堆中的block，此类型block是用copy修饰出来的block，它会随着对象的销毁而销毁，只要对象不销毁，我们就可以调用到堆中的block

```
int a = 5;
  self.block1 = ^(NSString *str, UIColor *color){
  	NSLog(@"%d",a);
  };
```
> 所以MRC下block修饰的原因是：将block的存储空间有堆区拷贝到栈区，避免在程序运行结束后，block所分配的存储空间被回收后，再访问block对象时由于出现野指针而崩溃,


--
#### 2. ARC下blcok用什么修饰
> 在ARC环境下，block使用copy或者strong修饰其实是一样的，因为blcok的retain就是用copy来实现的。
> 


--




