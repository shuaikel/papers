##### AndroidManifest 文件分析

AndroidManifest xml 是每一个Android应用程序必须拥有的全局配置文件，该文件描述了应用程序中用到的组件，权限，第三方引用等。不如我们需要的Activity，Service等组件都需要在该配置文件中配置，

> \<manifest\>:该标签是Androidmanifest文件的根元素，它必须包含一个<application>元素，并指定xmlns、package属性，xmlns指定Android的命名空间，默认为："http://schemas.android.com/apk/res/android"，package则是应用包名，也是一个应用进程的默认名称，在创建Android项目的时候，通常会让你填写一个包名，而你填写的那个包名默认就是这个package属性的默认值。另外还有常用的属性"VersionCode"，是给设备程序识别版本用的，必须是一个整数值，代表当前应用程序更新过多少次，VersionName则是给用户查看版本的。
> 
> \<uses-permission\>: 该标签用来指定权限，在Android系统中，需要指定相应的权限才能使用相应的功能，例如，当我们需要访问网络时，则必须配置android.permission.INTERNET
> 
> \<permisson\>:该标签用来指定给\<uses-permission\>标签使用的具体权限，如果你的应用程序需要提供一些数据或者可调用的代码，你就可以使用该标签指定访问你程序的权限。
> 
> \<instrumentation\>:该标签声明一个Instrumentation类，这个类能监视应用程序与系统的交互，并且instrumentation会先于应用程序中的其他组件被实例化。
> 
> \<uses-sdk\>:该标签用来声明应用程序中需要使用的SDK版本
> 
> \<uses-configuration\>: 该标签用来指明应用程序需要什么样的硬件和软件功能，例如：程序可能指定需要一个物理键盘或者特定的导航装置，比如，轨迹球
> 
> 	\<uses-feature\>:声明程序需要用到的单一硬件或软件功能。Google Play 就是通过检查程序的manifest文件中的\<uses-feature\>元素，然后根据用户的设备来决定显示或隐藏应用程序给用户。
> 
> 
> \<uses-library\>：指定了应用程序必须链接的共享库，该标签告诉系统将包含在库中的代码用加载器加载
>
> \<supports-screens\>:指定你应用程序所支持的屏幕大小，主要的属性有android:resizeable(屏幕自适应),android:smallScreens,android:normalScreens,android:largeScreens
> 
> \<application\>: 该标签为应用程序配置的根元素，位于manifest标签的下级，它包含了与应用相关的配置元素，常用的属性有，应用名android:label,应用图片android:icon等。
> 
> \<activity\>:该标签用来声明应用中用到的Activity组件，每一个程序中的Activity都必须在manifest文件中进行声明，否则系统识别不了Activity，常用的属性有android:name,Activity对应类名。另外Activity还可以包含\<intent-filter\>标签。
> 
> \<service\>：该标签用来声明Service组件，主要属性有android：name，Service类名。
> 
> \<receiver\>:该标签为Broadcast Receiver组件的声明标签，用来定义一个具体的广播接收器。主要属性有android:name，具体的类名
> 
> \<provider\>:该标签是Content Provider（内容提供者）的声明标签，主要属性有annroid：name具体类名，android：authorities，对指定URL授予权限标识。
> 
> 
> 
> 
> 


#### 1. Android的核心，Activity
Activity是android四大组件之一，也是Android中最基本的模块之一，几乎所有的Activity都是用来和用户交互的，因此，Activity主要关注于视图窗体的创建（你可以通过setContentView（View））方法来设置你的UI，而且Activity对于用户来说通常都表现为全屏的窗体，当然，你也可以
以其它方式呈现，比如浮动窗体。

##### Activity的生命周期
![](https://images2015.cnblogs.com/blog/675009/201607/675009-20160713164151842-1620929141.png)

Activity是由Activity栈进行管理，当来到一个新的Activity后，此Activity将被加入到Activity栈顶，之前的Activity位于此Activity底部，Activity一般意义上有四种状态：

1. Running状态： 当Activity位于栈顶时，此时正好处于屏幕最前方，此时处于运行状态并可和用户交互的激活状态。
2. Paused状态：当Activity被另一个透明或者Dialog样式的Activity覆盖的状态，此时它仍然与窗口管理器保持连接，系统继续维护其内部状态，它仍然可见，但它已经失去了焦点，故不可与用户交互。
3. Stopped状态：当Activity不可见时，Activity处于Stopped状态，当Activity处于此状态时，一定要保存当前数据和当前应用的UI状态，否则一旦Activity退出或者关闭时，当前的数据和UI的状态就会丢失
4. Killed状态：Activity被杀掉以后或者被启动之前，处于Killed状态，这时Activity已从Activity堆栈中移除，需要重新启动才可以显示和使用。

四种状态中，Running状态和paused状态是可见的，Stopped状态和Killed状态时不可见的。

1. onCreate():该方法是在Activity第一次被创建的时候调用的，这个方法通常用来做一些常规的设置，比如创建试图，绑定数据到list等，这个方法还提供了一个Bundle对象来保存先前冻结的状态，当然，前提是你之前已经将你需要冻结的内容放到了Bundle中，之后总会调用onStart（）方法，并且在调用这个方法之后，是不能被系统意外杀死的。

2. onRestart():从名字中就能看出，在Activity被停止后，如果需要重新启动，则会调用这个方法，之后会调用onStart()方法。

3. onStart(): 该方法在Activity将要对用户可见时调用，如果Activity将显示在前台，接着调用onResume（），如果Activity将变隐藏，则调用onStop（）方法，不能被系统意外杀死。

4. onPause()：该方法是在系统准备恢复其他Activity时调用，这个方法通常用来提交未保存变化的持久化数据，停止动画或其他可能消耗CPU的操作，由于在这个方法返回之前，下一个Activity是无法被恢复的，所以这个方法的实现不宜做耗时的操作。如果调用了这个方法之后，Activity又打算重新返回到前台，则会调用onResume（）方法，如果Activity变得对用户不可见，则调用onStop()方法，在系统极端内存的情况下可以被杀死。

5. onStop(): 该方法在Activity不再对用户可见时调用，因为其他Activity已经恢复并且正在覆盖当期的Activity，这个可能发生在一个新的Activity正在启动，而已经存在的Activity又被带到了这个Activity的前面，或者这个activity正在被销毁，调用这个方法之后，可能会被系统意外地杀死。

6. OnDestory():该方法是在Activity被销毁之前最后调用的一个方法，这个可能发生在Activity被完成的时候。   



##### 上图中的生命周期函数需要注意以下几点：

1. Activity实例是由系统自动创建，并在不同的状态期间回调相应的方法，一个最简单的完整地Activity生命周期会按照如下顺序回调：

onCreate->onStart-> onResume->onPause->onStop->onDestroy.称之为entire lifetime.

2. 当执行onStart回调方法时，Activity开始被用户所见（也就是说，onCreate时用户是看不到此Activity的，那用户看到的是哪个？当然是此Activity之前的那个Activity），一直到onStop之前，此阶段Activity都是被用户可见，称之为：visible lifetime

3. 当执行到onResume回调方法时，Activity可以响应用户交互，一直到onPause方法之前，此阶段Activity称之为visible lifetime。







##### 彻底理解ldpi、mdpi、hdpi、xhdpi、xxhdpi
在项目中的这四个文件夹存放的图片资源与屏幕密度有关，和具体的分辨率无关。

他们之间的关系为：low:medium:high:extra-high:extra-extra-high=3:4:6:8:12

就是说这五个文件夹的比例为 3:4:6:8:12
比如我有一个480*800的4寸手机，这个手机的屏幕密度按照Google的说法，就属于密度为high level的水平（通过分辨率和屏幕尺寸计算密度，然后Google自己有一套标准说你位于哪个范围属于哪个level的密度水平），然后这个手机的应用在使用图片的时候，就会去hdpi下找，并且以这个文件夹的图片为标准，也就是说比如我的应用去取一张aa.png的图片，这个图片的原图尺寸为30\*30，恰好hdpi下面有一张，那这张图片显示到屏幕上以后，他的显示尺寸长宽都是30px，那么问题来了，如果我的hdpi下没有这张图片，而只在xhdpi下有这张图片，图片的原始尺寸为30\*30，那请问显示到屏幕上的图片的尺寸会是多大呢？

对于这个问题的解释我们需要先知道：如果我的屏幕是hdpi，结果我的图片是放到了xhdpi下，那系统会把这张图片进行缩小显示，也就是说我的xhdpi下放了一张30\*30的图片，那显示在hdpi屏幕上肯定要比30\*30小，这样才能保证说大小屏幕界面显示效果是一致的，因为密度小的手机显示需要一张图片要比密度大的手机显示同一张图片的面积要打，要想显示面积一样就必须要把图片弄好缩放。

1. 800\*480、480\*320、320\*240分辨率的手机需要不同的图片资源
: 以现在的情况为例，最为节省的话，一般开发Android APP的时候会先考虑hdpi和mdpi的图片资源，所以也需要设计两套图的，很多为了节省工作量直接用一套hdpi的图片资源，在mdpi的时候直接压缩，这样图片质量会损失很大，所以最好还是尽量使用两套资源图片。

2. 如何设计三套图片资源来满足这三种分辨率的手机，这三套图片资源之间有什么样的大小比例关系？
: 视觉在根据交互原型进行设计的时候，可以考虑以mdpi，也就是320x480为蓝本进行设计，因为Android一般采用dp为单位，而我们设计的时候一般是px为单位的，这个就涉及到一个单位转换的问题，而在mdpi下，px和dp是1：1的关系，这样在标注坐标的时候能够很方便的进行单位转换，例如我们以320x480为蓝本的话，在photoshop测量的间距为10px，我们在给到RD时的坐标可以直接标注为10dp；因为dp的单位是可以程序自己去适配不同dpi屏幕的，所以就算设计三套不同dpi的图，一般也只用对mdpi的设计图进行坐标定位，这个坐标的标注可以用在所有dpi的资源上；

因为现在Android又添加了xhdpi的支持，假如我们在设计有（xhdpi\hdpi\mdpi\ldpi）四套不同dpi的图片资源，因为之前提到以mdpi为蓝本做第一个dpi的设计，相应的我们把mdpi的比例设定为1，相应不同dpi的图片资源尺寸的比例关系可以是xhdpi:hdpi:mdpi:ldpi等于2:1.5:1:0.75，也就是说，第一套图为mdpi的图片资源，xhdpi可以大小调整到200%，hdpi可以调整到150%，ldpi则是75%；















