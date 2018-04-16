+  [原文地址:](https://juejin.im/post/5ad3f1cc6fb9a028d9379c5f?utm_source=gold_browser_extension)
###### ios渲染过程
> 在开始iOS Rendering Process之前，我们需要对iOS的渲染概念有一个基本的认知：
> 
> 基于平铺的渲染
> 
> iOS设备的屏幕分为N*N像素的图块，每个图块都适合于SoC缓存，几何体在图块内被大量拆分，只有在所有几何体全部提交之后才可以进行光栅化。

 	+ 这里的光栅化指将屏幕上面被大量拆分出来的几何体渲染为像素点的过程。

> iOS Rendering 技术框架
> iOS 渲染相关的层级划分大概如下：

![](https://user-gold-cdn.xitu.io/2018/4/16/162cbea46b54c89e?imageslim)

+ UIKit

> 作为一名iOS Developer来说，应该对UIKit并不陌生，我们日常开发中使用的用户交互组件都来自于UIKit FrameWork，我们通过设置UIKit组件的Layout 以及BackgroundColor等属性来完成界面的绘画工作。
> 其实UIKit Framework本身并不具备在屏幕成像的能力，它主要负责对用户操作事件的响应，事件响应的传递大体是经过树层的视图树遍历实现的。
> 

+ Core Animation

> Core Animation其实是一个令人误解的命名。实际上它是从一个叫Layer Kit这么一个不怎么和动画有关的名字演变而来的，所以做动画仅仅是Core Animation特性的冰山一角。
> 
> Core Animation 本质上可以理解为一个复合引擎，旨在尽可能的组合屏幕上不同的显示内容，这些显示内容被分解成为独立的图层，即CALayer，CALayer才是你在屏幕上看见的一切的基础。
> 
> 视图的职责就是创建并管理这个图层，以确保当子视图在层级关系中被添加或者移除时，他们关联的图层也同样对应在层级关系树中有相同的操作。
> 

	+ 但是问什么iOS 要基于UIView 和CALayer提供两个平行的层级关系
	+ 呢？为什么不用一个简单的层级关系来处理所有的事情呢？
> 原因在与职责分离，这样也能避免很多重复代码，在iOS和Mac OS X 两个平台上，事件和用户交互有很多地方的不同，基于多点触控的用户界面和基于鼠标键盘的交互有着本质的区别，这就是为什么iOS有UIKit和UIView，而Mac OS X 有AppKit 和NSView的原因，他们功能上相似，但是在实现上有显著的区别。
> 
> 
> 

+ OpenGL ES & Core Graphics

+ [OpenGL ES](https://en.wikipedia.org/wiki/OpenGL_ES) ,即OpenGL for Embedded Systems，是OpenGL的子集，通常面向图形加速处理单元（GPU）渲染2D和3D计算机图形，例如视频游戏使用的计算机图形。

+ OpenGL ES 专为智能手机，平板电脑，视频游戏和PDA等嵌入式系统设计，OpenGL ES是“应用最广泛的3D图形API”。


+ [Core Graphics](https://developer.apple.com/documentation/coregraphics?language=objc) 基于Quzrtz高级绘图引擎，它提供了具有无与伦比的输出保真度的低级别轻量级2D渲染，你可以使用此框架处理基于路径的绘图，转换，颜色管理，离屏渲染，图案，渐变和阴影，图像创建和图像遮罩以及PDF文档创建，显示和分析。

+ Graphics HardWare 译为图形硬件，iOS设备中也有自己的图形硬件设备，也就是我们经常提及的GPU。

+ 图形处理单元（GPU）是一种专用电子电路，旨在快速操作和改变存储器，以加速在用于输出到显示设备的帧缓冲中创建图像，GPU被用于嵌入式系统，现代GPU在处理计算机图形和图像方面非常高效，并且GPU的高度并行结构使其在大数据并行处理的算法中比通用CPU更有效。


+ OpenGL 主要渲染步骤

> OpenGL 全称Open Graphics Library, 译为开放图形库，是用于渲染2D和3D矢量图形的跨语言，跨平台的应用程序编程接口（API），OpenGL可以直接访问GPU，以实现硬件加速渲染。

> 一个用来渲染图像的OpenGL 程序主要可以大致分为以下几个步骤：
 
> +  设置图元数据
> + 着色器-shader 计算图元数据（位置、颜色、其他）
> + 光栅化-resterization 渲染为像素
> + fragment shader 决定最终成像
> + 其他操作（显示、隐藏、融合）

+ OpenGL Render Pipeline
+ 渲染管道（渲染管线）
+ 其实REnder Pipeline指的是从应用程序数据转换到最终渲染的图像之间的一系列数据处理过程。
+ 好比我们上下文中提到的OpenGL 主要渲染步骤一样，我们开发应用程序时在设置图元数据这步为视图控件的设定布局，背景颜色，透明度以及阴影等等数据。

![](https://user-gold-cdn.xitu.io/2018/4/16/162cbea972a8889b?imageslim)

+ OpenGL 在最终成像之前还会经历一个阶段为计算着色阶段，这个阶段OpenGL 会计算最重要早屏幕中成像的像素位置以及颜色，如果在之前提交代码时用到CALayer 会引起blending的显示效果或者视图颜色或内容图片的alpha通道开启，都将会加大这个阶段的OpenGL的工作量。


+ Core Animation Pipeline

> 上文说道了iOS设备之所以可以成像不是因为UIKit而是因为LayerKit,即Core Animation。
> 
> Core Animation图层，即CALayer中包含一个属性contents，我们可以通过这个属性赋值来控制CALayer成像的内容，这个属性的类型定义为id，在程序编程时不论我们给contents赋予任何类型的值，都是可以编译通过的，但是在实践中，如果contents赋值类型不是CGImage,那么将会得到一个空白图层。


```
Note：造成contents属性的奇怪表现的原因是Mac OS X的历史包袱，
它之所以被定义为id类型的原因是Mac OS X 中对这个属性对CGImage 
和 NSImage 类型的值都起作用，但是在iOS中，
如果你赋予一个UIImage属性的值，仅仅会得到一个空白图层。
```

> 下面介绍下iOS中Core Animation Pipeline:
> 
>  + 在Application中布局UIKit视图控件间接的关联Core Animation 图层
> 
> +  Core Animation 图层相关的数据提交到iOS Render Server，即OpenGL ES & Core Graphics
> 
> + Render Server 将与GPU通信把数据经过处理之后传递给GPU。
> 
> + GPU调用iOS当前设备渲染相关的图形设备DisPlay。

![](https://user-gold-cdn.xitu.io/2018/4/16/162cbeac560efb4f?imageslim)

	+ Note: 由于iOS设备目前的显示屏最大支持60FPS的刷新率，
	+ 所以每个处理间隔为16.67ms

> 可以看到从Commit Transaction 之后我们的图元数据就将会在下一次RunLoop时被Application发送给底层Render Server，底层Render Server直接面向GPU经过一系列的数据处理将处理完毕的数据传递给GPU，然后GPU负责渲染工作，根据当前iOS设备的屏幕计算图像像素位置以及像素alpha通道混色计算等等最终在当前iOS设备中呈现图像。
> 


+ Commit Transaction
+ Core Anuimation Pipeline的整个管线中iOS常规开发一般可以影响的范围也就仅仅是在Application 中布局UIKit视图控件间接的关联Core Animation图层这一级，即Commit Transaction之前的一些操作。

+ 那么在Commit Tansaction 之前我们一般要做的事情有哪些？

> + Layout ,构建图层
> + Display,绘制视图
> + Prepare,额外的Core Animation工作
> + Commit，打包图层并将它们发送到Render Server

+ Layout

> 在Layout阶段我们需要把constraint写的尽量高效，iOS的Layout Constraint 类似于Android的 Relative Layout
> 这个阶段的Layout计算工作是在CPU完成的，包括layoutSubviews方法的重载，addSubview: 方法填充子视图。

+ Display

> 其实这里的Display仅仅是我们设置iOS设备要最终成像的图元数据而已，重载视图drawRect：方法可以自定义UIView的显示，其原理是在drawRect：方法内部绘制bitmap。

> + 重载drawRect：使用不当会造成CPU负荷过重，App内存飙升等问题。
> 

+ Prepare: 

> 这个步骤属于附加步骤，一般处理图像的解码 & 转换等操作。
> 

+ Commit

> commit 步骤指打包图层并将它们发送到 Render Server
> 
> commit 操作会递归执行，由于图层和视图一样是以树形结构存在的，当图层树过于复杂时Commit操作的开销也会非常大。
> 
> 

+ CATransaction

> CATransaction 是Core Animation 中用于将多个图层树操作分配到渲染树的原子更新中的机制，对图层树的每个修改都必须是事务中的一部分。
> 
> CATransaction 类没有属性或者实例方法，并且也不能用+alloc 和 -init方法创建它，我们只能用类方法 +begin 和 +commit分别来入栈或者出栈。
> 
> 事实上任何可动画的图层属性都会被添加到栈顶的事务，你可以通过+setAnimationDuration: 方法设置当前事务的动画时间，或者通过+animationDuration方法来获取时长。  
> 


+ iOS中卡顿产生的原因：
+ 在VSync信号到来后，系统图形服务器会通过CADisplayLink等机制通知App，App主线程开始在CPU中计算显示内容，比如视图的创建、图片渲染、布局计算、文本绘制等，随后CPU会将计算好的内容提交到GPU，有GPU进行变换、合成、渲染。随着GPU会把渲染结果提交到帧缓冲区去，CPU或者GPU没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏还是保留之前的内容不变，这就是界面卡顿的原因。


> iOS设备中的CPU & GPU
> 
> CPU ： 加载资源、对象创建、对象调整、对象销毁、布局计算、AutoLayout、文本计算、文本渲染、图片的解码、图像的绘制都是在CPU上面进行的。
> 
> 


###### ios常见的保持界面流畅的优化方案
+ 1、 TableviewCell 复用，在cellForRowAtIndexPath: 回调的时候只创建实例，快速返回cell，不绑定数据，在willDisPlayCell：forRowAtIndexPath：的时候绑定数据（赋值）。
+  2、 高度缓存，在tableView滑动时，会不断调用heightForAtIndexPath:，当cell高度需要自适应时，每次回调都要计算高度，会导致UI卡顿，为了避免重复无意义的计算，需要缓存高度。（通过字典、NSCache）
+  3、视图层级的优化，	

>  不要动态创建视图
> 
+  在内存可控的前提下，缓存subview
+  善用hidden。

> 减少视图层级
> 
+ 减少subview个数，使用layer绘制元素。
+ 少用clearColor，maskToBounds、阴影效果等。

> 减少多余的绘制操作
> 
> > + 图片

> > > + 不要使用JPEG的图片，应当使用PNG图片。
> > > + 子线程预解码（Decode），主线程直接渲染。因为当image没有Decode，直接赋值给imageView会进行一个Decode操作。
> > > + 优化图片大小，尽量不要动态缩放（contentMode）。
> > > + 尽可能将多张图片合成为一张进行显示。
> 
> 减少透明View
> 使用透明view会引起blending，在iOS的图形处理中，blending主要指的是混合像素颜色的计算，最直观的例子就是，我们把两个图层叠加在一起，如果第一个图层是透明的，则最终像素的颜色计算需要将两个图层也考虑进来，这一过程即为blending。

> 会导致blending的原因：
> > + UIView的alpha < 1
> > + UIImageView的image含有alpha channel（即使 UIImageView的alpha是1，但只要image含有透明通道，则仍会导致blending）
> > opaque设置为YES，减少性能损耗，因为GPU将不会做任何合成，而是简单从这个层拷贝。

> 减少离屏渲染
> 离屏渲染指的是在图像在绘制到当前屏幕前，需要先进性一次渲染，之后才绘制到当前屏幕。
> OpenGL 中，GPU屏幕渲染有以下两种方式：
> > + on-screen Rendering 即当前屏幕渲染，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行的。
> > + Off-screen Rendering 即离屏渲染，指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作

> 为什么离屏渲染会发生卡顿？主要包括两方面的内容：
> > + 创建新的缓冲区
> > + 上下文切换，离屏渲染的整个过程，需要多次切换上下文环境（CPU和GPU切换），先是从当前屏幕（ON-Screen）切换到离屏（Off-Screen）;等到离屏渲染结束后，将离屏缓冲区的渲染结果显示到屏幕上又需要将上下文从离屏切换到当前屏幕，而上下文切花是很消耗性能的。
> > 
> > + 设置一下属性时，都会触发离屏渲染：
> >  
> > > > + layer.shouldRasterize，光栅化
> > > > + layer.mask 遮罩
> > > > + layer.allowsGroupOpacity为YES，layer.opacity的值小于1.0
> > > > + layer.connerRadius，并且设置layer.masksToBounds为YES，则可以使用剪切过的图片，或者使用layer画来解决。
> > > > + layer.shadows（表示相关的shadow开头的属性），使用shadowPath代替。
> > > > 
> > > > > + 不使用shadowPath

![](https://user-gold-cdn.xitu.io/2018/3/21/1624430835219f66?imageslim)

> > > > > + 使用shadowPath

![](https://user-gold-cdn.xitu.io/2018/3/21/1624435303eade8b?imageslim)

性能差别，如下图：
![](https://user-gold-cdn.xitu.io/2018/2/23/161c09f5e3469ce4?imageslim)

离屏渲染的优化建议：
> + 使用ShadowPath指定layer阴影效果路径。
> + 使用异步layer渲染（Facebook 开源的异步绘制框架AsyncDisPlayKit）.
> 设置layer的opaque值为YES，减少复杂图层合成。
> 尽量使用不包含透明（alpha）通道的图片资源。
> 尽量设置layer的大小值为整形值。
> 直接让美工把图片切成圆角进行显示，
> 很多情况下用户上传图片进行显示，可以在客户端处理圆角。
> 使用代码手动生成圆角image设置到要显示的View上，利用UIBezierPath(core graphics框架)画出圆角图片。


+ 合理使用光栅化 shouldRasterize
+ 光栅化是把GPU的操作转到CPU上，生成位图缓存，直接读取复用。

优点：

+ CALayer 会被光栅为bitmap、shadows、connerRadius等效果会被缓存

缺点：

+  更新已经光栅化的layer，会造成离屏渲染。
+  bitmap超过100ms没有使用就会移除
+  受系统限制，缓存的大小为2.5 * screen size。

shouldRasterize 适合静态页面显示，动态页面也会增加开销，如果设置了shouldRasterize为YES，那么也要记住设置 rasterizationScale为contentsScale。

+ 异步渲染，在子线程中绘制，在主线程中渲染：

![](https://user-gold-cdn.xitu.io/2018/2/11/1618448a7403100f?imageslim)

+ 理性使用-drawRect：
+ 当你使用UIImageView 在加载一个视图的时候，这个视图虽然依然有CALayer，但是却没有申请到一个后备的存储，取而代之的是使用屏幕外渲染，将CGImageRef作为内容，并用渲染服务将图片数据绘制到帧的缓冲区，就是显示到屏幕上，当我们滚动视图的时候，这个视图将会重新加载，浪费性能，所以对于使用-drawRect:方法，更倾向于使用CALayer来绘制图层。因为使用CALayer的-drawInContext:，Core Animation将会为这个图层申请一个后备存储，用来保存那些方法绘制进来的位图。那些方法内的代码将会运行在 CPU上，结果将会被上传到GPU。这样做的性能更为好些。

+ 按需加载

> + 局部刷新，刷新一个Cell就能解决的，坚决不刷新整个section或者整个tableView,刷新最小单元元素。
> + 利用runloop 提高滑动流畅性，在滑动停止的时候再加载内容，就没有必要加载，可以默认占位图。
> 
> 

[网页地址](https://juejin.im/post/5ace078cf265da23994ee493?utm_source=gold_browser_extension)
