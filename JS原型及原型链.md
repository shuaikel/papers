+ 关于this

```
	var obj = {
		foo : function (){}
	};
	
	var foo = obj.foo;
	
// 写法一
	obj.foo();
// 写法二
	foo();
	
	上面代码中，虽然obj.foo和foo指向同一个函数，但是执行结果可能不一样，请看下面的例子：
	
	var obj = {
		foo : function() { console.log(this.bar)},
		bar : 1
	}
	
	var foo = obj.foo;
	var bar = 2;
	
	obj.foo(); // 1
	foo(); // 2
```

这种差异的原因，就在于函数内部使用了__this__关键字，很多教科书会告诉你，__this__指的是函数运行时所在的环境。对于__obj.foo()__来说，foo运行在__obj__环境，所以this指向obj；对于foo()来说，foo运行在全局环境，所以this指向全局环境，所以，两者的运行结果不一样。

这种解释没有错，但是教科书往往不告诉你，为什么会这样？也就是说，函数的运行环境到底是怎么决定的？举例来说，为什么obj.foo() 就是在obj环境执行，而一旦var foo = obj.foo,foo()就变成在全局环境中执行？


本文就来解释Javascript 这样处理的原理，跟内存里面的数据结构有关系。

```
	var obj = {foo : 5};
```

上面的代码将一个对象赋值给变量obj，JavaScript引擎会现在内存里面，生成一个对象__{foo : 5}__,然后把这个对象的内存地址赋值给变量obj。

![](https://www.wangbase.com/blogimg/asset/201806/bg2018061801.png)

也就是说，变量obj是一个地址__(reference)__。后面如果要读取obj.foo，引擎先从obj拿到内存地址，然后再从该地址读出原始的对象，返回它的foo属性。

原始的对象以字典结构保存，每一个属性都对应一个属性描述对象，举例来说，上面例子的foo属性，实际上是以下面的形式保存的

![](https://www.wangbase.com/blogimg/asset/201806/bg2018061802.png)

注意：foo属性的值保存在属性描述对象的value属性里面。

### 函数

___

这样的结构是很清晰的，问题在于属于属性的值可能是一个函数。

```
	var obj = {fooo : function(){}};
```

这时，引擎会将函数单独保存在内存中，然后将函数的地址赋值给__foo__属性的__value__属性。

![](https://www.wangbase.com/blogimg/asset/201806/bg2018061803.png)

```
    {
      foo: {
        [[value]]: 函数的地址
        ...
      }
    }
```

由于函数是一个单独的值，所以它可以在不同的环境（上下文）执行。

```
    var f = function () {};
    var obj = { f: f };

    // 单独执行
    f()

    // obj 环境执行
    obj.f()

```

### 环境变量

----

JavaScript允许在函数体内部，引用当前环境的其他变量。

```
	var f = function(){
		console.log(x);
	}
```

上面代码中，函数体里面使用了变量x，该变量由运行环境提供。

现在问题就来了，由于函数可以在不同的环境执行，所以需要有一种机制，能够在函数体内部获得当前的运行环境（context），所以，this就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。

```
	var f = function(){
		console.log(x);
	}
```

上面代码中，函数体里面的this.x就是指当前运行环境的x。

```
	var f = function(){
		console.log(this.x);
	}
	var x = 1; 
	var obj = {
		f : f,
		x : 2,
	},
	
// 单独执行
f(); // 1

// obj 环境执行
obj.f(); // 2
```

上面代码中，函数f在全局环境中执行，this.x指向当前环境的x。



![](https://www.wangbase.com/blogimg/asset/201806/bg2018061804.png)


在obj环境中执行，this.x 执向obj.x

![](https://www.wangbase.com/blogimg/asset/201806/bg2018061805.png)

原型与原型链是JS中不可避免要碰到的知识点，本文使用图片思维导图的形式捋一捋原型、原型链、实例、构造函数等概念之间的关系。

#### Constructor 构造函数

首先我们先写一个构造函数Person, 构造函数为了区别普通函数要求首字母大写：

```
	function Person(){}
```

#### prototype 原型
---

 原型是指一个对象，实例“”继承那个对象的属性，在原型上定义的属性，通过“继承”，实例也拥有这个属性，“继承”这个行为是在new操作符内部实现的。
 
 先不说实例，原型与构造函数的关系就是，构造函数内部有一个prototype的属性，通过这个属性就能访问到原型：
 
 ![](https://user-gold-cdn.xitu.io/2019/3/14/1697caabdd89c215?imageView2/0/w/1280/h/960/ignore-error/1)


Person 就是构造函数，Person.prototype 

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caabe391603b?imageView2/0/w/1280/h/960/ignore-error/1)

### instance 实例

有了构造函数，我们就可以在原型上创建可以“继承”的属性，并通过new 操作符创建实例

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caabe02c7068?imageView2/0/w/1280/h/960/ignore-error/1)

比方说Person, 我们要创建一个person实例，那么使用new操作符就可以实现，并通过instanceof来检查他们之间的关系：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caabe236a881?imageView2/0/w/1280/h/960/ignore-error/1)

我们在原型上定义一个属性，那么实例上也可以“继承”这个属性：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caabe3a805b9?imageView2/0/w/1280/h/960/ignore-error/1)


### proto 隐式原型：

实例通过__proto__访问到原型，所以如果是实例，那么就可以通过这个属性直接访问到原型：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac2aaef961?imageView2/0/w/1280/h/960/ignore-error/1)

所有两者是等价的：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac2ab3f410?imageView2/0/w/1280/h/960/ignore-error/1)

### constructor 构造函数：

---

既然构造函数通过prototype来访问到原型，那么原型也应该可以通过某种途径访问到构造函数。这就是constructor：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac2a873506?imageView2/0/w/1280/h/960/ignore-error/1)

因此两者的关系应该是这样的：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac2a745a9e?imageView2/0/w/1280/h/960/ignore-error/1)


### 实例、构造函数、原型之间的关系

---

这里我们可以看到如果实例想要访问到构造函数，那么应该是：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac46d22f60?imageView2/0/w/1280/h/960/ignore-error/1)

没有从实例直接访问到构造函数的属性或者方法：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac56809356?imageView2/0/w/1280/h/960/ignore-error/1)

在读取一个实例的属性的过程中，如果属性在该实例中没有找到，那么就会循着__proto__指定的原型上去寻找，如果还找不到，则尝试寻找原型的原型：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac5b7c916e?imageView2/0/w/1280/h/960/ignore-error/1)

### 原型链

---

原型同样也可以通过\_\_proto\_\_访问到原型的原型，比如说这里有个构造函数Person然后“继承”前者的有一个构造函数People得到实例p

当访问到p中的一个非自有属性的时候，就会通过\_\_proto\_\_作为桥梁连接的一些列原型、原型的原型、原型的原型的原型知道Object构造函数为止。

这个搜索的过程形成的链状关系就是原型链：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac62041820?imageView2/0/w/1280/h/960/ignore-error/1)


原型链关系图：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac6f1e804c?imageView2/0/w/1280/h/960/ignore-error/1)



null，原型搜索到null为止，搜不到那访问这个属性就是不存在的：

![](https://user-gold-cdn.xitu.io/2019/3/14/1697caac84763e75?imageView2/0/w/1280/h/960/ignore-error/1)






























