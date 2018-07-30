###### block定义：block本质上也是一个OC对象，他内部也有一个isa指针。block是封装了函数调用以及函数调用环境的OC对象。

```
void (*gloablBlock)(int a) = ((void (*)(int))&__OBJCClassTwo__init_block_impl_0((void *)__OBJCClassTwo__init_block_func_0, &__OBJCClassTwo__init_block_desc_0_DATA));
 
 
 上述定义代码中，可以发现，block定义中调用了__OBJCClassTwo__init_block_impl_0 函数，并且将__OBJCClassTwo__init_block_impl_0函数的地址赋值给了block，
```

```
<!--__OBJCClassTwo__init_block_impl_0 结构体-->

struct __OBJCClassTwo__init_block_impl_0 {
  struct __block_impl impl;
  struct __OBJCClassTwo__init_block_desc_0* Desc;
  __OBJCClassTwo__init_block_impl_0(void *fp, struct __OBJCClassTwo__init_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

// __OBJCClassTwo__init_block_impl_0结构体内有一个同名的构造函数
__main_block_impl_0，构造函数中对一些变量进行赋值并最终返回了一个新
的结构体。那么也就是说最终最终将一个
__OBJCClassTwo__init_block_impl_0 结构体的地址赋值给了block变量



<!--Block函数实现-->
static void __OBJCClassTwo__init_block_func_0(struct __OBJCClassTwo__init_block_impl_0 *__cself, int a) {

            NSLog((NSString *)&__NSConstantStringImpl__var_folders_bx_c20820xj0q11zkzx90g1fxx40000gn_T_OBJCClassTwo_62e89c_mi_1,((NSNumber *(*)(Class, SEL, int))(void *)objc_msgSend)(objc_getClass("NSNumber"), sel_registerName("numberWithInt:"), (int)(a)),test);
        }
        
 // 我们写在block中的代码封装成了__OBJCClassTwo__init_block_func_0函数，并将__OBJCClassTwo__init_block_func_0函数的地址传入了__OBJCClassTwo__init_block_impl_0的结构体函数然后保存在结构体内。
        
<!--Block结构体内存大小-->
static struct __OBJCClassTwo__init_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __OBJCClassTwo__init_block_desc_0_DATA = { 0, sizeof(struct __OBJCClassTwo__init_block_impl_0)};

// 可以看到__OBJCClassTwo__init_block_desc_0中存储着两个参数，reserved和Block_size，并且reserved赋值为0而Block_size则存储着__OBJCClassTwo__init_block_impl_0结构体占用的空间，最终将
__OBJCClassTwo__init_block_desc_0结构体的地址传入__OBJCClassTwo__init_block_impl_0中赋值给Desc。

```

```
__block_impl结构体内部就有一个isa指针，因此可以证明block本质上就是一个oc对象，而在构造函数中将函数中传入的值分别存储在__OBJCClassTwo__init_block_impl_0结构体实例中，最终将结构体的地址赋值给block
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};
```

> 上面对__OBJCClassTwo__init_block_impl_0结构体构造函数三个参数的分析我们可以得出结论：

```
1.__block_impl 结构体中isa指针存储着&NSConcreteStackBlock地址，可以暂时理解为其类对象地址，
2. block代码块中的代码被封装成__OBJCClassTwo__init_block_func_0函数，FuncPtr则存储着__OBJCClassTwo__init_block_func_0函数的地址
3. Desc指向__OBJCClassTwo__init_block_desc_0结构体对象，其中存储__OBJCClassTwo__init_block_impl_0结构体所占用的内存内存。
```

#### 调用block执行内部代码

```
((void (*)(__block_impl *, int, int))((__block_impl 
*)OBJCBlock)->FuncPtr)((__block_impl *)OBJCBlock, 10, 20);
```

通过上面的代码可以发现调用block是通过block找到FunPtr直接调用，通过上面的分析我们可以知道block是指向的是\_\_OBJCClassTwo\_\_init\_block\_impl\_0类型结构体，但是我们发现\_\_OBJCClassTwo\_\_init\_block\_impl\_0结构体并不直接就可以找到FunPtr，而FunPtr是存储在\_\_block\_impl中的，为什么block可以直接调用\_\_block\_impl中的FunPtr呢？

通过源码可以看到，（\_\_block\_impl *）block将block强制转化为\_\_block\_impl类型，因为\_\_block\_impl是\_\_OBJCClassTwo\_\_init\_block\_impl\_0结构体的第一个成员，相当于\_\_block\_impl 结构体的成员直接拿出来放在\_\_OBJCClassTwo\_\_init\_block\_impl\_0中，那么也就说明__block_impl的内存地址就是\_\_OBJCClassTwo__init_block_impl_0结构体的内存地址开头，所以可以转化成功，并找到FunPtr成员。


FunPtr中存储着block内代码块封装的函数地址__OBJCClassTwo__init_block_func_0，那么调用此函数，也就是会执行代码中的代码，并且回头查看__OBJCClassTwo__init_block_impl_0函数，可以发现第一个参数就是__OBJCClassTwo__init_block_impl_0类型的指针，也就是说将block传入__OBJCClassTwo__init_block_func_0函数中，便于从中取出block捕获的值。
![](https://user-gold-cdn.xitu.io/2018/5/20/1637de343b05ffaa?imageView2/0/w/1280/h/960/ignore-error/1)


###### block中的变量捕获

为了保证block内部能正常访问外部的变量，block有一个变量捕获机制

+ 局部变量 auto变量
> 值传递，不能在block内修改外部变量的值。可通过__block 实现，
 
+ static 变量
> static 修饰的变量为指针传递，同样会被block捕获。


+ 全局变量

> block中可以访问全局变量的原因是全局变量作用域的效果。
> 

###### 总结： 局部变量都会被block捕获，自动变量是值捕获，静态变量是地址捕获，全局变量则不会被block捕获。而且，即使是block中使用的是实例对象的属性，block中捕获的仍然是实例对象，并通过实例对象捕获的方式去获取使用到的属性。


+ block的类型：block有三种类型：

+ \_\_NSGlobalBlock\_\_
+ \_\_NSStackBlock\_\_
+ \_\_NSMallocBlock\_\_

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // 1. 内部没有调用外部变量的block
        void (^block1)(void) = ^{
            NSLog(@"Hello");
        };
        // 2. 内部调用外部变量的block
        int a = 10;
        void (^block2)(void) = ^{
            NSLog(@"Hello - %d",a);
        };
       // 3. 直接调用的block的class
        NSLog(@"%@ %@ %@", [block1 class], [block2 class], [^{
            NSLog(@"%d",a);
        } class]);
    }
    return 0;
}

``` 

##### 上面函数执行的结果是： 
\_\_NSGlobalBlock\_\_ ,\_\_NSMallocBlock\_\_， \_\_NSStackBlock\_\_


###### block在内存中的存储
![](https://user-gold-cdn.xitu.io/2018/5/20/1637de34c0579805?imageView2/0/w/1280/h/960/ignore-error/1)


###### 什么情况下ARC会自动将block进行一次copy操作？

+ block作为函数返回值时

+ 将block赋值给\_\_strong指针时

+ block被强指针引用时，RAC也会自动对block进行一次copy操作；

+  block作为Cocoa API中方法名含有usingBlock的方法参数时

+ block作为GCD API的方法参数时
+ 
 
