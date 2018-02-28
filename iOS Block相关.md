
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


#### blcok循环引用的场景
> 首先并不是所有的block，使用self都会出现循环引用，其实不是，系统和第三方框架的block绝大部分不会出现循环引用，只有少数block以及我们自定义的block会出现循环引用，
> 	> 1.0 直接强引用
> 	>	> 由于block会对block中的对象进行持有操作，而如果此时block中的对象又持有了该block，则会造成循环引用。如下：

```
typedef void(^block)();

@property (copy, nonatomic) block myBlock;
@property (copy, nonatomic) NSString *blockString;

- (void)testBlock {
	self.myBlock = ^() {
    	//其实注释中的代码，同样会造成循环引用
    	NSString *localString = self.blockString;
    	//NSString *localString = _blockString;
    	//[self doSomething];
	};
}
```
> 以下调用注释掉的代码同样会造成循环引用，因为不管是通过self.blockString还是\_blockString，或是调用函数[self doSomething]，因为只要block中用到了对象的属性或者函数，block就会持有该对象而不是该对象的某个属性或者函数。
> 
> 	> 1.1 间接强引用：self->某个类->block->self
> 可能会产生控制器self永远都不会被释放掉产生常驻内存。


######在实际开发中的循环引用
> 使用通知（NSNotification），调用系统自带的Block，在blcok中使用self会发生循环引用
> 

#### 解决循环引用的办法:
---
+	一般性解决办法

```
__weak typeof(self) weakSelf = self; 
```

通过__weak的修饰，先把self弱引用，然后在block回调里面用weakSelf，这样就会打破循环，从而避免循环引用
> 
> \_\_blcok与\_\_weak都可以用来解决循环引用，但是，\_\_block不管是ARC还是MRC模式下都可以使用，可以修饰对象，还可以修饰基本数据类型，\_\_weak只能在ARC模式下使用，也只能修饰对象，不能修饰基本数据类型，\_\_block对象可以在block中被重新赋值，\_\_block不可以。

#### weak的缺陷
> 缺陷
> 如果我们想在Block中延时运行某段代码时，这里会出现一个问题，如一下代码：

```
 - (void)viewDidLoad {
  	[super viewDidLoad];
  	MitPerson*person = [[MitPerson alloc]init];
  	__weak MitPerson * weakPerson = person;
  	person.mitBlock = ^{
  		dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
      		[weakPerson test];
  		});
  	};
  	person.mitBlock();
  }
```
> 直接运行这段代码会发现[weakPerson test];并没有执行，打印一下会发现，weakPerson已经是Nil了，这是由于当我们的ViewDidLoad方法运行结束，由于是局部变量，无论是MitPerson和WeakPerson都会被释放掉，那么这个时候在Block中就无法拿到真正的person内容了。

解决办法：在block中对于可能会被提前释放的对象强引用，

深入理解：

+ 首先了解一些概念：

> 堆里面的block（被copy过的block）有以下现象：
> 
> 1. block内部如果通过外部声明的强引用来使用，那么block内部会自动产生一个强引用指向所使用的对象
> 2. block内部如果通过外部声明的弱引用来使用，那么block内部会自动产生一个弱引用执行所使用的对象。

+ 这段代码的目的：
	+ 首先，我们需要在Block块中调用，person对象的方法，既然是在Block块中我们就应该使用弱指针来引用外部变量，以此来避免循环引用，但是又会出现问题，就是当计时器要执行方法的时候，发现对象已经被释放掉了。
	+ 为了避免person对象在计时器执行的时候被释放掉，为什么person对象会被释放掉呢？因为无论我们的person强指针还是weakPerson弱指针都是局部变量，当执行完ViewDidLoad的时候，指针会被销毁，对象只有在被强指针执行的时候才不会被销毁，而如果我们直接引用外部的强指针对象又会产生循环引用
	+ 解决这个问题的办法是blcok引用外部的weakPerson，并在内部创建一个强指针去指向person对象，因为在内部声明变量，Block是不会强引用这个对象的，这也就避免person.mitBlcok出现循环引用风险的同时，又创建出一个强指针指向对象。
	+ 之后再用GCD延时器Block来引用相对于它来说是外部的变量strongPerson，这时延时器Block会默认创建出来一个强引用person对象，当person.mitBlcok作用域结束之后strongPerson会被跟着销毁，内存中就仅剩下了延时器Block强引用着的person对象，2秒之后触发test方法，GCD Block内部方法执行完毕之后，延时器和对象都被销毁，这样就完美实现我们的需求

