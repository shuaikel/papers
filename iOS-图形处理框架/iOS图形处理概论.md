![](http://rmt-file.dogedoge.com/rmt_tk/img1/uueqE3u.png)


[文章链接](https://hk.saowen.com/a/18d5e2a371397d777268509e27325f5251a9e7a5ac0eff4844e9ac7bd59e6499)

总的来说，iOS与图形图像处理相关的框架：

+ 1、界面图形框架--UIKit
+ 2、核心动画框架--CoreAnimation
+ 3、苹果封装的图形框架--Core Graphics & Quartz 2D
+ 4、传统跨平台图形框架--OpenGL ES
+ 5、苹果最新力推的图形框架--Metal
+ 6、适合图片的苹果滤镜框架--Core Image
+ 7、适合适配的第三方滤镜方案--GPUImage
+ 8、游戏引擎--Sence Kit（3D）和Sprite Kit（2D）
+ 9、计算机视觉在iOS的应用--OpenCV for iOS


##### 1.界面图形框架 --UIKit

#### UIKit

UIKit是一组Objective-C API，为线条图形、Quartz图像和颜色提供Objective-C封装，并提供2D绘制、图像处理以及用户接口级别的动画。


在UIKit中，UIView类本身在绘制时自动创建一个图形环境，即Core Graphics层的CGContext类型，作为当前的图形绘制环境。在绘制时可以调用UIGraphicsGetCurrentContext函数获取当前的图形环境，例如：

```
- (void)drawRect:(CGRect)rect {
    // Drawing code
    NSLog(@"%s",__func__);
    //1.獲取上下文
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    //2.描述路徑
    UIBezierPath * path = [UIBezierPath bezierPath];
    //起點
    [path moveToPoint:CGPointMake(10, 10)];
    //終點
    [path addLineToPoint:CGPointMake(100, 100)];
    //設置顏色
    [[UIColor whiteColor]setStroke];
    //3.添加路徑
    CGContextAddPath(contextRef, path.CGPath);
    //顯示路徑
    CGContextStrokePath(contextRef);
    
}
```
这段代码就是在UIView的子类中调用UIGraphicsGetCurrentContext函数获取当前的图形环境，然后向该图形环境添加路径，最终绘制。

##### 2.核心动画框架 --Core Animation

#### Core Animation

Core Animation 是UIKit实现动画和变换的基础，也负责视图的复合功能，使用Core Animation可以实现定制动画和细粒度的动画控制，创建复杂的、支持动画和变换的layered 2D视图。

Core Animation 不属于绘制系统，但它是以硬件复合和操作显示内容的基础设施。这个基础设施的核心是layer对象，用来管理和操作显示内容。在iOS中**每一个视图都对应Core Animation的一个层对象**，与视图一样，层之间也组织为层关系树，一个层捕获视图内容为一个被图像硬件容易操作的位图，在多数应用中层作为管理视图的方式使用，但也可以创建独立的层到一个层关系树中来显示视图支持的显示内容。


OpenGL ES的内容也可以与Core Animation内容进行集成。

为了使用Core Animation实现动画，可以修改**层的属性值**来触发一个action对象的执行，不同的action对象实现不同的动画。

##### Core Animation相关基类以及子类

Core Animation 提供了一组应用可以采用的类来提供不同动画类型的支持：

+ CAAnimation  是一个抽象公共基类，CAAnimation采用CAMedia Timing和CAAction协议为动画提供时间（如周期、速度、重复次数等）和action行为（启动、停止等）。

+ CAPropertyAnimation 是CAAnimation的抽象子类，为动画提供一个由一个**key**路径规定的层属性的支持；

+ CABasicAnimation 是CAPropertyAnimation的具体子类，为一个层属性提供简单插入能力；

+ CAKeyframeAnimation 也是CAPropertyAnimation的具体子类，提供key帧动画支持


##### 3.苹果封装的图形框架 --Core Graphics & Quzrtz 2D

#### Core Graphics 

Core Graphics是一套**C-based API**，支持矢量图形、线、形状、图案、路径、位图图像和PDF内容的绘制

#### Quzrtz 2D

Quzrtz 2D是Core Graphics中的2D绘制呈现引擎，Quartz是资源和设备无关的，提供路径绘制，anti-aliased层现，梯度填充图案，图像、透明绘制和透明层、屏蔽和阴影、颜色管理、坐标转换、字体、offsereen呈现、pdf文档创建、显示和分析等功能。

Quartz 2D能够与所有的图形和动画技术（如Core Animation，OpenGL ES，和UIKit等）一起使用。

Quartz 2D采用paint模式进行绘制。

#### 图形环境Context

Quartz 2D中使用的图形环境也是一个类CGContext表示。

在Quartz 2D中可以把一个图形环境作为一个绘制目标，当使用Quartz 2D进行绘制时，所有设备特定的特性被包含在你使用的特定类型的图形环境中，因此通过给相同的图像操作函数提供不同的图像环境你就能够画相同的图像到不同的设备上，因此做到了图像绘制的设备无关性。

对于移动平台，有三种常见的图形环境Context：

+ 1. 位图上下文（A bitmap graphics context）： 一般用于绘制图片或者自定义控件。

```	
	+ 1.1 View Graphics Context： 由UIView自动创建，你重写UIView drawRect方法时，你的内容会画在这个上下文上。

	+ 1.2 Bitmap Graphics Context： 绘制在该上下文的内容会以点阵形式存储在一块内存中，简单来说，就是为图片开一块内存，然后在里面画东西，上下文帮你把图片内存抽象成一个Context（图层）了。
```

+ PDF上下文（A PDF graphics context）：用于生成pdf文档

+ 图层上下文（A layer context）：用于离屏绘制（offscreen drawing）

#### Quzrtz 2D 提供的主要类包括：

+ CGContext ： 表示一个图形环境；

+ CGPath ： 使用矢量图形来创建路径，并能够填充和stroke；

+ CGImage ： 用来表示位图；

+ CGLayer ： 用来表示一个能够用于能够用于重复绘制和offscreen绘制的绘制层；

+ CGPattern：用来表示Pattern，用于重复绘制；

+ CGShading和CGGradient；用于绘制梯度；

+ CGColor 和 CGColorSpace；用于进行颜色和颜色空间管理；

+ CGFont，用于绘制文本；

+ CGPDFContextStream，CGPDFScanner、CGPDFPage、CGPDFObject、CGPDFStream、CGPDFString等用来进行pdf文档的创建、解析和显示。

+ 传统跨平台图形框架--OpenGL ES

#### OpenGL ES

OpenGL ES 是一套多功能开放标准的用于嵌入系统的C-based的图形库，用于2D和3D数据的可视化，OpenGL被设计用来转换一组图形调用功能的底层图形硬件（GPU），由GPU执行图形命令，用来实现复杂的图形操作和运算，从而能够高性能、高帧率利用GPU提供的2D和3D绘制能力。

OpenGL ES规范本身不定义绘制表面和绘制接口，因此iOS为了使用它必须提供和创建一个OpenGL ES的呈现环境，创建和配置存储绘制命令结果的framebuffer以及创建和配置一个或多个呈现目标。

#### EAGL

在iOS中使用EAGL提供的EAGLContext类来实现和提供一个呈现环境，用来保持OpenGL ES使用到的硬件状态，EAGL是一个Objective-C API，提供使OpenGL ES与Core Animation 和UIKit集成的接口。

在调用任何OpenGL ES功能之前必须首先初始化一个EAGLContext对象，每一个iOS应用的每一个线程都有一个当前的Context，在调用OpenGL ES函数时，使用或改变此context中的状态。

EAGLContext 的类方法setCurrentContext：用来设置当前线程的当前context。EAGLContext的类方法currentContext返回当前线程的当前context，在切换相同线程的两个上下文之前，必须调用glFlush函数来确保先前已提交的命令被提交到图形硬件中。

#### GLKit

可以采用不同的方式使用OpenGL ES以便呈现OpenGL ES内容到不同目标：GLKit和CAEAGLLayer。

为了创建全屏幕的视图或使OpenGL ES内容与UIKit集成，可以使用GLKit。在使用GLKit时，GLKit提供的类GLKView类本身实现呈现目标及创建和维护一个framebuffer。

GLKit是一组Objective-C类，为使用OpenGL ES提供一个面向对象接口，用来简化OpenGL ES应用的开发。

#### CAEAGLayer

为了使OpenGL ES内容作为一个Core Animation层的部分内容时，可以使用CAEAGLLayer作为呈现目标，并需要另外创建frameBuffer以及自己实现和控制整个绘制流程。

##### GLKit支持四个3D应用开发的关键领域

+ GLKView和GLKViewController类提供一个标准的OpenGL ES视图和相关连的呈现循环，GLKView可以作为OpenGL ES内容的呈现目标，GLKViewController提供内容呈现的控制和动画，视图管理和维护一个framebuffer，应用只需在framebuffer进行绘画即可。


+ GLKTextureLoader 为应用提供从iOS支持的各种图像格式的源自动加载文理图像到OpenGL ES图像环境的方式，并能够进行适当的转换，并支持同步和异步加载方式。

+ 数学运算库，提供矢量、矩阵、四元数的实现和矩阵堆栈操作等OpenGL ES1.1功能。


+ 

























