
+ 真机调试包路径以及配置文件路径

#### 真机调试包路径

+ 点击桌面 按command+shift+G
+ /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport


### 配置文件路径

+ ~/Library/MobileDevice/Provisioning Profiles


### Cocoa框架Foundation和UIKit的区别

#### cocoa

Cocoa不是一种编程语言（它可以运行多种编程语言），它也不是一个开发工具（通过命令行我们仍然可以开发Cocoa程序），它是创建Mac OX X和iOS程序的原生面向对象API,为这两者应用提供了编程环境。我们通常称为”Cocoa框架“，事实上Cocoa本事是一个框架的集合，它包含了众多子框架，其中最重要的要数”Foundation“和”UIKit“,前者是框架的基础，和界面无关，其中包含大量常用的API;后者是基础的UI类库

![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%A4%A7%E6%9D%82%E7%83%A9/Cocoa%E6%A1%86%E6%9E%B6Foundation%E5%92%8CUIKit%E7%9A%84%E5%8C%BA%E5%88%AB/Cocoa.jpeg)


#### Foundation框架的特点：

+ 都是以NS为前缀
+ 类都是继承自超类Object

![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%A4%A7%E6%9D%82%E7%83%A9/Cocoa%E6%A1%86%E6%9E%B6Foundation%E5%92%8CUIKit%E7%9A%84%E5%8C%BA%E5%88%AB/Foundation1.jpeg)


![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%A4%A7%E6%9D%82%E7%83%A9/Cocoa%E6%A1%86%E6%9E%B6Foundation%E5%92%8CUIKit%E7%9A%84%E5%8C%BA%E5%88%AB/Foundation2.jpeg)

![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%A4%A7%E6%9D%82%E7%83%A9/Cocoa%E6%A1%86%E6%9E%B6Foundation%E5%92%8CUIKit%E7%9A%84%E5%8C%BA%E5%88%AB/Foundation3.jpeg)

#### UIKit框架图

![](https://raw.githubusercontent.com/SunshineBrother/JHBlog/master/iOS%E7%9F%A5%E8%AF%86%E7%82%B9/iOS%E5%A4%A7%E6%9D%82%E7%83%A9/Cocoa%E6%A1%86%E6%9E%B6Foundation%E5%92%8CUIKit%E7%9A%84%E5%8C%BA%E5%88%AB/UIKit.png)


#### UIView和CALayer区别?

UIView和CALayer都是UI操作的对象，两者都是NSObject的子类，发生在UIView上的操作本质上发生在对应的CALayer上，UIView是CALayer用于交互的抽象，UIView是UIResponder的子类（UIResponder是NSObject的子类），提供了很多CALayer所没有的交互上的接口，主要负责处理用户触发的种种操作。CALayer在图像和动画渲染上性能更好，这是因为UIView有冗余的接口，而且相比 CALayer 还有层级之分。CALayer 在无需处理交互时进行渲染可以节省大量时间。 CALayer的动画要通过逻辑树、动画树和显示树来实现


####  loadView是干嘛用的？

> loadView用来自定义view，只要实现了这个方法，其他通过xib或storyboard创建的view都不会被加载 。
> 
> 

####  layoutIfNeeded、layoutSubviews和setNeedsLayout的区别？

+ layoutIfNeeded：方法调用后，在主线程对当前视图及其所有子视图立即强制更新布局。
+  layoutSubviews：方法只能重写，我们不能主动调用，在屏幕旋转、滑动或触摸界面、子视图修改时被系统自动调用，用来调整自定义视图的布局。 
+  setNeedsLayout：方法与layoutIfNeeded相似，不同的是方法被调用后不会立即强制更新布局，而是在下一个布局周期进行更新。

