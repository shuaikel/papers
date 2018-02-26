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
