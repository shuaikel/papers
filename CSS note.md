

### Transitions properties

> Shorthand vc Longhand

在写CSS样式的时候我们经常将多个属性简写（shorthand）的形式，比如设置元素的padding值我们就可以这样写：

padding: 10px 20px 15px 25px; 

同理，transition 也有其简写形式：

transition : all 0.5s 1s linnear;

transition的简写形式对应各属性含义如下：

transition : [property] [duration] [delay] [timing-function];

让我们来看下各个属性具体的含义：

#### transition-property

通常是简写形式的第一个属性，规定应用过渡的CSS属性的名称。比如要修改background，就可以在这里写background，也可以使用all，所有适用的CSS过渡属性上创建过渡效果（不是所有的属性都可以创建过渡效果的）。


#### transition-duration

transition-duration 表示过渡效果的持续时间，transition-duration 设置为3秒是指设置3倍的1000ms。

#### transition-delay

transition-delay 表示过渡效果的延迟时间（效果开始前的等待时间），单位可以是秒或者毫秒，3s表示3秒。100ms表示100毫秒，也可以写成0.1s；

#### transition-timing-function

transition 和 animation 都会使用时间函数（timing function），时间函数有很多可选值。

> 过渡效果（transition）可以应用在很多的属性上，比如position、size、color、border、back-ground-position,但有些属性是不能应用过渡的，比如 font-family
> 

用CSS创建的背景图像（例如渐变效果），不能对其属性使用动画，这意味着浏览器用每帧动画重新创建背景图像，因此不支持。

不过你可以在opacity和 background-position上使用过渡，通过移动背景图（background images） 或者隐藏背景图可以创建很丰富的效果。


#### Timing functions

时间函数（timing function）是用来描述过渡过程中，属性值变化速度的，如果过渡过程中属性一直以一个固定的速度变化，那就回显得千篇一律，不那么生动，有了时间函数（timing function）的加持，我们可以制作更加生动有趣的效果。


## Animations in action

给元素添加animation跟添加transition类似，不过它还需要keyframes。

,element{
<!--animation -->
	animation : x
}

@keyframes animation-name{
<!--keyframes go here-->
}


## Animation Properties

与transition 一样，animation可以使用简写形式，也可以分开单独指定属性值。


#### animation-delay

与transition-delay 类似，此属性表示在开始之前动画的等待时间，在定义多个动画的情况下特别有用。

如果定义的动画是不断循环的，在这种情况下，delay属性并不会每次循环都有效，只有在元素添加上动画效果的时候才有效（只有第一次循环前会有等待时间。其余的就没有了）

实际上可以给这个属性一个负值，比如-1s，动画会直接从第1s开始执行，就好像这1s时间已经过去了。

####




























































