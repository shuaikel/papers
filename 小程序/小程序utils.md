# 微信小程序全局配置

### 小程序根目录下的app.json 文件用来对微信小程序进行全局配置，决定页面文件的路径、窗口表现、设置网络超时时间、设置多个tab等。

属性|类型|必填|描述|最低描述|
----|---|---|---|--------|
pages|String、Array|是|页面路径列表||
window|Object|否|全局的默认窗口表现||
Tabbar|Object|否|底部tab栏的表现|
networkTimeout|Object|否|网络超时时间|
debug|Boolean|否|是否开启debug模式，默认关闭|
functionalPages|Boolean|否|是否启用插件功能也，默认关闭|2.1.0|
subpackages|Object、Array|否|分包结构配置|1.7.3|
workers|String|否|Worker代码放置的目录|1.9.90|
requiredBackgroundModes|String、Array|否|需要在后台使用的能力，如【音乐播放】|
plugins|Object|否|使用到的插件|1.9.6|
preloadRule|Object|否|分包下载规则|2.3.0|
resizable|Boolean|否|iPad小程序是否支持屏幕旋转，默认关闭|2.3.0|
navigationToMiniProgramAppIdList|String、Array|否|需要跳转的小程序列表，详见[wx.navigationToMiniProgram]()|2.4.0|
usingComponents|Objects|否|全局[自定义组件]()配置|开发者工具|
permission|Object|否|小程序接口权限相关配置|微信客户端7.0.0|

---

### pages

用于指定小程序由哪些页面组成，每一项都对应一个页面的路径+文件名 信息。文件名不需要写文件后缀，框架会自动去寻找对应位置的.json，.js,.xwml,.wxss四个文件进行处理。

数组的第一项代表小程序的初始页面，小程序中新增/减少页面，的需要对pages数组进行修改。

### window
用于设置小程序的状态栏、导航条、标题、窗口背景色

属性|类型|默认值|描述|最低版本|
---|---|-----|----|-------|
navigationBarBackgroundColor|HexColor|#000000|导航栏背景颜色|
navigationBarTextColor|String|white|导航栏标题颜色，仅支持black、white|
navigationBarTitleText|String||导航栏标题文字内容|
navigationStyle|String|default|导航栏样式，仅支持以下值：default默认样式，custom 自定义导航栏，只保留右上角胶囊按钮|
backgroundColor|HexColor|#fffff|窗口的背景色||
backgroundTextStyle|String|dark|下拉loading的样式，仅支持dark、light|
backgroundColorTop|String|#ffffff|顶部窗口的背景色，仅iOS支持|微信客户端6.5.16|
backgroundBottom|String|#FFFFFF|底部窗口的背景色，仅iOS支持|微信客户端6.5.16|
enablePullDownRefresh|Boolean|false|是否开启当前页面的下拉刷新|
onreachBottomDistance|Number|50|页面上拉触底事件触发时距页面底部距离，单位为px|
pageOrientation|String|portrait|屏幕旋转设置，支持auto、portrait、landscape|2.4.0(auto)/2.5.0(landSpace)|

### tabBar

如果小程序是一个多tabBar应用（客户端窗口的底部或顶部有tab栏可以切换页面，）可以通过tabBar配置项指定tab栏的表现，以及tab切换时显示的对应页面。

属性|类型|必填|默认值|描述|最低版本|
----|---|---|-----|----|------|
color|HexColor|是||tab上的文字默认颜色，仅支持十六进制颜色|
selectedColor|HexColor|是||tab上的文字选中的颜色，仅支持十六进制颜色|
backgroundColor|HexColor|是||tab的背景色，仅支持十六进制的颜色|
borderStyle|String|否|black|tabbar上边框的颜色，仅支持black、white|
list|Array|是||tab的列表，详见list属性说明，最少2个、最多5个tab|
position|String|否|bottom||tabBar的位置，仅支持bottom、top|
custom|Boolean|否|false|自定义tabBar|

其中list接受一个数组，只能配置最少2个、最多5个tab。tab按照数组的顺序排序，每一项都是一个对象，其属性值如下：

属性|类型|必填|说明|
----|---|---|----|
pagePath|String|是|页面路径，必须在pages中先定义|
text|String|是|tab上按钮的文字|
iconPath|String|否|图片路径，icon大小限制为40kb，建议尺寸为81px*81px,不支持网络图片，当postion为top时，不显示icon|
selectedIconPath|String|否|选中时的图片路径，icon大小限制为40kb，建议尺寸为81px\*81px,不支持网络图片，当position为top时，不显示icon|

### requiredBackgroundModes
> 微信客户端6.7.2及以上版本支持

申明需要后台运行的能力，类型为数组，目前支持以下项目：

> + audio: 后台音乐播放

如：

```
{
	"pages":["pages/index/index"],
	"requiredBackGroundModes":["audio"]
}
```
注：在此处申明了后台运行的接口，开发版和体验版上可直接生效，正式版还需要通过审核。

### usingComponents
> 开发者工具1.02.1810190及以上版本支持

在此处声明的自定义组件视为全局定义组件，在小程序内的页面或自定义组件中可以直接使用而无需再声明。

### permission
> 微信客户端接口权限相关设置。字段类型为object，结构为：

属性|类型|必填|默认值|描述|
----|-----|----|---|---|
scope.userLocation|PermissionObject|否||位置相关权限声明|

### PermissionObject 结构

属性|类型|必填|默认值|描述|
----|-----|----|---|---|
desc|string|是||小程序获取权限时展示的接口用途说明。最长30个字符|

如：

```
{
	"pages":["pages/index/index"],
	"permisssion":{
		"scope.userLocation":{
			"desc" : "你的位置信息将用于小程序位置接口的效果展示"
		}
	}
}
```

### 页面配置

每一个小程序页面也可以使用.json文件来对本页面的窗口进行配置

页面的配置只能设置app.json中部分window配置项的内容，页面中配置项会覆盖app.json的window中相同的配置项。

配置示例：

```
{
	"navigationBarBackgroundColor":"#fff",
	"navigationBarTextStyle":"black",
	"navigationBarTitleText" : "微信接口功能展示",
	"backgroundColor":"#eee",
	"backgroundTextStyle":"light"
}
```


# 微信小程序 注册程序App()函数


### Page(Object)构造器

Page(Object)函数用来注册一个页面，接受一个Object类型参数，其指定页面的初始数据、生命周期回调、事件处理函数等。

Object参数说明：

属性|类型|描述|
----|----|---|
data|Object|页面的初始数据|
onLoad|Function|生命周期回调-监听页面加载|
onShow|Function|生命周期回调-监听页面显示|
onReady|Function|生命周期回调-监听页面初次渲染完成|
onHide|Function|生命周期回调-监听页面隐藏|
onUnload|Function|生命周期函数-监听页面卸载|
onPullDownRefresh|Function|监听用户下拉动作|
onReachBottom|Function|页面上拉触底事件的处理函数|
onShareAppMessage|Function|用户点击右上角转发|
onPageScroll|Function|页面滚动触发事件的处理函数|
onResize|Function|页面尺寸改变时触发，详见[](响应显示区域变化)|
onTabItemTap|Function|当前是tab页时，点击tab时触发|
其它|Any|开发者可以添加任意的函数或数据到	Object参数中，在页面的函数中使用this可以访问|

除了Page,作为高级用法，页面可以像自定义组件一样使用Component来创建，这样就可以使用自定义组件的特性，如behaviors等，具体细节请阅读[](Component 构造器)。




### App()

---

App()函数用来注册一个小程序，接受一个object参数，其指定小程序的生命周期函数等。

object参数说明：

属性|类型|描述|触发|
---|---|---|---|
onLaunch| function| 生命周期函数--监听小程序初始化| 当小程序初始化完成时，会触发onLaunch（全局触发一次）|
onShow|function|生命周期函数--监听小程序显示|当小程序启动，或从后台进入前台显示，会触发onShow|
onHide|Function|生命周期函数--监听小程序隐藏|当小程序从前台进入后台，会触发onHide|
onLoad|Function|生命周期函数--监听页面加载||
onUnload|Function|生命周期函数--监听页面卸载||
onPullDownReresh|Function|页面相关事件处理函数--监听用户下拉动作||
onReachBottom|Function|页面上拉触底事件的处理函数||
onShareAppMessage|Function|用户点击右上角转发||
onPageScroll|Function|页面滚动触发事件的处理函数||
onError|Function|错误监听函数|当小程序发生脚本错误，或者api调用失败时，或触发onError并带上错误信息|
onPageNotFound|Function|页面不存在监听函数|当小程序要打开的页面不存在的情况，会带上页面信息回调该函数
其他|Any||开发者可以添加任意的函数或数据到Object参数中，用this可以访问|

小程序前台、后台定义：当用户点击左上角关闭，或者按了设备Home键离开微信，小程序并没有直接销毁，而是进入了后台；当再次进入微信或者再次打开小程序，又会从后台进入前台。需要注意的是，只有当小程序进入后台一定时间，或者系统资源占用过高，才会被真正的销毁。

setData()参数格式

---

接受一个对象，以key，value的形式表示将this.data中的key对应的值改变成value.
其中key可以非常灵活，以数据路径的形式给出，如array[2],message,a.b.c.d,并且不需要在this.data中预先定义。

### 注意

> 1. 直接修改this.data 而不调用this.setData 是无法改变页面状态的，还会造成数据不一致。
> 2. 单次设置的数据不能超过1024kb，避免一次设置过多的数据。
> 
> 3. 请不要把data中任何一项的value设为undefined，否则这一项将不被设置并可能遗留一些潜在的问题。
> 
> 


# 路由

在小程序中所有的路由全部由框架进行管理

### 页面栈

路由方式|页面栈表现|
------|-------|
初始化|新页面入栈|
打开新页面|新页面入栈|
页面重定向|当前页面出栈，新页面入栈|
页面返回|页面不断出栈，直到目标返回页|
Tab切换|页面全部出栈，只留下新的Tab页面|
重加载|页面全部出栈，只留下新的页面|


### getCurrentPages()

getCurrentPages() 函数用于获取当前页面栈的实例，以数组形式按栈的顺序给出，第一个元素给出，第一个元素为首页，最后一个元素为当前页面。

注意：

+ 不要尝试修改页面栈，会导致路由以及页面状态错误。
+ 不要在App.onLaunch的时候调用getCurrentPages(),此时page还没有生成。


### 路由方式

对于路由的触发方式以及页面生命周期函数如下：

路由方式|触发时机|路由前页面|路由后页面|
------|-------|-------|---------|
初始化|小程序打开的第一个页面||onLoad，onShow|
打开新页面|调用API wx.navigationTo 或使用组件\<navigatior open-type="navigateTo"/\>| onHide| onLoad,onShow|
页面重定向|调用API wx.redirectTo或使用组件\<navigatior open-type="redirectTo"/\>|onUnload|onLoad,onShow|
页面返回|调用API wx.navigationBack 或使用组件\<navigator open-type="navigateBack"> 或用户按左上角返回按钮|onUnload|onShow|
Tab切换|调用API wx.switchTab 或使用组件\<navigator open-type="switchTba" \/> 或用户切换Tab||各种情况|
重启动|调用API wx.reLaunch 或使用组件 \<navigator open-type="reLaunch" \/>| onUnLoad| onLoad,onShow|

Tab 切换对应的生命周期（以A,B页面为Tabbar 页面，C是从A页面打开的页面，D页面是从C页面打开的页面为例）：

当前页面|路由后的页面|触发的生命周期（按顺序）|
-----|-------|--------|-------|
A|A|Nothing happend|
A|B|A.onHide(),B.onLoad(),B.onShow()|
A|B(再次打开)|A.onHide(),B.onShow()|
C|A|C.onUnLoad(),A.onShow()|
C|B|C.onUnload(),B.onLoad(),B.onShow()|
D|B|D.onUnload(),C.onUnload(),B.onLoad(),B.onShow()|
D(从转发进入)|A|D.onUnload(),A.onLoad(),A.onShow()|
D(从转发进入)|B|D.onUnload(),B.onLoad(),B.onShow()|

> Tips:

+ navigateTo, redirectTo 只能打开非tabBar页面
+ switchTab 只能打开tabBar页面
+ reLaunch 可以打开任意页面
+ 页面底部的tabBar有页面决定，及只要定义为tabBar的页面，底部都有tabBar
+ 调用页面路由参数可以在目标页面的onload中获取。


### 文件作用域

在Javascript文件中声明的变量和函数只在该文件中有效；不同的文件中可以声明相同名字的变量和函数，不会互相影响

通过全局函数getApp()可以获取全局的应用实例，如果需要全局的数据可以在App（）中设置，如：

```
App({
	globalData: 1
})
```


```
a.js
const localValue = "a"
const app = getApp()
// 
app.globalData++
```

### 模块化

可以将一些公共的代码抽离成为一个单独的js文件，作为一个模块，模块只有通过module.exports 或者exports才能对外暴露接口；

需要注意的是：

+ exports 是module.exports 的一个引用，因此在模块里边随意更改exports的指向会造成未知的错误，所以更推荐开发者采用module.exports 来暴露模块接口，除非你已经清晰知道两者的关系。
+ 小程序目前不支持直接引入node\_modules ,开发者需要使用到node\_modules 时候建议拷贝出相关的代码到小程序的目录中或者使用小程序支持的[](npm)功能。

```
function sayHello(name){
	console.log('hello $(name)')
}

function sayGoodBye(name){
	console.log("GoodBye $(name)");
}

module.exports.sayHello = sayHello
exports.sayGoodBye = sayGoodBye
```

在需要使用这些模块的文件中，使用require(path)将公共代码引入

```
const common = require('common.js')
```

> > require 暂时不支持绝对路径
> > 
> > 



### API

小程序开发框架提供丰富的微信原生API,可以方便的调起微信提供的能力，如获取用户信息，本地存储，支付功能等，详细介绍请参考[](API文档)

### 事件监听API

我们约定，以on开头的API用来监听某个事件是否触发，如wx.onSocketOpen,wx.onCompassChange 等；

这类API接受一个回调函数作为参数，当事件触发时调用这个回调函数，并将相关数据以参数形式传入。

示例代码：

```
wx.onCompassChange(function(res){
	console.log(res.direction)
})
```

#### 同步 API

我们约定，以Sync 结尾的API都是同步API，如：wx.setStorageSync, wx.getSystemInfoSync 等，此外，也有一些其他的同步API，如：wx.crwateWorker,wx.getBackgroundAudioManager 等，详情参见API文档中的说明。

代码示例：

```

try{
	wx.setStorageSync("key","value")
}catch(e){
	console.error(e)
}

```


#### 异步 API

大多数API都是异步API，如 wx.request, wx.login 等，这类API接口通常都接受一个Object类型的参数，这个参数都支持按需指定以下字段来接收接口调用结果。

参数名|类型|必填|说明|
------|---|----|---|
success|function|否|接口调用成功的回调函数|
fail|function|否|接口调用失败的回调函数|
complete|function|否|接口调用结束的回调函数（成功、失败都会执行）|
其他|Any||接口定义的其他参数|


# 视图层

## WXML

WXML (weixin Markup Language) 是框架设计的一套标签语言，结合基础组件，事件系统，可以构建出页面的结构。

WXML文件的能力：数据绑定、列表渲染、条件渲染、模板、事件、引用


#### 数据绑定使用Mustache语法（双大括号）将变量包起来，可以作用于：

##### 内容

```
<view>{{message}}</view>
```

##### 组件属性

```
<view id="item-{{id}}"></view>
```


#### 控制属性（需要在双引号之内）

```
<view wx:if="{{condition}}"></view>
```

#### 关键字(需要在双引号之内)

> true : boolean 类型的true,代表真值
> false : boolean 类型的false，代表假值

```
	<checkbox checked="{{false}}"></checkbox>
```

> 特别注意：不要直接写 checked = "false"; 其计算结果是一个字符串，转换成boolean类型后代表真值
> 

#### 运算

> 可以在{{}}内进行简单的运算，支持的有以下几种方式：

```
<view hidden="{{flag ? true : false}}">Hideen</view>
``` 

> 算数运算

```
	<view>{{a+b }} + {{c}} + d</view>
```


> 逻辑判断

```
<view wx:if="{{length > 5}}"></view>
```

> 字符串运算

```
<view>{{"hello" + name}}</view>
```

> 数据路径运算

```
	<view>{{object.key}}{{array[0]}}</view>
```

> 组合 	也可以在mustache内直接进行组合，构成新的对象或者数组

> > 

```
<view wx:for="{{[zero , 1 ,2 ,3, 4]}}">{{item}}</view>	
	Page({
		data : {
			zero : 0
		}
})
	
最终生成数组 [0,1,2,3,4];

```

> 对象

```
<template is="objectCombine" data="{{for : a , bar : b}}"></template>

Page({
	data: {
		a : 1 ,
		b : 2
	}
})

最终组合的对象是 {for : 1 , bar : 2}

也可以用扩展运算符... 来将一个对象展开

<template is="objectCombine" data="{{...obj1 , obj2 , e : 5}}"></template>


Page({
	data : {
		obj1 : {
			a : 1,
			b : 2
		},
		obj2 : {
			c : 3,
			d : 4
		}
	}
})

最终组合生成的对象是：{a : 1 , b: 2 , c :3 , d: 4, e:5}

如果对象的key和value相同，也可以间接表达：

<template id="objectCombine" data="{{foo , bar}}"></template>

Page({
	data : {
		foo : 'my-foo',
		bar : 'my-bar'
	}
})

最终合成的对象是： {foo : 'my-foo' , bar : 'my-bar'}

```

+ 注意上述方式可以任意组合，但是如有存在变量名相同的情况，后边的会覆盖前面，如：

```
<template is="objectCombine" data="{{...obj1 , ...obj2 m a m c :6 }}"></template>

Page({
	data : {
		obj1 : {
			a : 1,
			b : 2
		},
		
		obj2 : {
			b : 3,
			c : 4
		},
		c : 5
	}
})

最终组合生成的对象是 {a : 5 , b :3 , c :6}

注意： 花括号和引号之间如果有空格，将最终被解析成字符串


```

## 列表渲染

### wx:for

在组件上使用wx:for 控制属性绑定一个数组，即可使用数组中各项的数据重复渲染该组件。

默认数组的当前项的下标变量名为index，数组当前项的变量名默认为item

```
	<view wx:for="{{array}}">
		{{index}}:{{item.message}}
	</view>
	
	Page({
		data : {
		 	array : [{
		 		message : 'foo',
		 	},{
		 		message : "bar"
		 	}
		 	]
		}
	})
	
	使用 wx:for-item 可以指定数组当前元素的变量名
	使用 wx:for-index 可以指定数组当前下标的变量名
```

```
	<view wx:for="{{array}}" wx:for-index="idx" wx:for-item="itemName">
		{{idx}} : {{itemName.message}}
	</view>
```

### block wx:for

类似block wx:if, 也可以将wx:for 用在<block /> 标签上。以渲染一个包含多个节点的结构块,





#### wx:key

如果列表中项目的位置会动态改变或者有新的项目添加到列表中，并且希望列表中的数据保持自己的特征和状态（如：<input> 中的输入内容，<switch> 的选中状态），需要使用wx:key 来指定列表中项目的唯一标识符；

wx:key 的值以两种形式提供

> 字符串，代表在for循环的array中item的某个property,该property的值需要是列表中唯一的字符串或数字，且不能动态改变；
> 保留关键字*this，代表在for循环中的item本身，这种表示需要item本身是一个唯一的字符串或者数字；
> 

注意：

+ 当wx:for 的值为字符串时，会将字符串解析成字符串数组；
+ 花括号和引号之间如果有空格，将最终被解析成为字符串；

### 条件渲染

#### wx:if 

在框架中，使用wx:if="{{condition}}" 来判断是否需要渲染该代码块；

```
	<view wx:if="{{condition}}">True</view>
``` 

#### block wx:if

因为wx:if 是一个控制属性，需要将它添加到一个标签上，如果要一次性判断多个组件标签，可以使用一个<block/>标签将多个组件包装起来，并在上边使用wx:if控制属性。

+ 注意： <block /> 并不是一个组件，它仅仅是一个包装元素，不会再页面中做任何渲染，只接受控制属性。


#### wx:if VS hidden

因为 wx:if 之间的模板也可能包含数据绑定，所以当wx:if的条件值切换时，框架有一个局部渲染的过程，因为它会确保条件块在切换时销毁或重新渲染。

同时wx:if 也是惰性的，如果在初始渲染条件为false，框架什么也不做，在条件第一次变成真的时候才开始局部渲染

相比之下，hidden就简单的多，组件始终都会被渲染，只是简单的控制显示与隐藏。
一般来说，wx:if 有更高的切换消耗而hidden有更高的初始渲染消耗，因此，如果需要频繁切换的情景下，用hidden更高，如果在运行时条件不大可能改变则wx:if 较好




## 模板

WXML 提供模板(template)，可以在模板中定义代码片段，
然后在不同的地方调用；定义模板：

使用name属性，作为模板的名字，然后在\<template \/>内定义代码片段，如：

```

<tempalte name="msgItem">
	<view>
		<text>{{index}} : {{msg}}</text>
	</view>
</template>

```

使用模板：

使用is属性，声明需要使用的模板，然后将模板所需要的data传入，如：

```
	<tempalte is="msgItem" data="{{...item}}"></template>
```


is属性可以使用Mustache语法，来动态决定需要渲染哪个模板

+ 模板的作用域

模板有自己的作用域，只能使用data传入的数据以及模板定义文件中定义的<wxs /> 模块


## 事件

#### 什么是事件

+ 事件是视图层到逻辑层的通讯方式
+ 事件可以将用户的行为反馈到逻辑层进行处理
+ 事件可以绑定在组件上，当达到触发事件，就会执行逻辑层中对应的事件处理函数
+ 事件对象可以携带额外的信息，如 id，dataset，touches

#### 事件的使用方式

+ 在组件中绑定一个事件处理函数
+ 在相应的page定义中写上相应的事件处理函数，参数是event

### 事件详解

#### 事件分类:

事件分为冒泡事件和非冒泡事件:

+ 1. 冒泡事件：当一个组件上的事件被触发后，该事件会向父节点传递
+ 2. 非冒泡事件：当一个组件上的事件被触发后，该事件不会向父节点传递

类型|触发条件|最低版本|
----|-----|------|
touchstart|手指触摸动作开始||
touchmove|手指触摸后移动||
touchcancle|手指触摸动作被打断，如来电提醒，弹窗|
touchend|手指触摸动作结束|
tap| 手指触摸后马上离开|
longpress|手指触摸后，超过350ms再离开，如果指定了事件回调函数并触发了这个事件，tap事件将不被触发
longtap|手指触摸后，超过350ms再离开（推荐使用longpress）
transitionend|会在WXSS transition 或 wx.createAnimation动画结束后触发|
animationstart|会在一个WXSS animation动画开始时触发|
animationiteration|会在一个WXSS animation 一次迭代结束时触发|
animationend|会在一个WXSS animation动画完成时触发|
touchforcechange|在支持3D Touch的iPhone设备，重按时会触发|

注：除上表之外的其他组件自定义事件如无特殊说明都是非冒泡事件，如<form/>的submit事件，


## 引用

+ WXML 提供两种文件引用方式 import 和 include

### import 

import 可以在该文件中使用目标文件定义的template,如：

##### import 的作用域

+ import 有作用域的概念，即只会import目标文件中定义的template,而不会import目标文件import的template


+ include include可以将目标文件除了\<template \/ > <wxs \/> 外的整个代码引入，相当于拷贝到include位置，如：

```
	<include src="header.wxml">	
	</include>
```

## WXSS

WXSS （wenxin Style Sheets）是一套样式语言，用来描述WXML的组件样式

WXSS 用来决定WXML的组件应该怎么显示

为了适应广大的前端开发者，WXSS具有CSS大部分特性，同时为了更适合开发微信小程序，WXSS对CSS进行了扩充以及修改

+ 尺寸单位
+ 样式导入

#### 尺寸单位

+ rpx(responsive pixe): 可以根据屏幕宽度进行自适应，规定屏幕宽为750rpx,如在iPhone6上，屏幕宽度为375rpx,共有750个物理像素，则750rpx==375px==750物理像素，则1rpx=0.5px=1物理像素


#### 样式导入

使用\@import 语句可以导入外联样式表，@import后需要导入的外联样式表的相对路径，用；表示语句结束

#### 全局样式和局部样式

定义在app.wxss 中的样式为全局样式，作用于每一个页面，在page的wxss文件中定义的样式为局部样式，只作用于对应的页面，并会覆盖app.wxss中相同的选择器

### 基础组件






## 自定义组件

类似于页面，自定义组件拥有自己的wxml模板和wxss样式

### 组件模板

组件模板的写法与页面模板相同，组件模板与组件数据结合后生成的节点树，将被插入到组件的引用位置上，

在组件模板中可以提供一个\<slot\>节点，用于承载组件引用时提供的子节点

注意：在模板中引用到的自定义组件以及对应的节点名需要在json文件中显式定义，否则会被当做有个无意义的节点，除此之外，节点名也可以被声明为抽象节点


#### 模板数据绑定： 

与普通的WXML模板类似,可以使用数据绑定，这样就可以向子组件的属性传递动态数据

代码示例：

```
<view>
	<component-tag-name prop-a="{{dataFiledA}}">
		<!--这部分内容将被放置在组件<slot>的位置上-->
		<view>这里是插入到组件slot中的内容</view>
	</component>
</view>

在以上的例子中，组件属性propA 和 propB 将收到页面传递的数据，页面可以通过setData来改变绑定的数据字段。
```

+ 注意： 这样的数据绑定只能传递JSON兼容数据，自基础库版本2.0.9开始，还可以在数据中包含函数（但这些函数不能再WXML中直接调用，只能传递给子组件。）

#### 组件wxml的slot

在组件的wxml中可以包含slot节点，用于承载组件使用者提供的wxml结构。

默认情况下，一个组件wxml中只能有一个slot，需要使用多slot时，可以在组件js中声明启用。

```
Component({
	options : {
		multipleSlots : true // 在组件定义时的选项中启用slot支持
	},
	
	properties : {<!---->}
	methods :  {<!---->}
})
```

#### 组件样式

组件对应wxss文件的样式，只对组件wxml内的节点生效，编写组件样式时，需要注意以下几点：

+ 组件和引用组件的页面不能使用id选择器（#a）,属性选择器（[a]）和标签选择器，请改用class选择器。
+ 组件和引用组件的页面中使用后代选择器（.a .b）在一些极端情况下会有非预期的比表现。
+ 子元素选择器（.a>.b）只能用于view组件与其子节点之间，用于其他组件可能导致非预期的情况。
+ 继承样式，如font、color,会从组件外继承到组件内
+ 除继承样式外，app.wxss中的样式、组件所在页面的样式对自定义组件无效

除此之外，组件可以指定它所在节点的默认样式，使用：host选择器


#### 外部样式类

有时，组件希望接受外部传入的样式表，此时可以在Component中用externalClasses定义段定义若干外部样式表，这个特性从小程序基础库版本1.9.90开始鼓支持

这个特性可以用于实现类似于view组件的hover-class属性，页面可以提供一个样式类，赋予view的hover-class，这个样式类本身写在页面中而非view组件的实现中。

注意：在一个节点上使用普通样式和外部样式时，两个类的优先级是未定义的，因此最好避免这种情况。

代码示例：

```
 Component({
 	externalClasses : ['my-class']
 })
 
 <!--组件custom-component.wxml-->
 <custom-component class="my-class">
 	这段文本的颜色有组件外的class决定
 </custom-component>
```

这样，组件的使用者可以指定这个样式类对应的class，就像使用普通属性一样。

#### 使组件接受全局样式

默认情况下，自定义组件的样式只受到自定义组件wxss的影响，除非一下两种情况：

	+ app.wxss 或页面wxss中使用了标签名选择器（或一些其他特殊选择器）来直接指定样式，这些选择器会影响到页面和全部组件，通常情况下这是不推荐的做法。

	+ 在特定的自定义组件激活了addGlobalClass选项，这使得这个组件能被app.wxss或页面的wxss中的所有的样式定义影响。

要激活addGlobalClass 选项，只需要在Component构造器中将 options.addGlobalClass 字段设置为true，这个特性从小程序基础库版本2.2.3开始支持.

+ 注意：当激活了addGlobalClass选项后，存在外部样式污染组件样式的风险，请谨慎选择


## Component

#### 定义段与示例方法

Component 构造器可用于定义组件，调用Component构造器可以指定组件的属性、数据、方法等；

定义段|类型|是否必填|描述|最低版本|
------|----|-----|-----|------|
properties| ObjectMap|否|组件的对外属性，是属性名到属性设置的映射表，属性设置中可包含三个字段，type表示属性类型，value表示属性初始值、observer表示属性值被更改时的响应函数||
data|Object|否|组件的内部数据，和property一样可以用于组件的模板渲染||
methods|Object|否|组件的方法，包括事件响应函数和任意的自定义方法，关于事件响应函数的使用，详见[](组件事件)||
behaviors|String、Array|否|类似于mixins和traits的组件代码复用机制，参见[](behavious)|
created|Function|否|组件的生命周期函数，在组件实例刚刚被创建时执行，注意此时不能调用setData，参见[](组件生命周期)|
attached|Function|否|组件的生命周期函数，在组件实例进入页面节点树时执行，参见[](组件生命周期)|
ready|Function|否|组件生命周期函数，在组件布局完成后执行，参见[](组件生命周期)|
moved|Function|否|组件生命周期函数，在组件实例被移动到节点树另一个位置时执行，参见[](组件生命周期)|
detached|Function|否|组件生命周期函数，在组件实例被从页面节点树移除时执行，参见[](组件生命周期)|
relations|Object|否|组件间关系定义，参见[](组件间关系)|
externalClasses|String、Array|否|组件接受的外部样式类,参见[](外部样式类)|
options|Object、Map|否|一些选项（文档中介绍相关特性时会涉及具体的选项设置，这里暂不列举）|
lifetimes|Object|否|组件生命周期声明对象，详见[](组件生命周期)|
pageLifttimes|Object|否|组件所在页面的生命周期函数生声明对象，支持页面的show、hide等生命周期,参见[](组件生命周期)|
definitionFilter|Function|否|定义段过滤器，用于[](自定义组件扩展)|

生成的组件实例可以在组件的方法、生命周期函数和属性observer中通过this访问，组件中包含一些通用属性和方法。

属性|类型|描述|
----|-----|---|
is|Strig|组件的文件路径|
id| String| 节点id|
dataset|String|节点dataset|
data|Object|组件数据，包括内部数据和属性值|
properties|Object|组件数据，包括内部数据和属性值（与data一致）|

方法名|参数|描述|
----|----|----|
setData|Object|设置data并执行视图层渲染|
hasBehavior|Object|检查组件是否具有behaviors（检查时会递归检查被直接或间接引入的所有behaviors）|
triggerEvent|String,Object|触发事件，参见[](组件事件)|
groupSetData| Function |立刻执行callBack，其中的多个setData之间不会触发界面绘制（只有某些特殊场景中需要，如用于不同组件同时setData时进行界面绘制同步）|

## 组件间通信与事件

组件间的基本通信方式有以下几种：

+ WXML 数据绑定： 用于父组件向子组件的指定属性设置数据，仅能设置JSON兼容数据。

+ 事件：用于子组件向父组件传递数据，可以传递任意数据
 
+ 如果以上两种方式不足以满足需要，父组件还可以this.selectComponent方法获取子组件实例对象，这样就可以直接访问组件的任意数据个方法。

### 监听事件

事件系统是组件间通信的主要方式之一，自定义组件可以触发任意的事件，引用组件的页面可以监听这些事件，关于事件的基本概念和用法，参见[](事件)

代码示例：

```
<!--当自定义组件触发“myevnet事件时，调用onMyEvent”方法-->
<component-tag-name bindmyevnet="onMyEvent" />

<!--或者可以写成-->
<component-tag-name bind:myevent="onMyEvent" />

```

### 触发事件

自定义组件触发事件时，需要使用triggerEvent方法，指定事件名、detail对象和事件选项：

```

<!--在自定义组件中-->
<button bindtap="onTap">点击这个按钮将触发“myevent”事件</button>

```


```

	Component({
		
		properties : {},
		
		methods: {
			onTap(){
				const myEventDetail = {} // detail对象，提供给事件监听函数
				const myEventOption = {} // 触发事件的选项
				
				this.triggerEvent("myevent", myEventDetail , myEventOption)
			}
		}
	})

```

触发事件的选项包括：

选项名| 类型| 是否必填| 默认值| 描述|
------|-----|-----|----|------|
bubbles| Boolean| 否| false| 事件是否冒泡|
composed | Boolean| 否| false| 事件是否可以穿越组件边界，为false时，事件将只能在引用组件的节点树上触发，不进入其它任何组件内部|
capturePhase| Boolean| 否 | false| 事件是否拥有捕获阶段|


关于冒泡和捕获阶段的概念，

代码示例：

```
<!--页面 page.wxml-->

<another-component bindcustomeevent="pageEventLister1">
	<my-component bindcustomeevent="pageEventListener2"></my-component>
</another-component>


<!--组件 another-component.wxml-->
<view bindcustomeevent="anotherEventlistener">
	<slot />
</view>

<!--组件 my-component.wxml-->

<view bindcustomeevent="myEventlistener">
	<slot />
</view>


// 组件 my-component.js

Component({
	methods : {
		onTap(){
			this.triggerEvent("customevent" , {}) // 只触发 pageEventListener2
			
			this.triggerEvent('customevent', {} , {bubbles : true}) 
			// 会依次触发pageEventlistener2 、 pageEventlistener1
			
			this.triggerEvent('customevent',{},{bubbles : true , composed : true}) // 会依次触发 pageEventListener2 、 anotherEventlistener、pageEventLister1
		}
	}
})

```


## 组件的生命周期

#### 组件的主要生命周期

组件的生命周期，指的是组件自身的一些函数，这些函数在特殊的时间点遇到一些特殊的框架事件时会自动触发。其中，最重要的生命周期函数是created、attached、detached，包含一个组件实例生命流程的最主要时间点。


+ 组件实例刚刚被创建好时，created生命周期函数被触发，此时，组件数据this.data 就是在Component 构造器中定义的数据data，此时 还不能调用setData，通常情况下，这个生命周期只应该用于组件this添加一些自定义属性字段。


+ 在组件完全初始化完毕，进入页面节点树后，attached生命周期被触发，此时，this.data 已经初始化为组件的当前值，这个生命周期很有用，绝大多数初始化工作可以在这个时机进行

+ 在组件离开页面节点树后，detached生命周期被触发，退出一个页面时，如果组件还在页面节点树中，则detached会被触发。


     
#### 定义生命周期方法

生命周期方法可以直接定义在Component构造器的第一级参数中。

自小程序基础版本2.3.3起，组件的生命周期也可以在lifetimes 字段内进行声明（这是推荐的方式，其优先级最高）

在behaviors中也可以编写生命周期方法，同时不会与其他behaviors中的同名生命周期函数互相覆盖，但要注意，如果一个组件多次直接或间接引用同一个behavior，这个behavior中的生命周期函数在一个执行时机只会执行一次。

生命周期| 参数| 描述| 最低版本|
------|-----|-----|-----|
created| 无| 在组件实例刚刚被创建时执行|1.6.3|
attached|无|在组件实例进入页面节点树时执行|1.6.3|
ready|无|在组件在视图层布局完成后执行|1.6.3|
moved|无|在组件实例被移动到节点树另一个位置时执行|1.6.3|
detached|无|在组件实例被从页面节点树移除时执行|1.6.3|
error|Object Error | 每当组件方法抛出错误时执行|

#### 组件所在页面的生命周期

还有一些特殊的生命周期，它们并非与组件有很强的关联，但有时组件需要获知，以便组件内部处理，这样的生命周期称为“组件所在页面的生命周期”，在pageLifetimes 定义段中定义，其中可用的生命周期包括：

生命周期|参数|描述|最低版本|
------|----|----|------|
show| 无| 组件所在页面被展示时执行|2.2.3|
hide| 无| 组件所在页面被隐藏时执行| 2.2.3|
resize| Object Size| 组件所在页面


## behaviors

#### 定义和使用behaviors

behaviors 是用于组件间代码共享的特性，类似于一些编程语言中的“mixins”或“traits”

每个behavior可以包含一组属性、数据、生命周期函数和方法，组件引用它时，它的属性、数据和方法会被合并到组件中，生命周期函数也会在对应时机被调用。每个组件可以引用多个behavior。behavior也可以引用其他behavior。

behavior需要使用Behavior()构造器定义

代码示例：

```
 module.exports = Behavior({
 	
 	behaviors : [],
 	properties : {
 	
 		myBehaviorProperty : {
 			type : String
 		}
 	},
 	data : {
 		myBehaviorData : {}
 	}
 	attached(){},
 	methods : {
 		myBehaviorMethod(){}
 	}
 })
组件引用时，在behavior定义段中将它们逐个列出即可
```

代码示例：

```
const myBehavior = require('my-behavior')

Component({
	
	behaviors : [myNehavior],
	
	properties : {
	
		myProperty : {
			type : String
		}
	},
	
	data : {
		myData : {}
	},
	
	attached(){},
	methods : {
		myMethod(){}
	}
})

```

> 在上例中，my-Component 组件定义中加入了my-behavior,而my-behavior中包含myBehaviorProperty属性，myBehaviorData数据字段，myBehaviorMethod方法和一个attached生命周期函数，这将使得my-component中最终包含myBehaviorProperty、myProperty两个属性，myBehaviorData、myData两个数据字段，和myBehaviorMethod、myMethod两个方法，当组件触发attached生命周期时，会依次触发my-behavior中的attached生命周期函数和my-component中的attached生命周期函数。
> 
> 

#### 字段的覆盖和组合规则

组件和它引用的behavior中可以包含同名的字段，对这些字段的处理方法如下：

+ 如果有同名的属性和方法，组件本身的属性和方法会覆盖behavior中的属性或方法，如果引用了多个behavior，在定义段中靠后behavior中的属性或方法会覆盖靠前的属性和方法。

+ 如果有同名的数据字段，如果数据是对象类型，会进行对象合并，如果是非对象则会进行相互覆盖；

+ 生命周期函数不会相互覆盖，而是在对应触发时机逐个调用，如果同一个behavior被一个组件多次引用，它定义的生命周期函数只会被执行一次。

#### 内置behaviors

自定义组件可以通过内置的behavior来获得内置组件的一些行为；

代码示例：

```
Component({
	behaviors : ['wx://form-field']
})

在上例中，wx://form-field 代表一个内置behavior，它使得这个自定义组件有类似于表单控件的行为。

内置behavior往往会会组件添加一些属性，在没有特殊说明时，组件可以覆盖这些属性来改变它的type或添加observer；

```

##### wx://form-field

使自定义组件有类似于表单控件的行为，form组件可以识别这些自定义组件，并在submit事件中返回组件的字段名及其对应字段值，这将为它添加以下两个属性：


属性名|类型|描述|最低版本|
-----|---|-----|------|
name|String|在表单中的字段名|1.6.7|
value|任意|在表单中的字段值|1.6.7|

##### wx://component-export

使自定义组件支持export 定义段，这个定义段可以用于指定组件被selectComponent调用时的返回值。

未使用这个定义段时，selectConponent 将返回的自定义组件的this（插件的自定义组件将返回null）。使用这个定义段时，将以这个定义段的函数返回值代替。

代码示例：

```
// 自定义组件 my-component 内部

Component({
	
	behaviors : [wx://component-export],
	export(){
		return {myField : 'myvalue'}
	}
})


<!--使用自定义组件时-->
<my-component id="the-id" />

this.selectComponeny('#the-id')
```

## 组件间关系

#### 定义和使用组件间关系

有时需要实现这样的组件：

```
<custom-ul>
	<custom-li>item 1</custom-li>
	<custom-li>item 2</custom-li>
</custom-ul>

```

这个例子中，custom-ul 和 custom-li 都是自定义组件，他们有相互间的关系，相互间的通信往往比较复杂，此时在组件定义时加入relations定义段，可以解决这样的问题，示例：

```

// path/to/custom-li.js
	Component({
		relations : {
			'./custom-li' : {
				type : 'child', // 关联的目标节点应为子节点
			linked(target){
				// 每次有custom-li 被插入时执行，target是该节点实例对象，触发在该节点attached生命周期之后
			}，
			linkChanged(target){
			// 每次custom-li被移动后执行，target是该节点实例对象，触发在该节点moved生命周期之后
			},
			unlinked(target){
			// 每次custom-li 被移除时执行，target是该节点实例对象，触发在该节点detached生命周期之后
			
			}
				
			}
		}，
		
		methods:{
		
		_getAllli(){
			// 使用getRelationNodes可以获得nodes数组，包含所有已关联的custom-li,且是有序的
			const nodes = this.getRelationNodes('path/to/custom-li')
		}
		
		},
		
		ready(){
			this._getAllli()
		}
	})

```

```
// path/to/custom-ul.js

Component({
	relations: {
		'./custom-ul' : {
			type : 'parent', // 关联的目标节点为父节点
			linked(target){
				// 每次被插入到custom-ul时执行，target是custom-ul节点实例对象，触发在attached生命周期之后 
			},
			
			linkChanged(target){
			 // 每次被移动后执行，target是custom-ul 节点实例对象，触发在moved生命周期之后
			},
			
			unlinked(target){
			// 每次被移除时执行，target是custom-ul节点实例对象，触发在detached生命周期之后
			}
		}
	}
})

```

###注意 ： 必须在两个组件定义中都加入relations定义，否则不会生效###

#### 关联一类组件：

有时，需要关联的是一类组件，如：


```
<custom-form>
	<view>
		input
		<custom-input></custom-input>
	</view>
	
	<custom-submit>submit</custom-submit>
</custom-form>

```

custom-form 组件想要关联custom-input 和 custom-submit 两个组件，此时，如果这两个组件都有同一个behavior：

```
module.exports = Behavior({
		// ...
})

// path/to/custom-input.js
const customFormControls = require('./custom-form-controls')

Component({
	behaviors : [customFormControls],
	
	relations : {
	
		'./custom-form' : {
			type : 'ancestor', // 关联的目标节点应为祖先节点
		}
	}
}) 

// path/to/custom-submit.js

const customFormControls = require('./custom-form-controls')

Component({
	behaviors : [customFormControls],
	
	relations : {
		'./custom-form' : {	
			type : 'ancestor', //关联的目标节点为祖先节点
		}
	}
})

则在relations 关系定义中，可使用这个behavior来代替组件路径作为关联的目标节点：

// path/to/custom-form.js

const customFormControls = require('./custom-form-controls')


Component({

	relations : {
	
		customFormControls : {
			type : 'descendant' , // 关联的目标节点为子孙节点
			target : customFormControls
		}
	}
})

```

#### relations 定义段

选项|类型|是否必填|描述|
-----|---|------|----|
type|String|是|目标组件的相对关系，可选的值为parent、child、ancestor、descendant|
linked|Function|否|关系生命周期函数，当关系被建立在页面节点树中时被触发，触发时机在组件attached生命周期之后|
linkChanged|Function|否|关系生命周期函数，当关系在页面节点中发生改变时触发，触发时机在组件moved生命周期之后|
unlinked|Function|否|关系生命周期函数，当关系脱离页面节点时触发，触发时机在组件detached生命周期之后|
target|String|否|如果这一项被设置，则它表示关联的目标节点所应具有的behavior,所有拥有这一behavior的组件节点都会被关联|



## 抽象节点

这个特性自小程序基础库版本1.9.6开始支持

#### 在组件中使用抽象节点

有时，自定义组件模板中的一些节点，其对应的自定义组件不是自定义组件本身确定的，而是自定义组件的调用者确定的，这时可以把这个节点声明为“抽象节点”。

例如，我们现在来实现一个“选框组”组件，它其中可以放置单选框(custom-radio)或者复选框（custom-checkbox）。这个组件的wxml可以这样编写：

```
	<view wx:for="{{labels}}">
		<label>
			<selectable disabled="{{false}}"></selectable>
			{{item}}
		</label>
	</view>
```
其中，“selectable” 不是在任何在json文件的usingComponents字段中声明的组件，而是一个抽象节点，它需要在componentGenerics字段中声明：

```
{
	"componentGenerics" : {
		"selectable" : true
	}
}
```

#### 使用包含抽象节点的组件

在使用selectable-group 组件时，必须指定“selectable”具体是哪个组件：

```
 	<selectable-group generic:selectable="custom-radio"></selectable-group>
```

这样在生成这个 selectable-group 组件的实例时，“selectable”节点会生成“custom-radio”组件实例，类似的，如果这样使用：

```
	<selectable-group generic:selectable="custom-checkbox" />
```

"selectable" 节点则会生成“custom-checkbox”组件实例。

####注意：上述的custom-radio 和 custom-checkbox 需要包含在这个wxml对应json文件的usingComponents定义段中####

```
	{
		"usingComponents" : {
			"custom-radio" : "path/to/custom/radio",
			"custom-checkbox" : "path/to/custom/checkbox"
		}
	}

```

#### 抽象节点的默认组件

抽象节点可以指定一个默认组件，当具体组件未被指定时，将创建默认组件的实例，默认组件可以在componentGenerics字段中指定：

```
{	
	"componentGerics" : {
		"selectable" : {
			"default" : "path/to/default/component"
		}
	}
}
```

###Tips### 

+ 节点的generic 引用generic:xxx = "yyy" 中，值yyy只能是静态值，不能包含数据绑定，因而抽象节点特性并不适用于动态决定决定节点名的场景;


## 自定义组件扩展

为了更好定制自定义组件的功能，可以使用自定义组件扩展机制，从小程序基础库版本2.2.3开始支持

#### 扩展后的效果

为了更好的理解扩展后的效果，先举一个例子：

```
	module.exports = Behavior({
		definitionFilter(defFields){
			defFields.data.from = "behavior"
		},
	})
	
	
	Component({
	
		data : {
			from : 'component'
		},
		behaviors : [require('behavior.js')],
		ready(){
			console.log(this.data.from) // 此处会发现输出behavior而不是component
		}
	})
```

通过这个例子可以发现，自定义组件的扩展其实就是提供了修改自定义组件定义段的能力，上述例子就是修改了自定义组件中的data定义段中的内容。

#### 使用扩展

Behavior() 构造器提供了新的定义段definitionFilter, 用于支持自定义组件扩展，definiitionFilter是一个函数，在调用的时候会注入两个参数，第一个参数是使用该behavior的component/behavior的定义对象，第二个参数是该behavior所使用的behavior的definitionFilter函数列表。

以下举个例子来说明：

```
	<!--behavior3.js-->
	module.exports = Behavior({
		definitionFilter(deefFields , definitionFilterArr){}
	})
	
	<!--behavior2.js-->
	module.exports = Behavior({
		behviors : [require('behavior3.js')],
		definitionFilter(defField ,definitionFilterArr){}
	})
	
	<!--behavior1.js-->
	module.exports = Behavior({
		behaviors : [require('behavior2.js')]
	})
	
	<!--component.js-->
	Component（{
		behaviors : [require('behavior1.js')]
	}）
```

上述代码中声明了一个自定义组件和3个behavior，每个behavior都使用了definitionFilter定义段，那么按照声明的顺序会有如下的事情发生：

> 1. 当进行behavior2的声明时就会调用behavior3的definitionFilter函数，其中defields参数是behavior2的定义段，definitionFielterArr 参数即为空数组，因为behavior3没有使用其他的bahavior。
> 
> 2. 当进行behavior1的声明时就会调用behavior2的definitionFilter函数，其中defFields参数是behavior1的定于段。definitionFilterArr参数是一个长度为1的数组，definitionFilterArr[0]即为behavior3的definitionFilter函数，因为behavior2使用了behavior3，用户在此处可以自行决定在进行behavior1的声明时要不要调用behavior3的definitionFilter函数，如果需要调用，在此处补充代码definitionFilterArr[0]（defields）即可，definitionFilterArr参数会由基础库补充传入。
> 
> 3. 同理，在进行component的声明时就会调用behavior1的definitionFilter函数
> 

###### 简单来说，definitionFiter 函数可以理解为当A使用了B时，A声明就会调用B的definitionFilter函数并传入A的定义对象让B去过滤，此时如果B还使用了C和D，那么B可以自行决定要不要调用C和D的definitionFilter函数去过滤A的定义对象。

## 开发第三方自定义组件

小程序从基础库版本2.2.1 开始支持使用npm安装第三方包，因此也开始支持和使用第三方自定义组件包，关于npm功能的详情可先阅读[](相关文档)


## 网络

在小程序、小游戏中使用网络相关的API时，需要注意下列问题，

> 1. 服务器域名配置

> >
每个微信小程序需要事先设置一个通讯域名，小程序只可以跟指定的域名进行网络通信。包括普通HTTPS请求（wx.request）、上传文件（wx.uploadFile）、下载文件（wx.downloadFile）和WebSocket(wx.connectSocket)


配置流程：

服务器域名请在【小程序后台-开发设置-服务器域名】中进行配置，配置时需要注意：

+ 域名只支持https(wx.request、wx.uploadFile、wx.downloadFile)和wss(wx.connectSocket)协议;

+ 域名不能使用IP地址或者localhost;

+ 可以设置端口，如https://myserver.com:8080,但是配置之后只能向https://myserver.com:8080发起请求，如果向https://myserver.com、https://myserver.com:9091 等URL请求则会失败。

+ 如果不设置端口，如https://myserver.com:8080,那么请求的URL中不能包含端口，设置是默认的443端口也不行，如果向https://myserver.com:443请求则会失败。

+ 域名必须经过ICP备案

+ 出于安全考虑，api.weixin.qq.com 不能被设置为服务器域名，相关API也不能在小程序内调用，开发者应将AppSecret保存到后台服务器中，通过服务器使用getAccessToken接口获取access\_token, 并调用相关API；

+ 对于每个接口，分别可以配置最多20个接口。


## 网络请求


##### 超时时间

+ 默认超时时间和最大超时时间都是60s;

+ 超时时间可以在app.json 中配置;


##### 使用限制

+ 网络请求的referer header不可设置，其格式固定为https://servicewechat.com/{appid}/{version}/page-frame.html，其中{appid}为小程序的appid,{version}为小程序的版本号，版本号为0表示为开发版、体验版以及审核版本。版本号为devtools表示为开发者工具，其余为正式版本；

+ wx.request、wx.uploadFile、wx.downloadFile的最大并发限制为10个；

+ 小程序进入后台运行后（非置顶聊天），如果5s内网络请求没有结束，会回调错误信息fail interrupted; 在回到前台之前，网络请求接口调用都会无法调用；

##### 返回值编码

+ 建议服务器返回使用UTF-8编码，对于非UTF-8编码，小程序会尝试进行转换，但是会有转换失败的可能；

+ 小程序会自动对BOM头进行过滤（只过滤一个BOM头）

## 存储

每个小程序都可以有自己的本地缓存，可以通过wx.setStorage/wx.setStorageSync、wx.getStorage/wx.getStorageSync、wx.clearStorage/wx.clearStorageSync、wx.removeStorage/wx.removeStorageSync 对本地缓存进行读写和清理;

同一个微信用户，同一个小程序storage上限为10MB。storgae以用户维度隔离，同一设备上，A用户无法读取到B用户的数据。

#### 注意：如果用户存储空间不足，我们会清空最近最久未使用的小程序本地缓存。我们不建议将关键信息全部存储在storage，以防存储空间不足或用户换设备的情况。

## 文件系统

#### 文件系统是小程序提供的一套以小程序和用户维度隔离的存储以及一套相应的管理接口，通过wx.getFileSystemManager()可以获取到全局唯一的文件系统管理器，所有文件系统的管理操作通过FileSystemManager来调用；

```
	const fs = wx.getFileSystemManager();
```

文件主要分为两大类：

+ 代码包文件：代码包文件指的是在项目中添加的文件。

+ 本地文件：通过调用接口本地产生，或通过网络下载下来，存储到本地的文件。

其中本地文件又分为三种：

+ 本地临时文件：临时产生，随时会被回收的文件，不限制存储大小。
 
+ 本地缓存文件：小程序通过接口把本地临时文件缓存后产生的文件。不能自定义目录和文件名，除非用户主动删除小程序，否则不会被删除，跟本地用户文件共计，普通小程序最多可存储10MB，游戏类目的小程序最多可存储50MB.

+ 本地用户文件：小程序通过接口把本地临时文件缓存后产生的文件，允许自定义目录和文件名。除非用户主动删除小程序，否则不会被删除，跟本地缓存文件共计，普通小程序最多可存储10MB，游戏类的小程序最多可存储50MB；




#### 本地文件

本地文件指的是小程序被用户添加到手机后，会有一块独立的文件存储区域，以用户维度隔离，即同一台手机，每个温馨用户不能访问到其他登录用户的文件，同一用户不同appid之间的文件也不能相互访问。

本地文件的文件路径均以下格式：

{{协议名}}://文件路劲

> 其中，协议名在iOS、Android客户端为“wxfile”,在开发者工具上为：“http”,开发者无需关注这个差异，也不应该在代码中去硬编码完整文件路径。
> 

##### 本地临时文件

本地临时文件只能通过调用特定接口产生，不能直接写入内容，本地临时文件产生后，仅在当前生命周期内有效，重启之后即不可用，因此，__不可把本地临时文件路径存储起来下次使用__，如果需要下次再使用，可通过FileSystemManager.saveFile()或FileSystemManager.copyFile()接口把本地转换成本地缓存文件或本地用户文件。


#####  本地缓存文件

本地缓存文件只能通过调用特定接口产生，不能直接写入内容，本地缓存文件产生后，重启之后仍可使用，本地缓存文件只能通过FileSystemManager.saveFile()接口将本地临时文件保存获得。

示例：

```
	fs.saveFile({
		
		tempFilePath : "", // 传入一个本地临时文件路径
		success(res){
			caonsole.log(res.savedFilePath)// res.saveFilePath 为一个本地缓存文件路径
		}
	})
	
```

#### 本地用户文件

本地用户文件是从1.7.0 版本新增的概念，我们提供了一个用户文件目录给开发者，开发者对这个目录有完全自由的读写权限，通过__wx.env.USER_DATA_PATH__ 可以获取到这个目录的路径。

示例：

```
	const fs = wx.getFileSystemManage()
	fs.writeFileSync('${wx.env.USER_DATA_PATH}/hello.text','hello world' , 'utf-8')

```


## Canvas 画布

所有在\<canvas\> 中的画图必须使用Javascript中完成；

WXML： （我们在接下来的例子中如无特殊声明都会用这个WXML为模板，不在重复）

```
<canvas canvas-id="myCanvas" style="border:1px solid;"></canvas>
	
JS: (我们在接下来的例子中会将JS放在onLoad中)
	
const ctx = wx.createCanvasContext('myCanvas')
ctx.setFillStyle('red')
ctx.fillRect(10,10,150,75)
ctx.draw()
```
##### 第一步：创建一个Canvas绘图上下文#####

首先，我们需要创建一个canvas绘图上下文CanvasContext;


 CanvasContext 是小程序内建的一个对象，有一些绘图的方法：
 
 ```
 	const ctx = wx.createCanvasContext('myCanvas')
 ```
 
 
##### 第二步：利用Canvas绘图上下文进行绘图描述#####

接着，我们来描述要在Canvas中绘制什么内容。

设置绘图上下文的填充为红色:

```
	ctx.setFillStyle('red')
```

用fillRect(x,y,width,height) 方法画一个矩形，填充为刚刚设设置的红色：

```
	ctx.fillRect(10,10,150,75)
```

##### 第三步：画图#####

告诉\<canvas\> 组件你要讲刚刚的描述绘制上去:

```
	ctx.draw()
```

## 分包加载

在某些情况下，开发者需要将小程序划分为不同的子包，在构建时打包成不同的子包，用户在使用时按需进行加载。

在构建小程序分包项目时，构建会输出一个或多个分包，每个使用分包小程序必定包含一个主包，所谓的主包，即放置默认启动页面、TabBar页面以及一些所有分包需要用到公共资源、js脚本，而分包则是根据开发者的配置进行划分。

在小程序启动时，默认会下载主包并启动主包内页面，当用户进入分包某个页面时，客户端会把对应分包下载下来，下载完成后再进行展示：

目前小程序分包大小有如下限制：

+ 整个小程序的所有分包大小不超过8M
+ 单个分包、主包大小不能超过2M


## 开放能力

#### 小程序登录

小程序可以通过微信官方提供的登录能力方便地获取微信提供的用户身份标识，快速建立小程序内的用户体系；

![](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/image/api-login.jpg?t=19012322)

说明：

+ 1. 调用wx.login() 获取__临时登录凭证code__,并传回到开发者服务器；
+ 2. 调用code2Session接口，换取 __用户唯一标识 OpenID__ 和 __回话密钥session\_key__。

之后开发者服务器可以根据用户标识来生成自定义登录态，用于后续业务逻辑中前后端交互时识别用户身份。

__注意：__

+ 1. 回话密钥session_key 是对用户数据加密签名的密钥，为了自身的数据安全，开发者服务器不应该被回话密钥下发小程序，也不应该对外提供这个密钥。
+ 2. 临时登录凭证code只能用一次。










































