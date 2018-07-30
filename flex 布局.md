###### flex属性简介：
flex的属性分为两类：父容器属性和子item属性

#### 父容器属性

| 属性 | 作用 | 特性分类|
|-----|-----|--------|
| 	flex-direction|定义子项在容器内的排列方向|排列|
|	flex-wrap	   	 |定义子项在容器内的换行效果|排列|
|	flex-row 		 |flex-direction和flex-wrap的复合属性|排列|
|	justify-content| 定义子项在容器内水平对齐方式|对齐|
|	align-items	 | 定义子项在容器内的垂直对齐方式|对齐|
|	align-content | 定义多行子项在容器内整体垂直对齐方式|对齐|

#### 子item属性
| 属性 | 作用 | 特性分类|
|-----|-----|---------|
| 	order	|	定义子项们的排列方式|	排列|
|	flex\-grow|定义子项宽度之和不足父元素宽度时，子项拉伸的比例|占地面积|
|	flex\-shrink|定义子项宽度之和超过父元素宽度时，子项缩放的比例|占地面积|
|flex\-basis|定义子项的初始宽度，flex-grow和flex-shrink以此为基础缩放|占地面积|
|align\-self|定义单个子项与其他项目不一样的对齐方式|对齐|

###### Flex布局是Flexible Box的缩写，意思是弹性布局，用来为盒子模型提供最大的灵活性，分为flex和inline-box；


##### flex：将对象作为弹性伸缩盒显示
> flex 默认从左边开始布局，所以指定宽度之后，子item就开始从左往右依次布局。


##### inline-flex：将对象作为内联块级弹性伸缩盒显示
> inline-flex 将对象作为内联级容器，它会根据子item的大小自动适应宽度和高度，
> 

##### flex 父容器属性

> 1. flex direction: 作用于子元素在主轴的排列方向
> 
	> + row (default) - 主轴水平
	> + row-reverse 主轴水平反转
	> + column - 主轴垂直
	> + column- reverse 主轴垂直反转
> 2. flex wrap 定义item在主轴方向的换行效果
> > + nowrap 不换行
> > + wrap 自动换行
> > + wrap-reverse 自动换行但方向反转。
> 
> 3. flex-flow 该属性是flex-direction 和 flex-wrap两个属性的缩写，默认值为 row wrap，即水平自动换行。
> 4. justify content 定义item在主轴方向的对齐方式
> > + flex-start 从左往右
> > + flex-end 从右往左
> > + center 主轴居中
> > + space-between 两头的元素贴着开始的方向，中间间距弹性伸缩
> > + space-around 两头的元素与开始排列的轴之间的距离与元素之间的间距比例伸缩。

> 5. align-items 定义子元素在副轴的对其方式
> > + flex-start 从上往下
> > + flex-end 从下往下
> > + center 居中
> > + baseline 
> > + stretch 拉伸
> 6. 定义多行子item在交叉轴的对其方式

![](/Users/shuaike/Desktop/1843940-a2550969518e5dee.png)

##### 子item属性

> 1. order 定义子item的排列顺序，默认为0
> 2. flex-grow 定义子item的宽度小于容器时，子item的拉伸比例，默认为0，表示不拉伸
> 3. flex-shrink 定义item的宽度之和超过父容器时，子item的缩放比例，默认为1

![](/Users/shuaike/Desktop/1843940-5e8cb444fca99e01.png)

> 4. flex-basis 设置子item的宽度（flex-direction = row）或 高度（flex-direction = column）。如果设置改属性，那么flex-grow/flex-shrink 以该大小进行缩放。
> 5. flex 该属性是 flex-grow flex-shrink flex-basis 的简写，默认值是：0 1 auto，其后两个属性可选。
> 6. align-self ： 定义单个子item在交叉轴上的对齐方式，会覆盖默认的对其方式，默认为auto，表示继承父容器的align-items属性，如果没有父容器，则等同于stretch（stretch 伸缩项目在交叉轴方向占满伸缩容器，前提是不设置交叉轴方向的尺寸）

![](/Users/shuaike/Desktop/1843940-2fd61e76669d03f6.png)

##### 定位
display 属性定义子 item如何在父容器内布局，有两种类型：relative和absolute

1. display:relative
> 
>  在相对布局中，布局子item时需要用到margin、border、padding等盒子模型，其中，border和padding会显示在item的背景色中，而margin则不会，默认是透明色
![](/Users/shuaike/Desktop/1843940-32162efe1b37e37b.png)

2. display:absolute

> 使用绝对定位的item，如果父容器设置了relative布局，则以父容器位参考，如果父容器不设置relative布局，则以window做参考，
> 其中，设置布局的属性为：left、right、top、bottom、start、end

	




	


