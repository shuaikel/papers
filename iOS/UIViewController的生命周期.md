ViewController 总体生命周期：
![](https://user-gold-cdn.xitu.io/2018/1/30/161472afda091e30?imageslim)

###ViewController的多种实例化方法：
+ 通过Xib加载
当控制器View通过Xib加载的时候，可能会出现三种情况：
	+ 指定Xib名称
	+ 不指定xib名称，loadView就会加载与控制器同名的xib
	+ 不指定xib名称2，同时也没有与控制器同名的xib，会加载前缀与控制器相同而不带Controller的xib
+ init
> 我们经常会用代码通过init手动创建一个ViewController，其实本质还是调用了initWithNibName：bundle：并且都传入了nil，只不过以上三种情况都没有满足



+ storyboard 间接实例化（initWithCoder）
> 当你从storyboard初始化ViewController时，iOS会使用initWithCoder，而不是initWithNibName来初始化这个ViewController，然后那个storyboard会在自己内部生成一个nib（storyboard实例化view/ViewController时），会把nib的信息放在Coder中，调用initWithCoder)
> 
> 注意： storyboard加载的是控制器及控制器的View,而xib加载的仅仅是控制器的View，之所以这么说，我们结合控制器的awakeFromNib方法解释一下，顾名思义，当_控制器从nib加载的时候就会调用这个方法_，这个方法本身只是信号、消息，是一个空方法
> 
> 
> 
> 


### loadView
> 在这个方法中，要正式加载View了，首先我们知道，控制器的View是通过懒加载的方式进行加载的，所以一般我们不需要主动调用这个方法，当我们用到控制器的View时，就会调用控制器View的get方法，在get方法内部，首先判断View是否已经创建，如果已存在，这直接返回存在的View，如果不存在，则调用控制器的loadView方法，在控制器没有被销毁的情况下，loadView也可能会被多次执行
> 
> 当ViewController有以下情况时都会从nib文件中加载View:
> 
>	+ ViewController 是从storyboard中实例化的。
>	+ 通过initWithNibName：bundle：实例化。
>	+ 在App Bundle中有一个nib文件名称和本类名相同。
>
> 符合以上三点时，也就不需要重写这个方法，否则你无法得到你想要的nib中的View。
> 
> _是否需要调用[super loadView]_
> 	
> 	loadView 方法的默认实现是这样的：先寻找有关可用的nib文件的信息，根据这个信息来加载nib文件，如果没有有关的nib文件的信息，默认实现会创建一个空白的UIView对象，然后让这个对象成为controller的主View。
> 所以，重写这个函数时，你也应该这么做，并把子类的view赋给view属性
> 
>



###ViewController加载View的过程

![](https://user-gold-cdn.xitu.io/2018/1/30/16147297ea00b115?imageslim)
从图中可以看到，在View加载过程中首先会调用loadView方法,在这个方法中主要完成一些关键view的初始化工作，比如UINavigationViewController和UITabBarController等容器类的ViewController；接下来就是加载View，加载成功之后，会接着调用viewDidLoad方法，这里要记住一点的是，在loadView之前，是没有view的，也就是说，在这之前，view还没有被初始化，完成viewDidLoad方法后，viewController里面就成功的加载view了。

----
```
这里如果loadView没有加载view，即为nil，viewDidLoad会一直调用loadView加载view，因此构成了死循环，程序即卡死，所以我们常在ViewDidLoad里面创建View。
```

###ViewController卸载View的过程
![](https://user-gold-cdn.xitu.io/2018/1/30/161472cad3619f50?imageslim)

从图中可以看到，当系统发出内存警告时，会调用didReceiveMemoeryWarning方法，如果当前有能被释放的View，完成后调用viewDidUnload方法，至此，view就被卸载了，此时原本指向view的变量被置为nil，具体操作是在viewDidLoad方法中调用self.myButton = nil;


#### 3. viewDidLoad
> 当控制器的loadView方法执行完毕，view被创建成功之后，就会执行viewDidLoad方法，该方法与loadView方法一样，也有可能被执行多次，在开发中，我们可能从未遇到过执行多次的情况，那什么时候会执行多次呢？
> 
> 比如A控制器push出B控制器，此时，窗口显示的是B控制器的view，此时如果收到内存警告，我们一般会将A控制器中没用的变量及View销毁掉，之后当我们从B控制器pop到A控制器时，就会再次执行A控制器的loadView方法与viewDidLoad方法。
> 

#### 4. ViewwillAppear && ViewDidAppear
> + viewWillAppear
	+ viewWillAppear总是在ViewDidLoad之后被调用，但不是立即，当你只是引用了属性view，确没有立即把view添加到任何已经展示的视图上时，viewWillAppear不会被调用，这在view被外部引用时，就会发生，当然，随着ViewController的多次推入，多次进入子页面后返回，该方法会被多次调用，与viewDidLoad不同，调用该方法说明控制器一定会显示
	
----

> + _锁屏之后会被调用吗？_
> 
	> 不会，viewWillAppear关注的是view在层次中显示与消失，锁屏并没有改变App本身的层次。
> + _Window叠加后，会被调用吗？_
> 
	> 不会，同锁屏的原因类似，叠加Window并没有改变ViewController所在Window的视图层次，换句话说，view并没有被覆盖或删除（相对于自己所在的Window）。

> + ViewDidAppear
	> 视图已经在屏幕上渲染完成，子视图有自定义动画时，建议在Did方法中启动，在Will中启动动画时，动画效果将不会很理想。
	
#### 在viewWillAppear 与 viewDidAppear之间发生了什么
	以下两个方法将会被调用：
	- viewWillLayoutSubviews 	
	- viewDidLayoutSubviews

> +  viewWillLayoutSubviews
> 
	>  该方法在通知控制器将要布局view的子控件时调用，每当视图的bounds改变，view将调整其子控件位置，默认实现为空，可重写以在view布局子控件前做出改变，该方法调用时，AutoLayout未起作用。
	
> +  viewDidLayoutSubviews
	>	该方法在通知控制器已经布局view的子控件时调用，默认实现为空，可重写以在view布局子控件后作出改变，该方法调用时，AutoLayout未起作用。

--
######注意：
> 使用AutoLayout时，子视图大小只有在ViewDidLayoutSubviews才真正被设置好，所以这里才是获取子视图大小的正确位置，常见的错误是，在viewDidLoad中读取了某个view.frame，用来给其它子视图赋值，结果得到一堆大小“不定”的视图，甚至可能为零，在视图中看不见。


###5. viewWillDisAppear && viewDidDisAppear
>  + viewWillDisAppear

>  > 该方法在控制器view将要从视图层次被移除时被调用，可重写已提交变更，取消视图第一响应者状态


> + viewDidDisappear
> 	> 该方法在控制器的view已经从视图层次移除时被调用，可以重写清除或隐藏控件

---
>这两个配套调用，具体指子视图控制器是已push和present方法显示的，父视图控制器的以上两个方法会被触发。

> 特别的，addSubview会调用子控制器Appear系列方法，但不会调用父视图viewWillDisAppear方法。

---

###6. didReceiveMemoryWarning && viewDidUnload（ios6之后废除）
>
> 当系统内存不足时，首先ViewController的didReceiveMemoryWarning方法会被调用，而 didReceiveMemoryWarining 会判断当前 ViewController 的 view 是否显示在 window 上，如果没有显示在 window 上，则 didReceiveMemoryWarining 会自动将 ViewController 的 view 以及其所有子 view 全部销毁，然后调用 viewcontroller 的 viewdidunload 方法。如果当前 ViewController 的 view 显示在 window 上，则不销毁该 ViewController 的 view，当然，viewDidunload 也不会被调用了。

> iOS 升级到 6.0 以后，不再支持 viewDidUnload 了。官方文档的解释是系统会自动控制大的 view 所占用的内存，其他小的 view 所占用的内存是极其微小的，不值得为了省内存而去清理然后在重新创建。如果你需要在内存警告的时候释放业务数据或者做些其他的特定处理，你可以实现  didReceiveMemoryWarning 这个函数。

###7. dealloc
> 当发出内存警告时调用viewDidUnload方法时，只是释放了view，并没有释放viewController，所以并不会调用dealloc，即viewDidUnload和dealloc方法并没有任何关系，dealloc方法只会在viewcontroller被释放时被调用
> 

###8. 其它相关方法
> awakeFromNib
>	> 当nib文件被加载的时候，会发送一个awakeFromNib的消息到.nib文件中的每个对象，每个对象都可以定义自己的awakeFromNib方法来响应这个消息，执行一些必要的操作，也就是说通过nib文件创建view对象时执行awakeFromNib。
>


###9. 多个ViewController跳转时的生命周期
当我们点击push的时候首先会加载下一个界面然后才会调用界面的消失方法。

+ init: ViewController2

+ loadView: ViewController2

+ viewDidLoad: ViewController2

+ viewWillDisappear : ViewController1将要消失

+ viewWillAppear: ViewController2将要出现
 
+ viewWillLayoutSubviews ：ViewController2

+ viewDidLayoutSubviews： ViewController2

+ viewWillLayoutSubviews ： ViewController1

+ viewDidLayoutSubviews ： ViewController1

+ viewDidDisappear:ViewController1 完全消失

+ viewDidAppear:ViewController2 完全出现
当一个控制器push/present新的控制器，原先的控制器并不会销毁，但会消失，因此调用了ViewWillDisappear 和 viewDidDisappear方法。


###10 Pop/Dismiss




 
