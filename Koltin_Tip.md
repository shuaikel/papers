### 1、Koltin 类和对象

> Koltin类可以包含：构造函数和初始化代码块、函数、属性、内部类、对象声明。
> 
> Koltin中使用关键字class声明类,后面紧跟类名。
> 
> 

#### 类的属性
##### 属性定义
类的属性可以用关键字var声明为可变的，否则使用只读关键字val声明为不可变。

> Koltin 中的类可以有一个主构造器，以及一个或多个次构造器，主构造器是类头部的一部分，位于类名称之后：
> 
> class Person constructor（firstName：String）{}
> 
> 如果主构造器没有任何注解，也没有任何可见度修饰符，那么constructor关键字可以省略。
> 

###### 实例
> 以下实例定义了一个Person类，包含两个可变变量lastName和no，lastName修改了getter方法，no修改了setter方法

```
class Person {

    var lastName: String = "zhang"
        get() = field.toUpperCase()   // 将变量赋值后转换为大写
        set

    var no: Int = 100
        get() = field                // 后端变量
        set(value) {
            if (value < 10) {       // 如果传入的值小于 10 返回该值
                field = value
            } else {
                field = -1         // 如果传入的值大于等于 10 返回 -1
            }
        }

    var heiht: Float = 145.4f
        private set
}

// 测试
fun main(args: Array<String>) {
    var person: Person = Person()

    person.lastName = "wang"

    println("lastName:${person.lastName}")

    person.no = 9
    println("no:${person.no}")

    person.no = 20
    println("no:${person.no}")

}
```

Koltin中类不能有字段，提供了Backing Fields（后端变量）机制，备用字段使用field关键字声明，field关键字只能用于属性的访问器，

> 非空属性必须在定义的时候初始化，Koltin提供了一种可以延迟初始化的方案，使用lateinnit关键字描述属性：
> 

##### 主构造器
主构造器中不能包含任何代码，初始化代码可以放在初始化代码段中，初始化代码段使用init关键字作为前缀。

```
class Person constructor(firstName: String) {
    init {
        println("FirstName is $firstName")
    }
}
```
注意：主构造器的参数可以在初始化代码段中使用，也可以在类主体n定义的属性初始化代码中使用，一种简洁语法，可以通过主构造器来定义属性并初始化属性值（可以是var或者val）

#### 次构造函数
类也可以有二级构造函数，需要加前缀constructor

```
class Person { 
    constructor(parent: Person) {
        parent.children.add(this) 
    }
}
```
如果类有主构造函数，每个次构造函数都要，或直接或间接通过另外一个次构造函数代理主构造函数，在另一个类中代理另一个构造函数使用this关键字。

```
class Person(val name: String) {
    constructor (name: String, age:Int) : this(name) {
        // 初始化...
    }
}
```

如果一个非抽象类没有声明构造函数（主构造函数或次构造函数），它会产生一个没有参数的构造函数，构造函数是public。如果你不想你的类有公共的构造函数，你就要声明一个空的主构造函数：
```
class DontCreateMe private constructor () {
}
```
注意：在 JVM 虚拟机中，如果主构造函数的所有参数都有默认值，编译器会生成一个附加的无参的构造函数，这个构造函数会直接使用默认值。这使得 Kotlin 可以更简单的使用像 Jackson 或者 JPA 这样使用无参构造函数来创建类实例的库。
```
class Customer(val customerName: String = "")
```

###### 抽象类是面向对象编程的特征之一，类本身，或类中的部分成员，都可以声明为abstract的，抽象成员在类中不存在具体的实现，注意：无需对抽象类或抽象成员标注open注解；

```
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

###### 嵌套类
我们可以把类嵌套在其他类中，看以下实例：

```
class Outer {                  // 外部类
    private val bar: Int = 1
    class Nested {             // 嵌套类
        fun foo() = 2
    }
}

fun main(args: Array<String>) {
    val demo = Outer.Nested().foo() // 调用格式：外部类.嵌套类.嵌套类方法/属性
    println(demo)    // == 2
}
```

##### 内部类
内部类使用inner关键字来表示，内部类会带有一个外部类的对象的引用，所以内部类可以访问外部类成员属性和成员函数。

```
class Outer {
    private val bar: Int = 1
    var v = "成员属性"
    /**嵌套内部类**/
    inner class Inner {
        fun foo() = bar  // 访问外部类成员
        fun innerTest() {
            var o = this@Outer //获取外部类的成员变量
            println("内部类可以引用外部类的成员，例如：" + o.v)
        }
    }
}

fun main(args: Array<String>) {
    val demo = Outer().Inner().foo()
    println(demo) //   1
    val demo2 = Outer().Inner().innerTest()   
    println(demo2)   // 内部类可以引用外部类的成员，例如：成员属性
}
```

为了消除歧义，要访问来自外部作用域的 this，我们使用this@label，其中 @label 是一个 代指 this 来源的标签。


###### 匿名内部类
使用对象表达式来创建匿名内部类：

```
class Test {
    var v = "成员属性"

    fun setInterFace(test: TestInterFace) {
        test.test()
    }
}

/**
 * 定义接口
 */
interface TestInterFace {
    fun test()
}

fun main(args: Array<String>) {
    var test = Test()

    /**
     * 采用对象表达式来创建接口对象，即匿名内部类的实例。
     */
    test.setInterFace(object : TestInterFace {
        override fun test() {
            println("对象表达式创建匿名内部类的实例")
        }
    })
}
```



