Flutter是Google开发的一套全新的跨平台、开源UI框架，支持iOS、Android系统开发，并且是未来新操作系统Fuchsia的默认开发套件。自从2017年5月发布**第一个版本**以来，目前Flutter已经发布了近60个版本，并且在2018年5月发布了第一个**“Ready for Production Apps”**的Beta 3版本，6月20日发布了第一个**“Release Preview”**版本。


#### 初始Flutter


Flutter的目标是使同一套代码同时运行在Android和iOS系统上，并且拥有媲美原生应用的性能，Flutter甚至提供了两套控件来适配Android和iOS（滚动效果、字体和控件图标等等）为了让App在细节处看起来更像原生应用。

在Flutter诞生之前，已经有许多跨平台UI框架的方案，比如基于WebView的Cordova、AppCan等，还有使用HTML+JavaScript渲染成原生控件的React Native、Weex等。

基于WebView的框架优点很明显，它们几乎可以完全继承现代Web开发的所有成果（丰富得多的控件库、满足各种需求的页面框架、完全的动态化、自动化测试工具等等），当然也包括Web开发人员，不需要太多的学习和迁移成本就可以开发一个App。同时WebView框架也有一个致命（在对体验&性能有较高要求的情况下）的缺点，那就是WebView的渲染效率和JavaScript执行性能太差。再加上Android各个系统版本和设备厂商的定制，很难保证所在所有设备上都能提供一致的体验。


为了解决WebView性能差的问题，以React Native为代表的一类框架将最终渲染工作交还给了系统，虽然同样使用类HTML+JS的UI构建逻辑，但是最终会生成对应的自定义原生控件，以充分利用原生控件相对于WebView的较高的绘制效率。与此同时这种策略也将框架本身和App开发者绑在了系统的控件系统上，不仅框架本身需要处理大量平台相关的逻辑，随着系统版本变化和API的变化，开发者可能也需要处理不同平台的差异，甚至有些特性只能在部分平台上实现，这样框架的跨平台特性就会大打折扣。



**Flutter则开辟了一种全新的思路，从头到尾重写一套跨平台的UI框架，包括UI控件、渲染逻辑甚至开发语言。渲染引擎依靠跨平台的Skia图形库来实现，依赖系统的只有图形绘制相关的接口，可以在最大程度上保证不同平台、不同设备的体验一致性，逻辑处理使用支持AOT的Dart语言，执行效率也比JavaScript高得多。**

<p style="color:#333;font-size:16px;font-weight:400;">
Flutter同时支持Windows、Linux和macOS操作系统作为开发环境，并且在Android Studio和VS Code两个IDE上都提供了全功能的支持。Flutter所使用的Dart语言同时支持AOT和JIT运行方式，JIT模式下还有一个备受欢迎的开发利器“热刷新”（Hot Reload），即在Android Studio中编辑Dart代码后，只需要点击保存或者“Hot Reload”按钮，就可以立即更新到正在运行的设备上，不需要重新编译App，甚至不需要重启App，立即就可以看到更新后的样式。


在Flutter中，所有功能都可以通过组合多个Widget来实现，包括对齐方式、按行排列、按列排列、网格排列甚至事件处理等等。Flutter控件主要分为两大类，StatelessWidget和StatefulWidget，StatelessWidget用来展示静态的文本或者图片，如果控件需要根据外部数据或者用户操作来改变的话，就需要使用StatefulWidget。State的概念也是来源于Facebook的流行Web框架React，React风格的框架中使用控件树和各自的状态来构建界面，当某个控件的状态发生变化时由框架负责对比前后状态差异并且采取最小代价来更新渲染结果。
</p>


### Hot Reload

在Dart代码文件中修改字符串“Hello, World”，添加一个惊叹号，点击保存或者热刷新按钮就可以立即更
新到界面上，仅需几百毫秒：


**Flutter通过将新的代码注入到正在运行的DartVM中**，来实现Hot Reload这种神奇的效果，在DartVM程序中的类结构更新完成后，Flutter会立即重建整个控件树，从而更新界面。但是热刷新也有一些限制，并不是所有的代码改动都可以通过热刷新来更新：


+ 编译错误，如果编译后的Dart代码无法通过编译，Flutter会在控制台报错，这时需要修改对应的代码。

+ 控件类型从StatelessWidget到StatefulWidget的转换，因为Flutter在执行热刷新时会保留程序原来的state，而某个控件从stateless->stateful后会导致Flutter重新创建组件时报错“myWidget is  not a subtype of StatelessWidget”，而从stateful→stateless会报错“type ‘myWidget’ is not a subtype of type ‘StatefulWidget’ of ‘newWidget’”。

+ 全局变量和静态成员变量，这些变量不会在热刷新时更新。

+ 修改了main函数中创建的根控件节点，Flutter在热刷新之后会根据原来的根节点重新创建控件树，不会修改根节点。

+ 某个类从普通类型转换成枚举类型，或者类型的范型参数列表变化，都会使热更新失败。


热更新无法实现更新时，执行一次热启动（Hot Restart）就可以全量更新所有代码，同样不需要重启App，区别是restart会将所有Dart代码打包同步到设备上，并且所有状态都会重置。


#### Flutter 插件

Flutter使用的Dart语言无法直接调用Android系统提供的Java接口，这时就需要使用插件来实现中转，Flutter官方提供了丰富的原生接口封装：

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/android_alarm_manager">android_alarm_manager</a> 访问Android系统的AlertManager


+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/android_intent">android_intent</a>，构造Android的Intent对象。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/battery">battery</a>获取和监听系统电量变化。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/connectivity">connectivity</a>获取和监听系统网络连接状态。


+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/device_info">device info<i class="fa fa-link" aria-hidden="true"></i></a>，获取设备型号等信息。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/image_picker">image_picker<i class="fa fa-link" aria-hidden="true"></i></a> 从设备中选取或者拍摄照片。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/package_info">package_info<i class="fa fa-link" aria-hidden="true"></i></a> 获取App安装包的版本等信息。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/path_provider">path_provider<i class="fa fa-link" aria-hidden="true"></i></a> 获取常用文件路径。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/quick_actions">quick_actions<i class="fa fa-link" aria-hidden="true"></i></a>App图标添加快捷方式，iOS的<a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://developer.apple.com/ios/human-interface-guidelines/extensions/home-screen-actions">eponymous concept<i class="fa fa-link" aria-hidden="true"></i></a>和Android的<a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://developer.android.com/guide/topics/ui/shortcuts.html">App Shortcuts<i class="fa fa-link" aria-hidden="true"></i></a>

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/sensors">sensors<i class="fa fa-link" aria-hidden="true"></i></a> 访问设备的加速度和陀螺仪传感器。


+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/shared_preferences">shared_preferences<i class="fa fa-link" aria-hidden="true"></i></a> App KV存储功能。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/url_launcher">url_launcher<i class="fa fa-link" aria-hidden="true"></i></a> 启动URL，包括打电话、发短信和浏览网页等功能。

+ <a style="color:#16b9a9;font-size:16px;font-weight:400;line-height:1.8rem;" href="https://github.com/flutter/plugins/blob/master/packages/video_player">video_player<i class="fa fa-link" aria-hidden="true"></i></a> 播放视频文件或者网络流的控件。


#### Flutter 包管理

在Flutter，依赖包由**“Pub”**仓库管理，项目依赖配置在pubspec.yaml文件中声明即可（类似于NPM的版本声明），对于未发布的Pub仓库的插件可以使用git仓库地址或文件地址：

```
dependencies: 
  url_launcher: ">=0.1.2 <0.2.0"
  collection: "^0.1.2"
  plugin1: 
    git: 
      url: "git://github.com/flutter/plugin1.git"
  plugin2: 
    path: ../plugin2/
```

以shared_preferences为例，在pubspec中添加代码：

```
dependencies:
  flutter:
    sdk: flutter

  shared_preferences: "^0.4.1"
```

脱字号“^”开头的版本表示**和当前版本接口保持兼容**的最新版，^1.2.3 等效于 >=1.2.3 <2.0.0 而 ^0.1.2 等效于 >=0.1.2 <0.2.0，添加依赖后点击“Packages get”按钮即可下载插件到本地，在代码中添加import语句就可以使用插件提供的接口：


```
import 'package:shared_preferences/shared_preferences.Dart';

class _MyAppState extends State<MyAppCounter> {
  int _count = 0;
  static const String COUNTER_KEY = 'counter';

  _MyAppState() {
    init();
  }

  init() async {
    var pref = await SharedPreferences.getInstance();
    _count = pref.getInt(COUNTER_KEY) ?? 0;
    setState(() {});
  }

  increaseCounter() async {
    SharedPreferences pref = await SharedPreferences.getInstance();
    pref.setInt(COUNTER_KEY, ++_count);
    setState(() {});
  }
...
```


### Dart

---


[Dart](https://dart.dev/)是一种强类型、跨平台的客户端开发语言。具有专门为客户端优化、高生产力、快速高效、可移植（兼容ARM/x86）、易学的OO编程风格和原生支持响应式编程（Stream & Future）等优秀特性。Dart主要由Google负责开发和维护，在2011年10启动项目，2017年9月发布第一个2.0-dev版本。


Dart本身提供了三种运行方式：

+ 使用Dart2js编译成Javascript代码，运行在常规浏览器中（Dart Web）；

+ 使用DartVM直接在命令行中运行Dart代码（DartVM）。

+ AOT方式编译成机器码，例如Flutter App（Flutter）


Flutter在筛选了20多种语言后，最终选择Dart作为开发语言主要有几个原因：


+ 健全的类型系统，同时支持静态类型检查和运行时类型检查。

+ 代码体积优化（Tree Shaking），编译时只保留运行时需要调用的代码（不允许反射这样的隐式引用），所以庞大的Widgets库不会造成发布体积过大。

+ 丰富的底层库，Dart自身提供了非常多的库。

+ 多生代无锁垃圾回收器，专门为UI框架中常见的大量Widgets对象创建和销毁优化。

+ 跨平台，iOS和Android共用一套代码。

+ JIT & AOT运行模式，支持开发时的快速迭代和正式发布后最大程度发挥硬件性能。

在Dart中，有一些重要的基本概念需要了解：

+ 所有变量的值都是对象，也就是类的实例，甚至数字、函数和null都是对象，都继承自Object类。

+ 虽然Dart是强类型的语言，但是显式变量类型声明都是可选的，Dart支持类型推断，如果不想使用类型推断，可以用dynamic类型。

+ Dart支持泛型，List<int>表示包含int类型的列表，List<dynamic>则表示包含任意类型的列表。

+ Dart支持顶层（top-level）函数和类成员函数，也支持嵌套函数和本地函数。

+ Dart支持顶层变量和成员变量。

+ Dart没有public、protected和private这些关键字，使用下划线“_”开头的变量或者函数，表示只在库内可见。参考[库和可见性](https://dart.dev/guides/language/language-tour#libraries-and-visibility)

**DartVM的内存分配策略非常简单，创建对象时只需要在现有堆上移动指针，内存增长始终是线性的，省去了查找可用内存段的过程；**

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/f175a03b.png)

**Dart中类似线程的概念叫做Isolate，每个Isolate之间是无法共享内存的，所以这种分配策略可以让Dart实现无锁的快速分配。**

**Dart的垃圾回收也采用了多生代算法，新生代在回收内存时采用了“半空间”算法，触发垃圾回收时Dart会将当前半空间中的“活跃”对象拷贝到备用空间，然后整体释放当前空间的所有内存：**

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/0b52c944.png)

整个过程中Dart只需要操作少量的**“活跃”**对象，大量的没有引用的“死亡”对象则被忽略，这种算法也非常适合Flutter框架中大量Widget重建的场景。


#### Flutter Framework

Flutter的框架部分完全使用Dart语言实现，并且有着清晰的分层架构。分层架构使得我们可以在调用Flutter提供的便捷开发功能（预定义的一套高质量Material控件）之外，还可以直接调用甚至修改每一层实现（因为整个框架都属于“用户空间”的代码），这给我们提供了最大程度的自定义能力。Framework底层是Flutter引擎，引擎主要负责图形绘制（Skia）、文字排版（libtxt）和提供Dart运行时，引擎全部使用C++实现，Framework层使我们可以用Dart语言调用引擎的强大能力。


##### 分层架构

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/9924fbed.png)


**Fremework的最底层叫做Foundation，其中定义的大部分都是非常基础的。提供给其他所有层使用的工具类和方法，绘制库（Painting）封装了Flutter Engine提供的绘制接口，主要是为了在绘制控件等固定样式的图形时提供更直观、更方便的接口，比如绘制缩放后的位图、绘制文本、插值生成阴影以及在盒子周围绘制边框等等。Animation是动画相关的类，提供了类似Android系统的ValueAnimator的功能，并且提供了丰富的内置插值器。Gesture提供了手势识别相关的功能，包括触摸事件类定义和多种内置的手势识别器。GestureBinding类是Flutter中处理手势的抽象服务类，继承自BindingBase类。Binding系列的类在Flutter中充当着类似于Android中的SystemService系列（ActivityManager、PackageManager）功能，每个Binding类都提供一个服务的单例对象，App最顶层的Binding会包含所有相关的Bingding抽象类。如果使用Flutter提供的控件进行开发，则需要使用WidgetsFlutterBinding，如果不使用Flutter提供的任何控件，而直接调用Render层，则需要使用RenderingFlutterBinding。**

**Flutter本身支持Android和iOS两个平台，除了性能和开发语言上的“native”化之外，它还提供了两套设计语言的控件实现Material & Cupertino，可以帮助App更好地在不同平台上提供原生的用户体验。**


##### 渲染库(Rendering)

Flutter的控件树在实际显示时会被转换成对应的渲染对象（RenderObject）树来实现布局和绘制操作，一般情况下，我们只会在调试布局，或者需要使用自定义控件来实现某些特殊效果的时候，才需要考虑渲染对象树的细节。渲染库主要提供的功能类有：

```
abstract class RendererBinding extends BindingBase with ServicesBinding, SchedulerBinding, HitTestable { ... }
abstract class RenderObject extends AbstractNode with DiagnosticableTreeMixin implements HitTestTarget {
abstract class RenderBox extends RenderObject { ... }
class RenderParagraph extends RenderBox { ... }
class RenderImage extends RenderBox { ... }
class RenderFlex extends RenderBox with ContainerRenderObjectMixin<RenderBox, FlexParentData>,
                                        RenderBoxContainerDefaultsMixin<RenderBox, FlexParentData>,
                                        DebugOverflowIndicatorMixin { ... }
```

**RendererBinding**是渲染树和Flutter引擎的胶水层，负责管理帧重绘、窗口尺寸和渲染相关参数变化的监听。**RenderObject**渲染树中所有节点的基类，定义了布局、绘制和合成相关的接口。RenderBox和其三个常用的子类**RenderParagraph**、**RenderImage**、**RenderFlex**则是具体布局和绘制逻辑的实现类。

在Flutter界面渲染过程分为三个阶段：布局、绘制、合成，布局和绘制在Flutter框架中完成，合成则交由引擎负责。


![](https://p1.meituan.net/travelcube/c44757cf743d9bb3a903cba2346fd34f14239.png)


控件树中的每个控件通过实现**RenderObjectWidget#createRenderObject(BuildContext context) → RenderObject**方法来创建对应的不同类型的**RenderObject**对象，组成渲染对象树。因为Flutter极大地简化了布局的逻辑，所以整个布局过程中只需要深度遍历一次：

![](https://p1.meituan.net/travelcube/d583eb17b749a4563e34efe9df10edfb22637.png)


渲染对象树中的每个对象都会在布局过程中接受父对象的**Constraints**参数，决定自己的大小，然后父对象就可以按照自己的逻辑决定各个子对象的位置，完成布局过程。子对象不存储自己在容器中的位置，所以在它的位置发生改变时并不需要重新布局或者绘制。子对象的位置信息存储在它自己的**parentData**字段中，但是该字段由它的父对象负责维护，自身并不关心该字段的内容。同时也因为这种简单的布局逻辑，Flutter可以在某些节点设置布局边界**（Relayout boundary）**，即当边界内的任何对象发生重新布局时，不会影响边界外的对象，反之亦然：


![](https://p1.meituan.net/travelcube/2d0dd217413eae895c2de9a815cdac9e26812.png)


布局完成后，渲染对象树中的每个节点都有了明确的尺寸和位置，Flutter会把所有对象绘制到不同的图层上：


![](https://p0.meituan.net/travelcube/07ee1aac29f01789bcbf96164534965c26193.png)

因为绘制节点时也是深度遍历，可以看到第二个节点在绘制它的背景和前景不得不绘制在不同的图层上，因为第四个节点切换了图层（因为“4”节点是一个需要独占一个图层的内容，比如视频），而第六个节点也一起绘制到了红色图层。这样会导致第二个节点的前景（也就是“5”）部分需要重绘时，和它在逻辑上毫不相干但是处于同一图层的第六个节点也必须重绘。为了避免这种情况，Flutter提供了另外一个“重绘边界”的概念：


![](https://p1.meituan.net/travelcube/56606ba2bdce470c86c13275cb7e1f7a27564.png)


在进入和走出重绘边界时，Flutter会强制切换新的图层，这样就可以避免边界内外的互相影响。典型的应用场景就是ScrollView，当滚动内容重绘时，一般情况下其他内容是不需要重绘的。虽然重绘边界可以在任何节点手动设置，但是一般不需要我们来实现，Flutter提供的控件默认会在需要设置的地方自动设置。

[](https://tech.meituan.com/2018/08/09/waimai-flutter-practice.html)
#### 控件库(Widgets)















