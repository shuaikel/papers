######  state 的初始化

state 是与每个组件实例自身相关的，我们通常在constructor（）中，通过this.state来设置其初始值：

```
！注意点：
1. 在constructor（）构造器中必须首先调用super（props），否则在构造器中this.props是未定义的，可能引发bug；
2. constructor()是唯一直接设置this.state的地方，其他地方都通过setState（）方法设置state；
3. 在constructor（）中不能使用setState（）方法来设置state；
4. 尽量避免将props的值拷贝到state中，因为props修改时，对应state的值并不会同步修改

```

###### 被遗弃的hook方法

实际上，在官方的最新文档中，有几个生命周期方法已被标记为UNSAFE

+ UNSAFE_componentWillMount()
+ UNSAFE_componentWillUpdate()
+ UNSAFE_componentWillReceiveProps()

这几个方法对应的无前缀方法如下：

+ componentWillMount()
+ componentWillUpdate()
+ componentWillReceiveProps()

![](https://user-gold-cdn.xitu.io/2018/6/20/1641b85196eca8e6?imageslim)

这个流程中我们可以看到下面这个方法：

```
static getDerivedStateFromProps(props, state)

```

这是在 React 16.3 新加入的方法，主要是为了避免在 UNSAFE_componentWillReceiveProps 中使用 setState() 而产生的副作用。关于这个方法，需要注意几点：

+ 这个方法在每次 render() 之前都会调用；
+ 这是一个 static 方法，即类方法；
+ 它返回一个对象来更新 state，或者返回 null，什么也不做











