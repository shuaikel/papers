### 通过Object构造函数或对象字面量创建单个对象

这些方式有明显的缺点：使用同一个接口创建很多对象，会产生大量的重复代码，为了解决这个问题，出现了工厂模式；


#### 1. 工厂模式创建对象

所谓__工厂模式__,就是创建一个工厂方法，将创建对象的过程用一个函数进行封装，即放到工厂方法中，并且在这个函数（工厂方法）内进行对象的初始化，最终返回这个对象，如：

```
	function createPerson(name ,age){
		var o = new Object(); // 创建一个空的对象
		/*以下是给这个空间的对象进行初始化**/
		o.name = name;
		o.age = age; 
		o.sayName = function(){
			alert(this.name);
		}
		return o;
	}
```

工厂模式的优点：每次创建对象的时候只需要一个工厂方法即可，其解决了批量创建多个相似对象代码复用性的问题。
工厂模式的缺点：__通过工厂模式创建的对象都是Object类型，所以存在对象识别的问题；__


#### 构造函数模式创建对象

所谓__构造函数模式__,就是通过声明一个构造函数，构造函数内部显示的创建对象，也不显示返回对象，而是通过this来进行对象的初始化，然后通过“new”+构造函数的方式来创建对象，如：

```
	function Person(name ,age){
		/**通过this来创建对象的初始化*/
		this.name = name;
		this.age = age;
		this.sayName = function(){
			alert(this.name);
		}
	}
```

构造函数模式的优点：__其解决了对象识别的问题，不同的构造函数创建的对象属于不同的对象的实例；__,

1. 没有显示创建对象；
2. 直接将属性和方法赋给了this对象；
3. 没有return 语句；
4. 要创建新实例，必须使用new操作符；（否则属性和方法将会被添加到window对象）
5. 可以使用instanceof操作符检测对象类型

构造函数模式的缺点：__对象中的方法代码每次创建对象都会重新创建一遍，这就造成了内存的浪费；__


#### 原型模式创建对象

所谓__原型模式__,就是定义一个空的构造函数，然后通过给构造函数的prototype原型对象添加属性和方法来创建对象；

```
	function Person(){
		// 定义一个空的构造函数
	}
	/**在原型对象上添加属性和方法**/
	Person.prototype.name = "even";
	Person.prototype.age = 27;
	Person.prototype.sayName = function(){
		alert(this.name);
	}
```

原型模式的优点：__因为对象的属性和方法都定义在构造函数的prototype原型对象上，而原型上的属性和方法为所有对象公用，只会创建一遍，__

原型模式的缺点：__因为对象的属性和方法都是共用的，所以省略了对象的初始化过程，所有对象的默认值都是一样的，并且如果原型的属性值是一个引用类型，那么某个对象对其进行更改，所有对象都会被影响。__

#### 构造函数和原型模式混合模式创建对象

所谓__混合模式__,就是在构造函数中定义对象的实例属性，在原型中定义方法和共享的属性；

```
	function Person(name ,age){
		/**在构造函数中定义实例属性**/
		this.name = name;
		this.age = age;
	}
	
	/**在原型中定义方法**/
	Person.prototype.sayName = function(){
		alert(this.name);
	}
```

混合模式的优点：__因为每个实例对象都拥有一份属于自己的实例属性，同时又共享着方法的引用，所以可以最大限度的节省内存，是创建对象的好方式。__

混合模式的缺点：__构造函数和原型的定义处于分离状态，没有封装在一起，违背了封装的特性；__

#### 动态原型模式创建对象

所谓__动态原型模式__,就是将原型的定义放到构造函数内从而实现封装。如果直接将原型的定义放到构造函数内，那么会产生一个弊端，就是每次创建对象，构造函数内的原型定义都会被初始化一次，所以需要对其进行判断，如果原型已经初始化，那么久不再进行初始化，如：

```
	function Person(name ,age){
		this.name = name;
		this.age = age;
		if(typeof this.sayName !== function){
			Person.prototype.sayName = function(){
				alert(this.name);
			}
		}
	}
```
当第一次创建对象的时候，this.sayName 并不存在是undefined，所以其不等于function，然回给Person的原型对象上添加一个sayName()方法，下次创建对象的时候就不会重新初始化了。这种方式非常完美。


#### 寄生构造模式创建对象

所谓__寄生构造函数模式__,就是定义一个构造函数，但是这个构造函数内并不使用this进行初始化，而是将创建对象的代码封装到这个构造函数内，里面的代码和工厂模式一模一样，其个工厂模式的唯一区别就是，创建对象的时候是使用“new“+构造函数的方式去创建对象，所以寄生构造函数模式其实就是工厂模式和构造函数模式的结合，就是将工厂模式中的代码放到构造函数内，然后像构造函数那样去使用，如：

```
	function Person(name , age){
		var o = new Object();
		o.name = name;
		o.age = age;
		o.sayName = function(){
			alert(this.name);
		}
	}
	return o;
```

寄生构造函数模式通常用于在不修改某个构造函数的情况下扩展该构造函数，间接给该构造函数添加新的功能，即专门为JS原生的构造函数添加新的方法，
如：我们想要一个具有额外功能的数组，让这个数组可以直接调用某个方法就能输出一个用特定字符号连接的字符串，因此我们不能直接去修改Array构造函数，所以我们只能通过寄生构造函数的方式来实现，如：

```
	function SpecialArray(){
		var arr = new Array();
		arr.push.apply(arr , arguments);
		arr.toPipedString = function(){
			return this.join("|");
		}
		return arr; // 最后返回这个有特殊功能的对象
	}
```

var specicalArray = new SpecicalArray("red","blue","green");
alert(specicalArray.toPipedString());

寄生构造函数模式的缺点：就是使用计算构造函数模式创建的对象和构造函数之间没有什么关系，因此instaanceof操作符对这种对象没什么意义。

#### 稳妥构造模式创建对象

所谓__稳妥构造函数模式__,就是没有公共属性，而且方法也不引用this，创建对象的时候也不使用new，而是像工厂方法一样使用，所以稳妥构造函数模式适用于不能使用this和new的安全环境中使用，如：

```
	function Person(name ,age){
		var o = new Object();
		/**这里可以定义一些私有的变量**/
		o.sayName = function(){
			alert(name); // 这里不用this
		}
		return o;
	}
	
	var frinend = Person("even" , 27);
	alert(friend.name); // 无法获取到name属性
	frinend.sayName(); // Even
```

稳妥构造函数模式的优点：通过稳妥构造函数创建的对象其是一个稳妥对象，其属性只能通过对外暴露的方法来访问，直接通过点属性的方式是无法访问到的，可以保证对象中原始数据的安全性；

稳妥构造函数模式的缺点：使用稳妥构造函数模式创建的对象和构造函数之间没有什么关系，因此instanceof操作符对这种对象没有什么意义；

















