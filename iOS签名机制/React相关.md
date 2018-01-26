### react中更新组件的方法

- 1.ReactDOM.render()

- 2.state的变化,在调用setState() 方法的时候，系统会调用render()方法来刷新界面。




### 组件的生命周期

+  componentDidMount:
  
	> 组件已经被加载（在DOM中完成渲染）

+  componentWillUnmount:

   > 组件即将被卸载
   
+  constructor


#### 关于State需要注意的地方：

- 在除constructor的方法以外需要更新State属性值时，只能通过调用this.setState();

- state触发刷新动作是异步的，

 > 如果要在setState方法后，直接取用更改后的state值，需要利用setState的第二个参数，传入一个回调函数。

 > 另外一种方式则是用componentDidUpdate()这个生命周期方法，把确定state更新后要执行的代码放在里面
 
 **setState方法与包含在其中的执行是一个很复杂的过程，这段程式码从React最初的版本到现在，也有无数次的修改。它的工作除了要更动this.state之外，还要负责触发重新渲染(render)，这里面要经过React核心中diff演算法，最终才能决定是否要进行重渲染，以及如何渲染。而且为了批次与效能的理由，多个setState呼叫有可能在执行过程中还需要被合并，所以它被设计以异步的或延时的来进行执行是相当合理的。**
 
 那么setState会在何时以同步的方式来执行，也就是立即更动this.state？答案是在React库控制之外时，它就会以同步的方式来执行，在下面两篇文章中，都有类似的例子:
 
+ [链接](https://www.bennadel.com/blog/2893-setstate-state-mutation-operation-may-be-synchronous-in-reactjs.htm)

但大部份的使用情况下，我们都是使用了React库中的表单组件，例如select、input、button等等，它们都是React库中人造的组件与事件，是处于React库的控制之下，在这个情况下，setState就会以异步的方式执行。所以一般来说，我们会认为setState就是异步执行，并不是用原始码来看它是不是有使用像setTimeout或Promise之类的方式转为JavaScript的异步执行方式，而是以它在React库的控制之下，以执行行为与顺序来认定。

#### setState可能会引发不必要的渲染(renders)
>state本身的设计是无法直接更改，setState的设计是用来更动state值，也会触发重新渲染(re-render)，按照逻辑就是反正不管如何，只要开发者呼叫setState，React就去作整个视图的重新渲染就是。所以setState必定会作重新渲染的执行，只是要如何渲染是由React来决定。

>重新渲染(re-render)指的主要是页面上视图(View)的重新再呈现，这是React原本的核心设计，但这个设计是有一些问题的。最主要的是state(状态)并不一定单纯只用来记录与视图(View)有关的状态，也有可能是某个内部控制用的属性值，或是只套用在内部使用的资料。当你改变了这些与视图无关的state(状态)值，以现在的React设计来说，照样要触发重新渲染的执行过程，这在某些复杂的应用时，由于造成不必要的渲染，也有可能造成效能上的问题。

>当然，React提供了shouldComponentUpdate方法让开发者可以自行判断，自行提供对应的解决方式。也有Performance Tools可以进行剖析检测。算得上是一些补强的作法。
>

#### setState无法完全掌控应用中所有组件的状态
> state(状态)是独立于每个组件内部的，而且它是个不能直接更动的对象，这个设计当然是为了要保持组件的封装与独立性，但所以如果当要开发一个复杂的应用时，必定需要使用那些能掌控所有组件资料，以及能提供各组件间资料互动的函式库，例如Flux, Redux或MobX等等。

> React组件目前只能透过各种生命周期的方法，与外部资源、计时器或DOM事件来进行挂勾(Hook)，这些都无法直接使用setState方法来进行管理，因此setState并没有办法完全掌控一个应用中所有组件的状态，它比较像是每个组件中的都有的一种接口方法，单纯要依靠setState方法来管控整个React应用，完全是不足够的


 