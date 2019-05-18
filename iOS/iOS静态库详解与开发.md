+ 库是共享程序代码的方式，一般分为静态库和动态库

> 静态库：链接时完整的拷贝至可执行文件中，被多次使用就有多份冗余拷贝。
> 
> 动态库：链接时不复制，程序运行时有系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存。
> 
> 但是在项目中如果使用了自己定义的动态库，苹果是不允许上架的，在iOS8.0以后苹果开放了加载.tbd的接口，用于挂载.tbd动态库。

![](https://user-gold-cdn.xitu.io/2018/2/5/16164012817a6f6e?imageslim)

+ 两种形式中.framework的区别

> 如上图所示，静态库的形式包含.a和.framework两种形式，动态库的形式包含.dylib（现在是.tbd格式）和.framework，其中.framework包括私有的.framework和系统的.framework。
> 
> 静态库和动态库都有私有的.framework，但是根本上有所不同，动态库私有的.framework，上架会有机审规则，并且不能动态下载。静态库私有的.framework，类似一个封装的代码块，没有过多的限制。
> 

+ 静态库中.a和.framework的区别

> .a是一个纯二进制文件，.framework中除了有二进制文件之外还有资源文件，.a文件不能直接使用，至少需要.h文件配合，.framework文件可以直接使用，因为本身包含了h文件和其它文件。
> 
> 静态库的优点：

> + 实现程序的模块化，将固定的业务模块化成静态库
> + 方便共享代码，即可以和别人分享你的代码库，但别人又看不到代码的实现
> + 开发第三方SDK的需要，
> 
> 动态库的好处：
> 
> + 使用动态库，可以将最终可执行文件体积缩小
> + 使用动态库，可以多个应用程序共享内存中得到同一份库文件，节省资源
> + 使用动态库，可以不重新编译链接可执行程序的前提下，更新动态库文件达到更新应用程序的目的

+ 静态库的实现
	1. 新建一个静态库工程
		
		> 打开Xcode，点击File\New\Project，选择iOS\FrameWork & Library\Cocoa Touch Static Libraey新建一个静态库工程。
![](https://user-gold-cdn.xitu.io/2018/2/1/1614f948bc5f09ee?imageslim)
在开发中，为了让开发的静态库使用起来更方便，只需要让使用者导入一个文件，便可以访问你提供的接口，并且通过接口进行数据回调。

	2. 导入头文件
		
		> 导入UIKit的头文件，这是创建一个库所需要的，
		点击项目Targets，选择Build Phases，选择Editor\Add Build Phase\Add Headers Build Phase,
		
		> Copy Headers下的Public部分中所添加的类均是对外公开的。 这里有三个分组，Public公开的，Private下的头文件是可以被用户看到的，Project下的文件是私有的，这里建议尽量将文件放在Public和Project下。
		
		![](/Users/luleios/Desktop/E451A089-3788-4BD6-997C-229AC13D8004.png)
		
		
	3. 添加配置
		> 添加配置主要是在Build Setting下操作，点击项目名，选择Targets，搜索pubilc header，双击Public Headers Folder Path，在弹出视图键入如下内容
		
			include/$(PROJECT_NAME)
		截图如下：
		![](/Users/luleios/Desktop/8F42FE06-565E-4428-9B2F-C663D3BE7BF7.png)
		
		> 因为你创建好的静态文件供他人使用，最好禁掉无效代码和debug符号，让用户自己选择对自己的项目有利的部分使用。
在搜索框中分别搜索Dead Code Stripping、Strip Debug Symbol During Copy、Strip Style配置如下：

			+ Dead Code Stripping设置为NO
			+ Strip Debug Symbol During Copy 全部设置为NO
			+ Strip Style设置为Non-Global Symbols
	> 到目前为止，项目的构建已经完成，选择目标为Generic iOS Device，按下command + B进行编译，工程导航栏中Product目录下libRWUIControls.a文件将从红色变为黑色，表明现在该文件已经存在了。右键单击libRWUIControls.a，选择Show in Finder，如下图所示：

![](/Users/luleios/Desktop/0330AFE3-2B8B-4489-A0EC-F9A94A199F6B.png)
>从上图可以看到对外公开的DNStaticFrameWorkDemo.h类，其他实现类均以二进制的形式在ibDNStaticFrameWorkDemo.a中。

	4. 合并静态库
> 分别选择编译目标为虚拟机以及Generic iOS Device，编译运行后，合并通用静态库：打开终端，输入_lipo -create 真机库.a的路径 模拟器库.a的路径 －output 合成库的路径和名字(可以复制模库.a的路径，修改名字).a；回车就可以在模拟库的文件夹中看到新合成的.a文件了_，


### 静态库的使用

1. 导入静态库和.h头文件，

----------


> 注意：制作同时支持armv7，armv7s，arm64，i386，x86\_64的静态库

> 几个重要的概念：

1. ARM：
> ARM处理器，特点是体积小、低功耗、低成本、高性能，所以几乎所有手机处理器都是基于ARM，在嵌入式系统中应用广泛。

2. ARM处理器指令集
> armv6 | armv7 | armv7s | arm64都是ARM处理器的指令集，这些指令集都是向下兼容的，例如armv7指令集兼容armv6，只是使用armv6的时候无法发挥出其性能，无法使用armv7的新特性，从而会导致程序执行效率没那么高。
> 还有两个我们也很熟悉的指令集：i386 | x86\_64 是Mac处理器的指令集，i386是针对Intel通用微处理器32位架构的，x86\_64是针对x86架构的64位处理器。所以当使用iOS模拟器的时候会遇到i386 | X86\_64，iOS模拟器没有arm指令集。







----


### .frameWork静态库
> 新建Xcode工程，需要选择这个Cocoa Touch FrameWork这个模板，

```
注意事项：
+ 如果打包文件中有设置图片的地方，如果还是通过[UIImage imageNamed:]方式设置，图片可能不会显示
+ 图片最好单独打包一个bundle，这个时候设置图片的方式为：
//拿到路径
NSString *path = [[NSBundle mainBundle] pathForResource:@"bundle" ofType:@"bundle"];
//设置图片
UIImage *image = [UIImage imageNamed:@"delete" inBundle:[NSBundle bundleWithPath:path] compatibleWithTraitCollection:nil];
```

lipo -info framework静态库文件下二进制文件的名字，可以查看iOS的CPU框架。

> 合并静态库
> 
> 合并静态库，让模拟器和真机使用一个静态库，命令如下：lipo -create 第一个framework文件下二进制文件的绝对路径 第二个framework文件下二进制文件的绝对路径 -output 最终的二进制文件路径。


