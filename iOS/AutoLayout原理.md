#### AutoLayout 

AutoLayout 是苹果公司提供的一个基于约束布局，动态计算视图大小和位置的库，并且已经集成到了XCode开发环境里。

在引入Auto Layout这种自动布局方式之前，iOS开发都是采用手动布局的方式，而手动布局的方式，原始落后、界面开发维护效率低，对从事过前端开发的人来说更是难以适应，所以，苹果需要提供更好的界面引擎来提升开发者的体验，Auto Layout随之出现。


苹果公司早在iOS6系统时就引入了Auto Layout,但是直到现在还有很多开发者迟迟不愿使用它，其原因就在于对其性能的担忧，即使后来，苹果公司推出了在Auto Layout基础上模仿前端Flexbox 布局思路的UIStackView工具，提升了开发体验和效率，也无法解决开发者们对其性能的顾虑。


那么，AutoLayout到底是如何实现自动布局的，这种布局算法真的会影响性能吗？


另外，苹果公司在WWDC 2018的” WWDC 220 Session High Perform“ Session 中介绍说：iOS12将大幅提高Auto Layout性能，使滑动达到满帧，这又是如何做到的呢？

上图记录了两个时间点：

+ 一个是1997年，Auto Layout用到布局算法Cassowary被发明出来；

+ 另一个是2011年，苹果公司将Cassowary算法运用带自己的布局引擎Auto Layout中；

Casswoary 能够有效解析线性等式系统和线性不等式系统，用来表示用户界面中那些相等关系和不等关系，基于此，Casswary开发了一种规则系统，通过约束来描述视图间的关系。约束就是规则，这个规则能够表示出一个视图相对于另一个视图的位置；

由于 Cassowary 算法本身的先进性，更多的开发者将 Cassowary 运用到了各个开发语言中，比如 JavaScript、.NET、Java、Smalltalk、C++ 都有对应的库。 


### Auto Layout的生命周期

Auto Layout不只有布局算法Casswary,还包含了布局在运行时的生命周期等一整套布局引擎系统，用来统一管理布局的创建、更新和销毁，了解Auto Layout的生命周期，是理解它的性能相关话题的基础，这样，在遇到问题时，我们才能从根上找到原因，从而避免或改进类似的问题。

这一整套布局引擎系统叫作Layout Engine,是Auto Layout的核心，主导着整个界面布局；

每个视图在得到自己的布局之前，Layout Engine 会将视图、约束、优先级、固定大小通过计算转换成最终的大小和位置。在 Layout Engine 里，每当约束发生变化，就会触发 Deffered Layout Pass，完成后进入监听约束变化的状态。当再次监听到约束变化，即进入下一轮循环中。整个过程如下图所示：

![](https://static001.geekbang.org/resource/image/7c/1c/7ca0e14ef02231c9aba7cb49c7e9271c.png)

图中， Constraints Change 表示的就是约束变化，添加、删除视图时会触发约束变化。Activating 或 Deactivating，设置 Constant 或 Priority 时也会触发约束变化。Layout Engine 在碰到约束变化后会重新计算布局，获取到布局后调用 superview.setNeedLayout()，然后进入 Deferred Layout Pass。

Deferred Layout Pass 的主要作用是做容错处理。如果有些视图在更新约束时没有确定或缺失布局声明的话，会先在这里做容错处理。

接下来，Layout Engine 会从上到下调用 layoutSubviews() ，通过 Cassowary 算法计算各个子视图的位置，算出来后将子视图的 frame 从 Layout Engine 里拷贝出来。

在这之后的处理，就和手写布局的绘制、渲染过程一样了。所以，使用 Auto Layout 和手写布局的区别，就是多了布局上的这个计算过程。那么，多的这个 Cassowary 布局，就是在 iOS 12 之前影响 Auto Layout 性能的原因吗？

![](https://static001.geekbang.org/resource/image/ef/16/ef9b42666a226226b8f5fee6d9621b16.png)


上图是 WWDC 2018 中 [202 Session](https://developer.apple.com/videos/play/wwdc2018/202/) 里讲到的 Auto Layout 在 iOS 12 中优化后的表现。可以看到，优化后的性能，已经基本和手写布局一样可以达到性能随着视图嵌套的数量呈线性增长了。而在此之前的 Auto Layout，视图嵌套的数量对性能的影响是呈指数级增长的。

在 1997 年时，Cassowary 是以高效的界面线性方程求解算法被提出来的。它解决的是界面的线性规划问题，而线性规划问题的解法是 

[Simplex 算法](https://en.wikipedia.org/wiki/Simplex_algorithm)。单从 Simplex 算法的复杂度来看，多数情况下是没有指数时间复杂度的。而 Cassowary 算法又是在 Simplex 算法基础上对界面关系方程进行了高效的添加、修改更新操作，不会带来时间复杂度呈指数级增长的问题。

![](https://static001.geekbang.org/resource/image/b2/89/b2e91f2bdcb60d4a7fa4b6dbdaebca89.png)

<center><p>Auto Layout 在兄弟视图独立开布局的情况</p></center>

可以看到，兄弟视图之间没有关系时，是不会出现性能呈指数增加问题的。这就表示 Cassowary 算法在添加时是高效的。但如果兄弟视图间有关系的话，在视图遍历时会不断处理和兄弟视图间的关系，这时会有修改更新计算。

由此可以看出，Auto Layout 并没有用上 Cassowary 高效修改更新的特性。

实际情况是，iOS 12 之前，很多约束变化时都会重新创建一个计算引擎 NSISEnginer 将约束关系重新加进来，然后重新计算。结果就是，涉及到的约束关系变多时，新的计算引擎需要重新计算，最终导致计算量呈指数级增加。

总体来说， 
iOS12 的 Auto Layout 更多地利用了 Cassowary 算法的界面更新策略，使其真正完成了高效的界面线性策略计算。
