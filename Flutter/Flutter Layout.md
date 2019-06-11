#### 核心概念

##### Unbounded Constraints
---
> either the maximum width or the maximum height is set to double.INFINITY


**ScrollView** 和它的子类比如**ListView** 或**GridView**是常见的 Unbounded Constraints. 也就是在某一个方向没有限制大小，其它的widget只要能够设置width或height为double.INFINITY的也算，有时也会用as big as possible 来描述这些widhgets。


##### Flex
---
> when in bounded constraints, try to be as big as possible in that direction.
>
> when in unbounded constraints, try to fit their children in that direction.
> 

当在有限的空间内，会撑满整个空间；如果在一个 unbounded constraints 容器里，就匹配子widget 的大小。


最常见的是Row和Column，如果不嫌麻烦的话，也可以使用**Flex**widget。里面可以放**Flexible**widget，也可以不是，如果有**Flexible**widgets会把剩余空间计算出来分配给这些widgets；

##### Flexible
---

跟Flex搭配使用，Flexible可以用来声明使用百分之多少的空间，比如flex=1 也就是1/all,如果有两个widgets，另一个也是1，那么all=2，每个widget分配到50%的空间。

**Expanded**是最常见的**Flexible** widget，它会填满主方向上可用的空间（比如Row的水平空间或Colimn的垂直空间）

#### 主要Widgets

##### Container

> Containers with no children try to be as big as possible unless the incoming 
constraints,are unbounded, in which case they try to be as small as possible.
Containers with children size themselves to their children.The width, height, and constraints arguments to the constructor override this.


```
return MaterialApp(
  home: Scaffold(
    body: Container(
      color: Colors.yellow,
    ),
  ),
);
```

这是一个没有孩子的container，因此它会表现地尽量大，就想这样：

![](https://limboy.me/image/Flutter-container.png)

如果设置了width或height，则会根据设置的来表现：


```
return MaterialApp(
  home: Scaffold(
    body: Container(
      color: Colors.yellow,
      width: 100,
      height: 100,
    ),
  ),
);
```
![](https://limboy.me/image/Flutter-container-width.png)

如果有size，则会以child的size为准

```
return MaterialApp(
  home: Scaffold(
    body: Container(
      color: Colors.yellow,
      child: Text('hello'),
    ),
  ),
);
```

除此之外，还可以设置padding、margin、child的对齐方式，等等。


#### Stack
--

Stack有点像css的绝对布局，可以在上面盖一些widgets，比如profile页的背景图上放一些个人信息。

> Each child of a Stack widget is either positioned or non-positioned.
> Positioned children are those wrapped in a Positioned widget that has at least one non-null property.
> The stack sizes itself to contain all the non-positioned children, which are positioned according to alignment.
> 

Stack的children如果没有用Positioned修饰的话，就会用Stack的fit和aligment来帮它们找到合适的位置。


```
Stack(
  fit: StackFit.loose,
  alignment: Alignment.center,
  children: <Widget>[
    Text('world'),
    Positioned(
     bottom: 10,
     child: Text('hello'),
   )
 ],
),
```
![](https://limboy.me/image/flutter-stack-1.png)

StackFit.loose 的意思是：如果child size不比Stack的大，而是child的size，而如果设置StackFit.expand,则会让所有非 Positioned的 widgets使用Stack的size


Text('world')现在看起来就跟Stack一样大了，所以看起来就是alignment.center 没有生效。




#### Row and Column

---


它们都是Flex widgets，Row可以将children横着放，column可以将children竖着放。


**crossAxisAlignment**表示要如何对齐另一侧，比如横着一排的widgets，垂直方向上它们应该顶部对齐还是居中对齐。


**mainAxisSize**默认是*MainAxisSize.max*,如果想让它变成Row或Column的真实高度，可以将它设置为*MainAxisSize.min*;


#### SizeBox

---

使用它可以得到一个确定尺寸的widget。

#### SafeArea
---

使用**SafeArea**可以让child widget在顶部和底部腾出足够的空间方便处理iPhone X这类手机。


## 原则

<p style="color:#333;font-size:18px;font-weight:600">不要在Flex widget里设置unbounded constraints</p>


**Column** 是Flex widget，所以在里面放**ListView**的话，系统是不会答应的。


```
return MaterialApp(
  home: Column(
    children: <Widget>[
      ListView.builder(
        itemBuilder: (context, index){
          return Text('hello');
        },
        itemCount: 3,
      )
    ],
  ),
);
```

系统会给出这样的error：

```
flutter: The following assertion was thrown during performResize():
flutter: Vertical viewport was given unbounded height.
...
```

因为Column作为Flex它不知道应该如何安放一个**as big as possible**的widget。解决办法也很简单，只要设置ListView的**shrinkWrap = true**即可。这就是告诉ListView把自己尽可能地缩小。

可以在Column或Row里使用Expanded，因为它是Flexible，就应该待在Flex里面。


#### 不要在unbounded widgets里设置flex为不等于0的数值

----
因为空间无限，如果两个Flexible 分别设置为1和2的话，那么Flex根本不知道该如何将空间划分为1：2，如果真要这么做的话，会收到这样的error：

```
RenderFlex children have non-zero flex but incoming height constraints are 
unbounded.
```

> 测试
> 
> 下面这段代码会让Hello World被包裹在中间的小方块里吗？
> 

```
return MaterialApp(
  home: Container(
    alignment: Alignment.center,
    constraints: BoxConstraints.tight(Size(100, 100)),
    decoration: BoxDecoration(color: Colors.yellow),
    child: Text('Hello World'),
  ),
);
```

![](https://limboy.me/image/Flutter-squrebox.png)

答案是，不会，它会变成这样：

![](https://limboy.me/image/Flutter-squarebox-real.png)

不是设置了constraints系统就要按照这个constraints来，在经过计算之后，系统会发现这个constraints无法满足需求，而被舍弃，具体过程如下：


**Container**的build方法里，发现有设置过constraints，最终会返回一个BoxConstraints：

```
BoxConstraints(
  minWidth: minWidth.clamp(constraints.minWidth, constraints.maxWidth),
  maxWidth: maxWidth.clamp(constraints.minWidth, constraints.maxWidth),
  minHeight: minHeight.clamp(constraints.minHeight, constraints.maxHeight),
  maxHeight: maxHeight.clamp(constraints.minHeight, constraints.maxHeight)
)
```

这里的clamp方法指的是当minWidth值比左边的值小时，去左边值，当右边的值大时，取右边值，因为parent的constraints也就是screen size是固定的，因此，minWidth在跟他们比较之后，还是使用了它们的值。

正确的做法是外面套一层**Center**或 **Align Widget**


如何得到父widget的constraints？

---

使用LayoutBuilder，有时会需要这些信息来做一些显示上的调整。

```
var container = Container(
  // Toggling width from 100 to 300 will change what is rendered
  // in the child container
  width: 100.0,
  // width: 300.0
  child: LayoutBuilder(
    builder: (BuildContext context, BoxConstraints constraints) {
      if(constraints.maxWidth > 200.0) {
        return Text('BIG');
      } else {
        return Text('SMALL');
      }
    }
  ),
);
```

#### 如何获取屏幕尺寸

----


使用MediaQuery，除了size外，还能拿到devicePixelRatio 之类的device信息。

















