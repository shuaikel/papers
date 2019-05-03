```
	void (^block)(void) = ^(){
	    int a = 100;
    	 static int b = 1000;
        NSLog(@"this is a block!:%d===%d",a,b);
        c = @"ads";
        NSLog(@"block capture c :%@",c);
    };
    
    // 在项目.m中写入上面上面的block代码块，用编译命令行命令：xcrun  -sdk  iphoneos  clang  -arch 
     arm64  -rewrite-objc main.m，编译.m文件，执行结束后，会生成main.cpp 文件，拉倒最下面可以看到
     main函数的实现的。
    
    我们得到c++代码的block实现：
    
    int a = 100;
    static int b = 1000;
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, 
    &__main_block_desc_0_DATA, a, &b));
    
    我们知道（void*）这种类型的都是类型的强制转换，为了更好的识别我们的这个block代码，我们把类型转化去掉
    
    void（*block）（void） = &__main_block_impl_0(__main_block_func_0 , &__mian_block_desc_0_DATA));
    
    我们在分别查询__main_block_impl_0 , __mian_block_func_0 , __main_block_desc_0_DATA分别代表
    什么意思：
    
    (1)struct __main_block_impl_0{
    		struct __block_impl impl; 
    		struct __main_block_desc_0 *Desc;
    		// 构造函数
    		__main_block_impl_0(void *fp , struct __mian_block_desc_0 *desc,int flag){
    			impl.isa = &_NSConcreteStackBlock;
    			impl.Flags = flags;
    			impl.FuncPtr = fp;
    			Desc = desc;
    		}
    }
    
    (2)struct __block_impl{
    	void *isa;
    	int flag;
    	int Reserved;
    	void *FuncPtr;
    }
    
    (3)struct __main_block_desc_0{
    	size_t reserved;
    	size_t Block_size; // 内存大小描述
    }__main_block_desc_0_DATA;
    
   	所以我们可以总结：
    + 1. __main_block_impl_0 中 __block_impl存放的是一些变量信息，其中存在isa，所以可以判断block的本质其实就是oc对象。
    + 初始化

    再来查看block方法。
    
    void (*block)(void) = &__main_block_impl_0(__main_block_func_0,
	&__main_block_desc_0_DATA));
	
	对应上面的初始化我们可以看出第一个参数传递的是：执行方法，第二个参数为：描述信息
	
	成员变量的捕获：
	为了保证block内部能正常访问外部变量，block有个变量捕获机制
```
<img src="https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/images/Block2.png"></img>

```
	
```


Block 类型：

+ block有三种类型，可以通过调用class方法或者isa指针查看具体的类型，但是最终都是继承NSBlock类型：

 + 1. NSGlobalBlock ,没有访问auto变量
 + 2. NSStcakBlock 访问了auto变量
 + 3. NSMallocBlock, \_\_NSStackBlock\_\_ 调用了copy方法。

 auto变量修饰符\_\_weak
 
 + 1. 当Block内部访问了auto变量时，如果block是在栈上，将不会对auto变量产生强引用。

 + 2. 如果block被拷贝到堆上，会根据auto变量的修饰符（__strong,__weak,__unsafe_unretained）,对auto变量进行强引用或者弱引用。

 + 3.如果block从堆上移除的时候，会调用block内部的dispose函数，该函数自动释放auto变量。

 + 4.在多个block相互嵌套的时候，auto属性的释放取决于最后的那个强引用什么时候释放。

  



	

#### 1. block为什么要用Copy修饰

> 1.1 内存堆栈的理解
> 
> > 内存栈区：
> > 由编译器自动分配释放，存放函数的参数值，局部变量的值等，栈空间分静态分配和动态分配两种，静态分配是编译器完成，比如自动变量（auto）的分配，动态分配由alloca函数完成,不需要程序员来操心，其操作方式类似于数据结构中的栈
> > 
> > 内存堆区：
> > 一般由程序员分配释放，若程序员不释放，程序结束时可能由OS回收，尽管后边苹果引入ARC机制，但是ARC的机制其实仅仅是帮助程序员添加了retain，release，autorelease代码，并不是说系统就可以自动管理了，它的系统管理的原理还是基于引用计数
> > 
> > 全局区（静态区）（static）全局变量和静态变量的存储是放在一起的，初始化的全局变量和静态变量存放在一块区域，未初始化的全局变量和静态变量在相邻的另一块区域，程序结束后由系统释放。
> > 

总结： 

1、因为自动变量（auto）分配的内存空间在栈区（stack），编译器会自动帮我们释放，如果我们把block写在内外一个方法中调用，自动变量age就会被释放，block在使用的时候就已经被释放了，所以需要重新copy一下，

2、静态变量在程序结束后由系统释放，所以不需要担心被释放，block只需要知道他的内存地址就行。

3、对于全局变量，任何时候都可以直接释放，所以根本不需要捕获


> 
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


+ __Block的内存管理原则

 > 1. 当Block在栈上的时候，并不会对__Block 变量进行强引用
 
 > 2. 当block被copy到堆时，会调用block内部的copy函数，copy函数内部会调用_Block_object_assign函数,_Block_object_assign函数会对__block变量形成强引用（retain）
 
 > 3. 当block从堆中移除时，会调用block内部的dispose函数，dispose函数内部会调用_Block_object_dispose函数，_Block_object_dispose函数会自动释放引用的__block变量（release)
 
 总结：
 
+ 在ARC环境下，Block被引用的时候，会被Copy一次，由栈区copy到堆。

+ 在Block被copy的时候，Block内部被引用的变量也同样被copy一份到堆上面。

+ 被__BlocK修饰的变量，在被Block引用的时候，会变成结构体也就是oc对象，里面的\_\_forwarding也会由栈copy到堆上面。

+ 栈上\_\_block变量结构体中\_\_forwaring指针指向堆上面\_\_block变量结构体，堆上\_\_block变量结构体中\_\_forwaring指针指向自己。

+ 当block从堆中移除时，会调用block内部的dispose函数，dispose函数内部会调用\_Block\_object\_dispose函数，\_Block\_object\_dispose 函数会自动释放引用的\_\_block变量。


### 解决循环引用的问题

block 内引用的自动变量为OC对象的时候，在没有修饰符修饰的时候Block内部会强引用OC对象，而对象如果也持有Block的时候就会产生相互作用，也就是循环引用的问题。

我们也只能在Block持有OC对象的时候，给OC对象添加弱引用修饰符才比较合适，有两个弱引用修饰符\_\_weak和\_\_unsafe_unretained;

+ 1. \_\_weak ： 不会产生强引用，指向的对象销毁时，会自动让指针置为nil；
+ 2. \_\_unsafe\_unretained: 不会产生强引用，不安全，指向的对象销毁时，指针存储的地址值不变。

其实还有一种解决办法，那就是使用\_\_Block,需要在Block内部把OC对象置为nil；
