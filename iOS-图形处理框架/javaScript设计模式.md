##### 什么是设计模式？
--

在软件工程中，设计模式是软件设计中常见问题的可重用的解决方案，设计模式代表着经验丰富的软件开发人员的最佳实践，设计模式被认定是编程模板。

##### 为什么使用设计模式？

--

使用适当的设计模式可以帮助你编写更好，更易理解的代码，并且代码可以轻松维护，因为它们更容易理解。

最重要的是，设计模式为软件开发人员提供了一些沟通上的便利。它们会立即向学习你代码的人显示你的代码意图。

例如：如果你在项目中使用装饰者模式，那么新程序员将立即知道该代码正在做什么，并且他们可以更专注于解决业务问题，而无需发费精力去理解你的代码正在做什么。

###### 1.模块模式
--

模块是一段独立的代码，因此我们可以在不影响其他代码的情况下单独更新模块，模块还允许我们为变量创建单独作用域来避免命名空间的污染，当它们与其他代码段分离时，我们也可以在其他项目中重用模块。

模块是任何现代Javascript应用程序中不可或缺的一部分，有助于保持代码清洁、分离和组织。有许多方法可以在Javascript中创建模块，其中一种是模块模式。

与其他编程语言不同，JavaScript没有访问修饰符的特性，也就是说，你不能将变量声明为私有（private）或公开（public），因此模块模式也常常用于模块封装的概念。


此模式使用IIFE（立即调用的函数表达式），闭包和函数作用域来模拟此概念，例如：

```
const myModule = (function() {
  
  const privateVariable = 'Hello World';
  
  function privateMethod() {
    console.log(privateVariable);
  }
 
  return {
    publicMethod: function() {
      privateMethod();
    }
  }
 
})();
 
myModule.publicMethod();

```

由于上面的代码是IIFE，代码会立即执行，返回的对象被分配到myModule变量，由于闭包，即使在IIFE完成之后，返回的对象仍然可以访问IIFE内定义的函数和变量。

因此，在IIFE中定义的变量和函数对外部作用域来说基本上是隐藏的，因此他们对myModule变量是私有的。

执行完代码后，myModule变量如下所示：

```
const myModule = {
  publicMethod: function() {
    privateMethod();
  }
};
```

因此，我们可以在调用publicMethod(),转而调用privateMethod(),例如：

```
// Prints 'Hello World'
module.publicMethod();
```

###### 暴露模块模式

暴露模块模式是对模块模式略微的改进版本，模块模式的问题是我们必须创建新的公开函数来调用私有函数和变量，

在暴露模块模式中，我们将返回的对象的属性映射到我们想要公开的私有函数，这就是为什么它被称为暴露模块模式的原因；

```
const myRevealingModule = (function() {
  
  let privateVar = 'Peter';
  const publicVar  = 'Hello World';
 
  function privateFunction() {
    console.log('Name: '+ privateVar);
  }
  
  function publicSetName(name) {
    privateVar = name;
  }
 
  function publicGetName() {
    privateFunction();
  }
 
  /** reveal methods and variables by assigning them to object     properties */
 
return {
    setName: publicSetName,
    greeting: publicVar,
    getName: publicGetName
  };
})();
 
myRevealingModule.setName('Mark');

// prints Name: Mark
myRevealingModule.getName();
```

这种模式使我们更容易理解我们可以公开访问那些函数和变量，这有助于代码的可读性。

代码执行后，myRevealingModule 如下所示：

```
const myRevealingModule = {
  setName: publicSetName,
  greeting: publicVar,
  getName: publicGetName
};

```

我们可以调用 myRevealingModule.setName('Mark')，来引用内部的 publicSetName ，以及调用myRevealingModule.getName() ，来引用内部的 publicGetName 。例如：

```
myRevealingModule.setName('Mark');
 
// prints Name: Mark
myRevealingModule.getName();
```

#### 暴露模块模式相较于模块模式的优点：


+ 我们可以修改 return 语句中的一行代码，来将成员从 public(公开) 更改为 private(私有) ，反之亦然。


+ 返回的对象不包含函数定义，所有右侧表达式都在 IIFE 中定义，使代码清晰易读。


##### ES6模块 （ES6 Modules）
--

在ES6之前，JavaScript 没有内置的模块系统，所以开发人员必须依赖第三方库或模块模式来实现模块化。但是在 ES6 中，JavaScript 拥有了原生的模块系统。

ES6 模块存储在单独的文件中。每个文件只能有一个模块。默认情况下，模块中的所有内容都是私有的。函数、变量和类使用 export 关键字来向外公开。模块内的代码总是在 严格模式(strict mode) 下运行。


##### 导出模块

--

导出函数和变量声明的两种方法：

+ 1）通过在函数和变量声明前添加export关键字，例如：

```
// utils.js
export const greeting = 'Hello World';
 
export function sum(num1, num2) {
  console.log('Sum:', num1, num2);
  return num1 + num2;
}
 
export function subtract(num1, num2) {
  console.log('Subtract:', num1, num2);
  return num1 - num2;
}
 
// This is a private function
 
function privateLog() {
  console.log('Private Function');
}
```

+ 2) 通过在代码末尾添加 export 关键字，并包含我们要导出的函数和变量的名称。例如：

```
// utils.js
function multiply(num1, num2) {
  console.log('Multiply:', num1, num2);
  return num1 * num2;
}
function divide(num1, num2) {
  console.log('Divide:', num1, num2);
  return num1 / num2;
}
// This is a private function
function privateLog() {
  console.log('Private Function');
}
export {multiply, divide};
```

##### 导入模块

与导出模块类似，有两种方法可以使用import关键字导入模块：例如

+ 1）一次导入多个项目

```
// main.js
 
// importing multiple items
import { sum, multiply } from './utils.js';
```

+ 2) 导入所有模块

```
// main.js
 
// importing all of module
import * as utils from './utils.js';
 
```

##### 导入、导出模块可以使用别名

如果要避免命名冲突，可以在导出和导入时使用别名，例如：

+ 1）重命名导出

```
// utils.js
 
function sum(num1, num2) {
  console.log('Sum:', num1, num2);
  return num1 + num2;
}
 
function multiply(num1, num2) {
  console.log('Multiply:', num1, num2);
  return num1 * num2;
}
 
export {sum as add, multiply};
```

+ 2) 重命名导入

```
// main.js
 
import { add, multiply as mult } from './utils.js';
```

#### 单例模式（singleton Pattern）
--

Singleton（单例）是一个只能实例化一次的对象，如果不存在，则单例模式会创建类的实例，如果存在实例，则它只返回对该对象的引用。 对构造函数的任何重复调用总是会获取相同的对象。

JavaScript 一直支持单例模式。 我们只是不称他们为单例，我们称之为 对象字面量。 例如：

```
const user = {
  name: 'Peter',
  age: 25,
  job: 'Teacher',
  greet: function() {
    console.log('Hello!');
  }
};
```

因为 JavaScript 中的每个对象占用一个唯一的内存位置，当我们调用 user 对象时，我们实际上是返回对该对象的引用。

如果我们尝试将用户变量复制到另一个变量并修改该变量。 例如：

```
const user1 = user;
user1.name = 'Mark';
```

可以使用构造函数实现单例模式。例如：

```

let instance = null;
 
function User() {
  if(instance) {
    return instance;
  }
 
  instance = this;
  this.name = 'Peter';
  this.age = 25;
  
  return instance;
}
 
const user1 = new User();
const user2 = new User();
 
// prints true
console.log(user1 === user2); 
```

调用此构造函数时，它会检查 instance 对象是否存在。 如果该对象不存在，则将该变量分配给 instance 变量。如果对象存在，它只返回该对象。

单例模式也可以使用模块模式实现。 例如：

```
const singleton = (function() {
  let instance;
  
  function init() {
    return {
      name: 'Peter',
      age: 24,
    };
  }
 
  return {
    getInstance: function() {
      if(!instance) {
        instance = init();
      }
      
      return instance;
    }
  }
})();
 
const instanceA = singleton.getInstance();
const instanceB = singleton.getInstance();
 
// prints true
console.log(instanceA === instanceB);

```

在上面的代码中，我们通过调用 singleton.getInstance 方法创建一个新实例。 如果实例已存在，则此方法仅返回该实例，如果实例不存在，则通过调用 init() 函数创建新实例。


##### 工厂模式（Factory Pattern）

工厂模式是一种使用工厂方法创建对象的设计模式，而不指定创建对象的确切的类或构造函数。

工厂模式用于在不公开实例化逻辑的情况下创建对象。当我们需要根据特定条件生成不同的对象时，可以使用此模式。例如：


```
class Car{
  constructor(options) {
    this.doors = options.doors || 4;
    this.state = options.state || 'brand new';
    this.color = options.color || 'white';
  }
}
 
class Truck {
  constructor(options) {
    this.doors = options.doors || 4;
    this.state = options.state || 'used';
    this.color = options.color || 'black';
  }
}
 
class VehicleFactory {
  createVehicle(options) {
    if(options.vehicleType === 'car') {
      return new Car(options);
    } else if(options.vehicleType === 'truck') {
      return new Truck(options);
      }
  }
}
```

在这里，我创建了一个 Car 和 Truck 类（带有一些默认值），用于创建新的 car 和 truck 对象。 我已经定义了一个 VehicleFactory 类，用于根据 options 对象中收到的 vehicleType 属性创建并返回一个新对象。
```
JavaScript 代码:
const factory = new VehicleFactory();
 
const car = factory.createVehicle({
  vehicleType: 'car',
  doors: 4,
  color: 'silver',
  state: 'Brand New'
});
 
const truck= factory.createVehicle({
  vehicleType: 'truck',
  doors: 2,
  color: 'white',
  state: 'used'
});
 
// Prints Car {doors: 4, state: "Brand New", color: "silver"}
console.log(car);
 
// Prints Truck {doors: 2, state: "used", color: "white"}
console.log(truck);
```



#### 装饰者模式（Decorator Pattern）

装饰者模式用于扩展对象的功能，而无需修改现有的类或构造函数。 此模式可用于向对象添加功能，而无需它们修改底层代码。

这种模式的一个简单例子是：

```
function Car(name) {
  this.name = name;
 
  // Default values
  this.color = 'White';
}
 
// Creating a new Object to decorate
const tesla= new Car('Tesla Model 3');
 
// Decorating the object with new functionality
 
tesla.setColor = function(color) {
  this.color = color;
}
 
tesla.setPrice = function(price) {
  this.price = price;
}
 
tesla.setColor('black');
tesla.setPrice(49000);
 
// prints black
console.log(tesla.color);
```

这种模式的一个更实际的例子是：

比方说，汽车的成本取决于它的功能数量。 如果没有装饰者模式，我们必须为不同的功能组合创建不同的类，每个类都有一个成本方法来计算成本。 例如：

```
class Car() {
}
 
class CarWithAC() {
}
 
class CarWithAutoTransmission {
}
 
class CarWithPowerLocks {
}
 
class CarWithACandPowerLocks {
}
```

但是使用装饰者模式，我们可以创建一个基类 ·Car`，并使用装饰者函数将不同配置的成本计算方法添加到其对象中。例如：


```
class Car {
  constructor() {
    // Default Cost
    this.cost = function() {
      return 20000;
    }
  }
}
 
// Decorator function
function carWithAC(car) {
  car.hasAC = true;
  const prevCost = car.cost();
  car.cost = function() {
    return prevCost + 500;
  }
}
 
// Decorator function
function carWithAutoTransmission(car) {
  car.hasAutoTransmission = true;
   const prevCost = car.cost();
  car.cost = function() {
    return prevCost + 2000;
  }
}
 
// Decorator function
function carWithPowerLocks(car) {
  car.hasPowerLocks = true;
  const prevCost = car.cost();
  car.cost = function() {
    return prevCost + 500;
  }
}
```

首先，我们创建一个基类 Car，用于创建 Car 对象。 然后，然后我们为了不同的功能创建了装饰者函数，并将 Car 对象作为参数传递。 然后我们覆盖该对象的成本函数，该函数返回汽车的更新成本，并向该对象添加新属性以指示添加了哪个特征。

要添加新功能，我们可以执行以下操作：

```
const car = new Car();
console.log(car.cost());
 
carWithAC(car);
carWithAutoTransmission(car);
carWithPowerLocks(car);
```

最后，我们可以像这样计算汽车的成本：

```
// Calculating total cost of the car
console.log(car.cost());
```

























