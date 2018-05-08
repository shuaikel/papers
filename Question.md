```
@property (nonatomic, strong) NSString *strongString;
@property (nonatomic, weak)   NSString *weakString;

strongString =  [NSString stringWithFormat:@"%@",@"string1"];
weakString =  strongString;
strongString = nil;

NSLog(@"%@", weakString);

结果会输出 “string1”
: 首先，stringWithFormat方法创建的字符串是autorelease的，本身就会延迟释放，直接log的话肯定不会输出null，然而，在64位环境下，苹果对NSString做了优化，具体表现为，当非字面值常量的数字，英文字母字符串长度小于等于9的时候会自动成为NSTaggedPointString类型，如果有中文或者其他特殊符号的话会直接成为__NSCFString类型。而NSTaggedPointString是常量释放不掉，所以，如果使用@“”或者initWithString：@“”的方式创建的字符串，会被转换成__NSCFConstantString,也是个常量，释放不掉不会输出null。
```

```
Block中可以修改全局变量，全局静态变量，局部静态变量吗？
:可以
：Blcoks是C语言的扩充功能，在iOS4中引入这个新功能，可以这样来形容Blocks，带有自动变量（局部变量）的匿名函数。
Block在OC中的实现如下：

struct Block_layout {
    void *isa;
    int flags;
    int reserved;
    void (*invoke)(void *, ...);
    struct Block_descriptor *descriptor;
    /* Imported variables. */
};

struct Block_descriptor {
    unsigned long int reserved;
    unsigned long int size;
    void (*copy)(void *dst, void *src);
    void (*dispose)(void *);
};

```
![](https://upload-images.jianshu.io/upload_images/1194012-1739b7e85e46b4db.png?imageMogr2/auto-orient/)


```
从结构图中很容易看到isa，所以OC处理Block是按照对象来处理的，在iOS中，
isa常见的就是_NSConcreteStackBlock，_NSConcreteMallocBlock，
_NSConcreteGlobalBlock这三种
```

###### 研究工具：
> 为了研究编译器的实现原理，我们需要使用clang命令。clang命令可以将Objective-C的源码改写成C/C++语言，借此我们可以研究block中各个特性的源码实现方式。
> 
```
> 该命令是：clang -rewrite-objc block.c 
```


C语言中变量有：
	
	+ 自动变量
	+ 函数参数
	+ 静态变量
	+ 静态全局变量
	+ 全局变量
研究Block的捕获外部变量就要除去函数参数这一项，下面一一根据这四种变量类型的捕获情况进行分析。

	+ 自动变量
	+ 静态变量
	+ 静态全局变量
	+ 全局变量

测试源码：

```
#import <Foundation/Foundation.h>

int global_i = 1;

static int static_global_j = 2;

int main(int argc, const char * argv[]) {
   
    static int static_k = 3;
    int val = 4;
    
    void (^myBlock)(void) = ^{
        global_i ++;
        static_global_j ++;
        static_k ++;
        NSLog(@"Block中 global_i = %d,static_global_j = %d,static_k = %d,val = %d",global_i,static_global_j,static_k,val);
    };
    
    global_i ++;
    static_global_j ++;
    static_k ++;
    val ++;
    NSLog(@"Block外 global_i = %d,static_global_j = %d,static_k = %d,val = %d",global_i,static_global_j,static_k,val);
    
    myBlock();
    
    return 0;
}
``` 
运行结果：

```
Block 外  global_i = 2,static_global_j = 3,static_k = 4,val = 5
Block 中  global_i = 3,static_global_j = 4,static_k = 5,val = 4
```

将源码用clang命令转换出来分析：

```
int global_i = 1;

static int static_global_j = 2;

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int *static_k;
  int val;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int *_static_k, int _val, int flags=0) : static_k(_static_k), val(_val) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int *static_k = __cself->static_k; // bound by copy
  int val = __cself->val; // bound by copy

        global_i ++;
        static_global_j ++;
        (*static_k) ++;
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_6fe658_mi_0,global_i,static_global_j,(*static_k),val);
    }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};


int main(int argc, const char * argv[]) {

    static int static_k = 3;
    int val = 4;

    void (*myBlock)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, &static_k, val));

    global_i ++;
    static_global_j ++;
    static_k ++;
    val ++;
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_6fe658_mi_1,global_i,static_global_j,static_k,val);

    ((void (*)(__block_impl *))((__block_impl *)myBlock)->FuncPtr)((__block_impl *)myBlock);

    return 0;
}
```
首先全局变量global\_i和静态全局变量static\_global_j的值增加，以及它们被Block捕获进去，这一点很好理解，因为是全局的，作用域很广，所以Block捕获了它们进去之后，在Block里面进行++操作，Block结束之后，它们的值依旧可以保存下来。

在__main_block_impl_0中，可以看到静态变量static_k 和 自动变量val,被Block从外面捕获进来，成为__main_block_impl_0 这个结构体的成员变量。

接着看构造函数：

```
__main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int *_static_k, int _val, int flags=0) : static_k(_static_k), val(_val)

```
这个构造函数中，自动变量和静态变量被捕获为成员变量追加到了构造函数中。

main里面的myBlock闭包中__main_block_impl_0结构体，初始化如下：

```
void (*myBlock)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, &static_k, val));


impl.isa = &_NSConcreteStackBlock;
impl.Flags = 0;
impl.FuncPtr = __main_block_impl_0; 
Desc = &__main_block_desc_0_DATA;
*_static_k = 4；
val = 4;

```

到此，__main_block_impl_0结构体就是这样把自动变量捕获进来，也就是说，在执行Block语法的时候，Block语法表达式所使用的自动变量的值是被保存进了Block的结构体实例中，也就是Block自身中。

这里值得说明的一点是，如果Block外面还有很多自动变量、静态变量、等等，这些变量在Block里面并不会被使用到，那么这些变量并不会被Block捕获进来，也就是说并不会在构造函数里面传入它们的值。

Block捕获外部变量仅仅只捕获Block闭包里面会用到的值，其他用不到的值，它并不会去捕获。

在研究一下源码，我们注意到__main_block_func_0 这个函数的实现：

```
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int *static_k = __cself->static_k; // bound by copy
  int val = __cself->val; // bound by copy

        global_i ++;
        static_global_j ++;
        (*static_k) ++;
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_6fe658_mi_0,global_i,static_global_j,(*static_k),val);
    }
```

可以发现，系统自动给我们加上的注释，bound by copy，自动变量val虽然被捕获进来了，但是是用__cself->val来访问的。Block仅仅捕获val的值，并没有捕获val的内存地址，所以在__main_block_func_0这个函数中即使我们重写这个自动变量val的值，也不能去改变Block外面自动变量val的值。

小结一下：
到此为止，上面提出的第二个问题就可以解释了，自动变量是以值传递方式传递到Block的构造函数里面去的，Block只捕获Block中会用到的变量，由于只捕获了自动变量的值，并非内存地址，所以Block内部不能改变自动变量的值，Block捕获的外部变量可以改变值得是全局变量，静态全局变量，静态变量，

回到上面例子上面，4种变量里面只有静态变量，静态全局变量，全局变量这三种是可以在Block里面改变值得，仔细看源码，我们可以看出这三个可以改变值得原因。

```
1、 静态全局变量，全局变量由于作用域的原因，
于是可以直接在Block里面被改变。他们也都存储在全局区。

```

![](https://upload-images.jianshu.io/upload_images/1194012-cf406451fc813cdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

```
2、 静态变量传递给Block是内存地址值，所以能在Block里面直接改变值。
```
根据官方文档我们了解到，苹果要求我们在自动变量前加入__block关键字，就可以在Block里面改变外部自动变量的值了。

总结一下在Block中改变变量值有两种方式，一是传递内存地址指针到Block中，而是改变存储区的方式（__Block）

先来实验一下第一种方式，传递内存地址到Block，改变变量的值

```
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    
  NSMutableString * str = [[NSMutableString alloc]initWithString:@"Hello,"];
    
        void (^myBlock)(void) = ^{
            [str appendString:@"World!"];
            NSLog(@"Block中 str = %@",str);
        };
    
    NSLog(@"Block外 str = %@",str);
    
    myBlock();
    
    return 0;
}
控制台输出：
Block 外  str = Hello,
Block 中  str = Hello,World!

```
转换一下源码：

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  NSMutableString *str;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, NSMutableString *_str, int flags=0) : str(_str) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  NSMutableString *str = __cself->str; // bound by copy

            ((void (*)(id, SEL, NSString *))(void *)objc_msgSend)((id)str, sel_registerName("appendString:"), (NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_33ff12_mi_1);
            NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_33ff12_mi_2,str);
        }
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->str, (void*)src->str, 3/*BLOCK_FIELD_IS_OBJECT*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->str, 3/*BLOCK_FIELD_IS_OBJECT*/);}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};

int main(int argc, const char * argv[]) {
    NSMutableString * str = ((NSMutableString *(*)(id, SEL, NSString *))(void *)objc_msgSend)((id)((NSMutableString *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("NSMutableString"), sel_registerName("alloc")), sel_registerName("initWithString:"), (NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_33ff12_mi_0);

        void (*myBlock)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, str, 570425344));

    NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_33ff12_mi_3,str);

    ((void (*)(__block_impl *))((__block_impl *)myBlock)->FuncPtr)((__block_impl *)myBlock);

    return 0;
}
```
在\_\_main\_block\_func_0 里面可以看到传递的是指针，所以成功改变了变量的值。

+ Block的Copy 和 dispose

OC中的Block就分为一下三种：

> + _NSConcreteStackBlock : 只用到外部局部变量、成员属性变量，且没有强指针引用的block都是StackBlock。StackBlock的生命周期由系统控制， 一旦返回之后，就被系统销毁了。

> + _NSConcreteMallocBlock: 有强指针引用或copy修饰的成员属性引用的block会被复制一份到堆中成为MallocBlock，没有强指针引用即销毁，生命周期由程序员控制
> 
> + \_NSConcreteGlobalBlock：没有用到外界变量或只用到全局变量、静态变量的block为\_NSConcreteGlobalBlock,生命周期从创建到应用程序结束。
> 
> 


+ 3、Block中__block实现原理

普通非对象的变量,先来看看普通变量的情况。

```
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    
    __block int i = 0;
    
    void (^myBlock)(void) = ^{
        i ++;
        NSLog(@"%d",i);
    };
    
    myBlock();
    
    return 0;
}

```
将上述代码用clang转换成源码：

```
struct __Block_byref_i_0 {
  void *__isa;
__Block_byref_i_0 *__forwarding;
 int __flags;
 int __size;
 int i;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_i_0 *i; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_i_0 *_i, int flags=0) : i(_i->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_i_0 *i = __cself->i; // bound by ref

        (i->__forwarding->i) ++;
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_3b0837_mi_0,(i->__forwarding->i));
    }
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->i, (void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
int main(int argc, const char * argv[]) {
    __attribute__((__blocks__(byref))) __Block_byref_i_0 i = {(void*)0,(__Block_byref_i_0 *)&i, 0, sizeof(__Block_byref_i_0), 0};

    void (*myBlock)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_i_0 *)&i, 570425344));

    ((void (*)(__block_impl *))((__block_impl *)myBlock)->FuncPtr)((__block_impl *)myBlock);

    return 0;
}

```
从源码我们能发现，带有\_\_block的变量也被转化成了一个结构体\_\_Block\_byref\_i\_0,这个结构体有5个成员变量，第一个是isa指针，第二个是指向自身类型的\_\_forwarding指针，第三个是一个标记flag，第四个是它的大小，第五个是变量值，名字和变量名同名。

```
__attribute__((__blocks__(byref))) __Block_byref_i_0 i = {(void*)0,(__Block_byref_i_0 *)&i, 0, sizeof(__Block_byref_i_0), 0};

```
源码中初始化，\_\_forwarding指针初始化传递的是自己的地址。然而这里\_\_forwarding指针并没有指向自己。

地址不同就可以很明显的说明\_\_forwarding指针并没有指向之前的自己，那么\_\_forwarding指针指向哪里了呢！

Block里面的\_\_block的地址和Block的地址相差1052，我们可以很大胆的猜想，\_\_block现在也在堆上。

由第二章里面分析可以知道，堆上的Block会持有对象，我们把Block通过copy到了堆上，堆上也会重新复制一份Block，并且该Block也会继续持有该\_\_block，当Block释放的时候，\_\_block没有被任何对象引用，也会被释放销毁。

\_\_forwarding指针这里的作用就是针对Block，把原来\_\_forwarding指针指向自己，换成指向\_NSConcreteMallBlock上复制之后的\_\_block自己。然后堆上的变量的\_\_forwarding在指向自己，这样不管\_\_block怎么复制到堆上，还是在栈上，都可以通过（i->\_\_forwarding->i）来访问到变量值。
![](https://upload-images.jianshu.io/upload_images/1194012-5f5f486bab68191f.jpg?imageMogr2/auto-orient/)

所以在\_\_main\_block\_func\_0 函数里面就是写的（i->\_\_forwarding->i）

这里还有一个需要注意的地方，还是从例子说起：

```
//以下代码在MRC中运行
    __block int i = 0;
    NSLog(@"%p",&i);
    
    void (^myBlock)(void) = ^{
        i ++;
        NSLog(@"Block 里面的%p",&i);
    };
    
    
    NSLog(@"%@",myBlock);
    
    myBlock();

```

结果可以看到，block并没有复制到堆上，地址一直都在栈上，这与ARC环境不一样。

解释：
> ARC环境下，一旦Block赋值就会触发copy，__block就会copy带堆上，Block也是\_\_MallocBlock，ARC环境下也是存在\_\_NSStackBlock的时候，这种情况下，__Block就在栈上。
> 
> MRC环境下，只有copy，\_\_block才会被复制带堆上，否则，\_\_block一直都在栈上，block也只是\_\_NSStackBlock,这个时候\_\_forwarding指针就只指向自己了
> 
> 

对象的变量：

```
//以下代码是在ARC下执行的
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
     
    __block id block_obj = [[NSObject alloc]init];
    id obj = [[NSObject alloc]init];

    NSLog(@"block_obj = [%@ , %p] , obj = [%@ , %p]",block_obj , &block_obj , obj , &obj);
    
    void (^myBlock)(void) = ^{
        NSLog(@"***Block中****block_obj = [%@ , %p] , obj = [%@ , %p]",block_obj , &block_obj , obj , &obj);
    };
    
    myBlock();
   
    return 0;
}
```
转换之后的源码：

```
struct __Block_byref_block_obj_0 {
  void *__isa;
__Block_byref_block_obj_0 *__forwarding;
 int __flags;
 int __size;
 void (*__Block_byref_id_object_copy)(void*, void*);
 void (*__Block_byref_id_object_dispose)(void*);
 id block_obj;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  id obj;
  __Block_byref_block_obj_0 *block_obj; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, id _obj, __Block_byref_block_obj_0 *_block_obj, int flags=0) : obj(_obj), block_obj(_block_obj->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_block_obj_0 *block_obj = __cself->block_obj; // bound by ref
  id obj = __cself->obj; // bound by copy

        NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_e64910_mi_1,(block_obj->__forwarding->block_obj) , &(block_obj->__forwarding->block_obj) , obj , &obj);
    }
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->block_obj, (void*)src->block_obj, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_assign((void*)&dst->obj, (void*)src->obj, 3/*BLOCK_FIELD_IS_OBJECT*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->block_obj, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_dispose((void*)src->obj, 3/*BLOCK_FIELD_IS_OBJECT*/);}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};


int main(int argc, const char * argv[]) {

    __attribute__((__blocks__(byref))) __Block_byref_block_obj_0 block_obj = {(void*)0,(__Block_byref_block_obj_0 *)&block_obj, 33554432, sizeof(__Block_byref_block_obj_0), __Block_byref_id_object_copy_131, __Block_byref_id_object_dispose_131, ((NSObject *(*)(id, SEL))(void *)objc_msgSend)((id)((NSObject *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("NSObject"), sel_registerName("alloc")), sel_registerName("init"))};

    id obj = ((NSObject *(*)(id, SEL))(void *)objc_msgSend)((id)((NSObject *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("NSObject"), sel_registerName("alloc")), sel_registerName("init"));
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_45_k1d9q7c52vz50wz1683_hk9r0000gn_T_main_e64910_mi_0,(block_obj.__forwarding->block_obj) , &(block_obj.__forwarding->block_obj) , obj , &obj);

    void (*myBlock)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, obj, (__Block_byref_block_obj_0 *)&block_obj, 570425344));

    ((void (*)(__block_impl *))((__block_impl *)myBlock)->FuncPtr)((__block_impl *)myBlock);

    return 0;
}

```
在转换出来的源码中，我们可以看到，Block捕获了\_\_block，并且强引用了，因为在\_\_Block\_byref\_block\_obj\_0结构体中，有一个变量是id block\_obj，这个默认也是带\_\_strong所有权修饰符的。

根据打印出来的结果来看，ARC环境下，Block捕获外部变量，是都会copy一份的，地址都不同，只不过带有\_\_block修饰符的变量会被捕获到Block内部持有。

而如果将上面的代码在MRC环境下运行，\_\_block根本不会对指针指向的对象执行copy操作，而只是把指针进行了复制。而在ARC环境下，对于声明为__block的外部对象，在block内部会进行retain，以至于在block环境内能安全的引用外部对象，所以才会产生循环引用的问题。




##总结：
5种变量：自动变量、函数参数、静态变量、静态全局变量、全局变量，如果严格的来说，捕获是必须在Block结构体\_\_main\_block\_impl\_0 里面有成员变量的话，Block能捕获的变量就只有自动变量和静态变量了。捕获进Block对象会被Block持有，

> 对于非对象来讲，被copy进Block，不带\_\_block的自动变量只能在里面被访问，并不能改变值，

>带\_\_block的自动变量和静态变量就是直接地址访问，所以在Block里面可以直接改变变量的值。

>而剩下的静态全局变量、全局变量、函数参数，也是可以直接在Block里面改变变量的值得，但是他们并没有变成Block结构体\_\_main\_block\_impl\_0的成员变量，因为他们的作用域大，所以可以直接更改他们的值。

>需要注意的是：静态全局变量、全局变量、函数参数他们并不会被Block持有，也就是说不会增加retainCount的值。
>

> 对于对象来说：

> 	+ MRC 环境下，\_\_block根本不会被指针所指向的对象执行copy操作，而只是把指针进行复制。
> 而在ARC环境下，对于声明为\_\_block的外部对象，在block内部会进行retain，以至于在block环境内能安全的引用外部对象。
> 

在ARC环境下，Block也是存在\_\_NSStackBlock的时候的，平时见到最多的是\_NSConcreteMallocBlock，是因为我们会对Block有赋值操作，所以在ARC下，block类型通过“=”进行传递时，会导致调用objc\_retainBlock->\_Block\_copy->\_Block\_Copy\_internal方法链。并导致\_\_NSStackBlock\_\_类型的Block转换为\_\_NSMallocBlock\_\_类型。
