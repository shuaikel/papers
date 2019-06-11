#### 1、什么是JSX?

---

一个JSX语法的示例，如下所示

```
 const element = <h1>Hello, world!</h1>;
```

这种语法形式，既不是HTML，也不是字符串，而是称之为JSX，是React里用来描述UI和样式的语法，JSX最终会被编译为合法的JS语句调用（编译器在遇到{时采用JS语法进行解析，遇到<就采用HTML规则进行解析）

#### 2、嵌入表达式

---
JSX中，可以使用花括号{}嵌入任意的JavaScript合法表达式，如：2 + 2、user.firstName、
formatName(user)都是合法的。示例如：

```
	const user = {
    firstName: 'Zhang',
    lastName : 'Busong'
};

const elem = (
    <h1>Hello, {formatName(user)}</h1>
);
/*
这里的()，实际上是可选的，但是React推荐加入()，这样子就会被视为一个表达式，而不会导致
自动插入分号的问题
*/

ReactDOM.render(
    element,
    document.getElementById('app')
)

```


#### 3、JSX也是一种表达式

---
JSX本身也是一种表达式，所以它可以像其他表达式一样，用于给一个变量赋值、作为函数实参、作为函数返回值，等等。如：

```
function getGreeting(user) {
    if (user) {
        return <h1>Hello, {formatName(user)}</h1>
    }
    return <h1>Hello, Guest!</h1>;
}
```

**注意：** 1、在JSX中，声明属性时不要使用引号，如果声明属性的时候使用引号，那么将被作为字符串解析，而不会被作为一个表达式解析，如：

```
	<div firstName="{user.firstName}" lastName={user.lastName}></div>
```

解析后，可以得到：

```
	<div firstName="{user.firstName}" lastName="Lau"></div>
```

因此，

1、当我们需要使用一个字符串字面量的时候，可以使用引号，但是如果要作为表达式解析的时候，则不应当使用引号

2、在JSX中，有些属性名称需要进行特殊处理。如class应该用className代替，tabindex则用tabIndex代替。这是因为JSX本质上更接近于JavaScript，而class是JavaScript中的保留字。同时，应该使用camelCase来命名一个属性，而不是使用HTML的属性命名方式

3、JSX本身已经做了防注入处理，对于那些不是明确编写的HTML代码，是不会被解析为HTML DOM的，ReactDOM会将他们一律视为字符串，在渲染完成前就转化为字符串，所以可以防止XSS攻击

4、如果JSX标签是闭合的，那么结尾需要用/>，另外，JSX标签是可以互相嵌套的，这和HTML里是一样的




#### 4、 JSX实质

SX通过babel编译，而babel实际上把JSX编译给React.createElement()调用。如下JSX代码：

```
	const element = (
    <h1 className="greeting">
        Hello, world!
    </h1>
);
```

是等同于以下语句的：

```
	const elem = React.createElement(
    'h1',
    {className: 'greeting'},
    'Hello, world!'
);
```
React.createElement()方法会首先进行一些避免BUG的检查，然后返回类似以下例子的对象：

```
	const element = {
    type: 'h1',
    props: {
        className: 'greeting',
        children: 'Hello, world'
    }
}
```

这样的对象，则称为**React元素**，代表所有呈现在屏幕上的东西。React正是通过读取这些对象来构建DOM，并且保持数据和UI同步的。


#### 5、元素渲染


元素（elements）是构成React应用的最小单元，元素描述了想要在屏幕中看到的内容，如：

```
const element = <h1>Hello, world</h1>;
```

和DOM元素不同的是，React元素是纯对象，创建的代价低。并且React会进行优化处理，只把有必要的变化更新到DOM上。此外，元素和组件的概念，是不一样的，组件是由元素组成的。


#### 6、将元素渲染进DOM

---

在React中，使用ReactDOM.render()方法来将React元素渲染进一个DOM中。如：

```
	ReactDOM.render(
    element,
    document.getElementById('root')
)
```

React元素是不可变的，所以一旦一个元素创建完成后，我们是无法改变其内容或者属性的。一个元素就像是动画里的一帧，它代表UI在某一时间点的样子。如果非要使用元素来构成可变化的UI界面，就需要使用**setInterval**了，如：


```
	function tick() {
    const element = (
        <div>Now is {new Date().toLocaleTimeString()}</div>
    );
    ReactDOM.render(
        element,
        document.getElementById('root')
    );
}
setInterval(tick, 1000);

```

在实际开发中，大多数React应用只会调用一次ReactDOM.render()，所以更好的方式是使用**有状态组件**


#### 7、组件和Props

---


组件（**component**）能够将UI划分成独立的、可复用的部分，这样我们就只需专注于构建每一个单独的部件，从概念上看，组件就想函数：接受任意的输入（称为属性，Props）,返回React元素，react中有两种定义组件的方式：函数定义和类定义。


##### 1.函数定义组件

这种方式是最简单的定义组件的方式，就像写一个JS函数一样，如：

```
	function Welcome (props) {
    return <h1>Hello, {props.name}</h1>;;
}
```

##### 2.类定义组件

还可以使用ES6里的类来定义一个组件，如下所示：

```
	class Welcome extends React.Component {
    render () {
        return <h1>Hello, {this.props.name}<h1>;
    }
}
```
 这种方式比起**函数定义**方式则更加灵活

##### 3.组件渲染

---

先前，我们遇到的React元素只是呈现一个DOM标签，如：

```
	const element = <div />
```

然而，React元素也可以是用户自定义的**组件**，如：


```
	const element = <Welcome name="Tom" />
```

Welcome组件中声明了一个属性name="Tom"，而这个属性，将以props.name的方式传递给组件，如下方式：

```
	function Welcome (props) {
    return <h1>Hello, {props.name}</h1>;
}
```

此时，对于以下的代码：

```
	ReactDOM.render(
    <Welcome name="张不怂" />,
    document.getElementById('root')
)
```

最终就会以**\<h1\>Hello, 张不怂\</h1\>**的方式呈现。在这个过程中，发生了如下的事情：

+ 对**<Welcome name="张不怂" \/>**元素调用了**ReactDOM.render()**丰富

+ React将**{ name: '张不怂' }**作为props实参来调用Welcome组件

+ Welcome完成渲染，返回*\<h1>Hello, 张不怂</h1>**元素


+ ReactDOM计算最小更新代价，然后更新DOM

##### 4.组合组件


组件是可以组合的。即组件内部可以引用其他组件，如：

```
	function Welcome (props) {
    return <h1>Hello, {props.name}</h1>;
}

function App () {
    return (
        <div>
            <Welcome name="Tom" />
            <Welcome name="Jack" />
            <Welcome name="Mike" />
        </div>
    )
}

ReactDOM.render(
    <App />,
    document.getElementById('root')
)

```

**注意：** 在React中，组件必须返回**单一**的根元素，这也是为什么App组件中需要用**<div>**标签包裹的原因。如以下的方式，是错误的（因为它有3个根元素）：

```
	function App () {
    return (
        <Welcome name="Tom" />
        <Welcome name="Jack" />
        <Welcome name="Mike" />
    )
}
```
##### 5.属性是只读的

考虑以下这种情况：

```
	function sum (a, b) {
    return a + b;
}
```

这种函数称为纯函数：它不改变自己的输入值，且总是对相同的输入返回相同的结果。 与之对立的，则是非纯函数，如：

```
function withdraw (account, amount) {
    account.total -= amount;
}
```

**非纯函数**在函数内改变了输入的参数。在React中，无论是通过**function**还是**class**声明组件，我们都不应该修改它自身的属性（**props**）。虽然React相当灵活，但是它也有一个严格的规定：**所有的React组件都必须像纯函数那样来使用它们的props。**


#### 8、state与生命周期

---

使用类定义组件有一些额外的好处，如拥有本地状态这一特性。 以下是一个类定义组件

```
	class Clock extends React.Component {
    render () {
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>Now is {this.props.date.toLocaleTimeString()}</h2>
            </div>
        );
    }
}
```

需要注意的有：

+ **类名** 即为 **组件名**（无论是函数定义组件还是类定义组件，组件名称的首字母都必须大写，并且继承自**React.Component**）；

+ 使用 **render()** 方法，用来返回需要呈现的内容


##### 1、在类中加入state

state是属于一个组件自身的。我们可以在类的构造函数**constructor**中来初始化状态，如：

```
	constructor (props) {
    super(props)
    this.state = {
        date: new Date()
    }
}
```

如此一来，我们就可以在**render()**函数中使用**this.state.xxx**来引用一个状态


##### 2、生命周期

在应用里，往往都会有许许多多的组件。在组件销毁后，回收和释放它们所占据的资源非常重要。
在时钟应用的例子里，我们需要在第一次渲染到DOM的时候设置一个定时器，并且需要在相应的DOM销毁后，
清除这个定时器。那么，这种情况下，React为我们提供了生命周期的钩子函数，方便我们进行使用。在
React中，生命周期分为：

+ (1）Mount 已插入真实DOM
+ (2）Update 正在重新渲染
+ (3）Unmount 已移出真实DOM

而相应的，生命周期钩子函数有：

+ componentWillMount

+ componentDidMount

+ componentWillUpdate(newProps, nextState)

+ componentDidUpdate(prevProps, prevState)

+ componentWillUnmount()

此外，还有两种特殊状态的处理函数：

+ **componentWillReceiveProps(nextProps)** 已加载的组件收到新的参数时调动

+ **shouldComponentUpdate(nextProps, nextState)** 组件判断是否重新渲染时调用



需要注意的是： 
(1）**render()**里用不到的state，不应该声明在**state**里 
(2）不能直接使用**this.state.xxx = xxx**的方式来改变一个**state**的值，应该使用**this.setState()**。如：

```
	setName () {
    this.setState({
        name: '张三'
    })
}
```

**this.setState()**会自动覆盖**this.state**里相应的属性，并触发**render()**重新渲染。

(3）**状态更新可能是异步的** React可以将多个setState()调用合并成一个调用来提升性能。且由于**this.props**和**this.state**可能是异步更新的，所以不应该依靠它们的值来计算下一个状态。这种情况下，可以给**setState**传入一个函数，如：

```
	this.setState((prevState, props) => ({
    counter: prevState.counter + props.increment
}));
```


#### 9、事件处理

---

React元素的事件与DOM元素类似，不过也有一些区别，如： 1）React事件使用**camelCase**命名（**onClick**），而不是全小写的形式（**onclick**） 2）使用JSX，传入的是事件的句柄，而不是一个字符串 如以下的HTML：

```
	<button onclick="increment()">ADD</button>
```

使用React的方式描述如：

```
	<button onClick={increment}>ADD</button>
```

还有一个不同在于，在原生DOM中，我们可以通过返回**false**来阻止默认行为，但是这在React中是行不通的，在React中需要明确使用**preventDefault()**来阻止默认行为。如：

```
function ActionLink () {
    function handleClick (e) {
        e.preventDefault();
        alert('Hello, world!');
    }

    return (
        <a href="#" onClick={handleClick}>Click Me</a>
    );
}
```
这里，事件回调函数里的**event**是经过React特殊处理过的（遵循W3C标准），所以我们可以放心地使用它，而不用担心跨浏览器的兼容性问题。
注意： 在使用事件回调函数的时候，我们需要特别注意**this**的指向问题，因为在React里，**除了构造函数和生命周期钩子函数里会自动绑定this为当前组件外，其他的都不会自动绑定this的指向为当前组件**，因此需要我们自己注意好this的绑定问题，通常而言，在一个类方式声明的组件里使用事件回调，我们需要**在组件的constructor里绑定回调方法的this指向**，如：

```
	class Counter extends React.Component {
    constructor (props) {
        super(props);
        this.state = {
            counter: 0
        }
        // 在这里绑定指向
        this.increment = this.increment.bind(this);
    }
    increment () {
        this.setState({
            counter: this.state.counter + 1
        });
    }
    render () {
        return (
            <div>
                The counter now is: {this.state.counter}
                <button onClick={this.increment}>+1</button>
            </div>
        );
    }
}

```


当然，我们还有另外一种方法来使用箭头函数绑定指向，就是使用**实验性**的属性初始化语法，如：

```
class Counter extends React.Component {
    increment: () => {
        this.setState({
            counter: this.state.counter + 1
        });
    }
    // ...
}

```

像事件处理程序传递参数 我们可以为事件处理程序传递额外的参数，方式有以下两种：

```
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>

```

需要注意的是，使用箭头函数的情况下，参数**e**要显式传递，而使用bind的情况下，则无需显式传递（参数**e**会作为最后一个参数传递给事件处理程序）


#### 10、条件渲染
---

在React里，我们可以创建不同的组件来封装我们需要的功能。我们也可以根据组件的状态，只渲染组件中的一部分内容，而条件渲染就是为此而准备的。在React中，我们可以像在JavaScript中写条件语句一样地写条件渲染语句，如：

```
function Greet(props) {
    const isLogined = props.isLogined;
    if (isLogined) {
        return <div>Hello !</div>;
    }
    return <div>Please sign in</div>;
}

ReactDOM.render(
    <Greet isLogined={true} />,
    document.getElementById('root')
);

```

这将渲染出：

```
	<div>Hello !</div>
```


##### 1、使用变量来存储元素

我们也可以使用变量来存储元素，如：

```

function LogBtn(props) {
    var button;
    const isLogined = props.isLogined;
    if (isLogined) {
        button = <button>退出</button>
    } else {
        button = <button>登陆</button>
    }
    return <div>You can {button}</div>;
}

ReactDOM.render(
    <LogBtn isLogined={false} />,
    document.getElementById('root')
);

```

##### 2、使用&&运算符来进行渲染

---


由于JavaScript语法对待&&运算符的性质，我们也可以使用&&运算符来完成条件渲染，如

```
function LogBtn(props) {
    var button;
    const isLogined = props.isLogined;
    return (
        <div>Hello
        {!isLogined && (
            <button>请登陆</button>
        )}
        </div>
    )
}
```

当**props.isLogined**为false的时候，就会渲染出：


```
<div>Hello <button>请登录</button></div>
```


##### 3、使用三目运算符进行渲染

我们可能已经发现了，其实JSX可以像一个表达式那样子灵活使用，所以，我们自然也可以使用三目运算符进行渲染，如：

```
function LogBtn (props) {
    const isLogined = props.isLogined;
    return (
        <div>You can 
            <button>{isLogined ? '退出' : '登陆'}</button>
        </div>
    )
}

```


##### 4、阻止整个组件的渲染

有时候，我们希望是整个组件都不渲染，而不仅仅是局部不渲染，那么这种情况下，我们就可以在render()函数里返回一个null，来实现我们想要的效果，如：

```
	function LogBtn (props) {
    const isLogined = props.isLogined;
    const isShow = props.isShow;
    if (isShow) {
        return (
            <div>You can 
                <button>{isLogined ? '退出' : '登陆'}</button>
            </div>
        )
    }
    return null;
}
```

注意：组件里返回**null**不会影响组件生命周期的触发，如**componentWillUpdate**和**componentDidUpdate**仍然会被调用


#### 11、列表渲染与keys

在JavaScript中，我们可以使用map()函数来对一个数组列表进行操作，如：

```
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(number => number*2);
console.log(doubled); // 得到[2, 4, 6, 8, 10]

```
同样的，在React里，我们也可以使用**map()**来进行列表渲染，如：

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map(number => {
    return (
        <li>{number}</li>
    )
});

ReactDOM.render(
    <ul>{listItems}</ul>,
    document.getElementById('root')
)

```
这将得到：

```
<ul><li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
</ul>
```

当然，我们还可以进行更好的封装，如：

```
function NumberList (props) {
    const numbers = props.numbers;
    const listItems = numbers.map(number => {
        return (
            <li>{number}</li>
        )
    });

    return <ul>{listItems}</ul>
}

```

当我们运行以上的代码的时候，会发现控制台提示：**Each child in an array or iterator should have a unique "key" prop，**因此，我们需要为列表项的每一个项分配一个**key**，来解决这个问题，通常而言，我们可以使用以下几种方式来提供**key**：

+ 使用数据项自身的ID，如**\<li key=\{item.itemId\}>**;

+ 使用索引下标（**index**），如：

```
const listItems = numbers.map((number, index) => {
    <li key={index}>{number}</li>
});
```

但是React不推荐在需要重新排序的列表里使用索引下标，因为会导致变得很慢。

**注意：** 只有在一个项的同胞里区分彼此的时候，才需要使用到key，key不需要全局唯一，只需要在一个数组内部区分彼此时唯一便可。key的作用是给React一个提示，而不会传递给组件。如果我们在组件内需要同样的一个值，可以换个名字传递，如：

```
const content = posts.map(post => (
    <Post key={post.id} id={post.id} title={post.title} />
));
```

#### 12、表单

---

表单和其他的React中的DOM元素有所不同，因为表单元素生来就是为了保存一些内部状态。在React中，表单和HTML中的表单略有不同

##### 1、受控组件

HTML中，**\<input\>、\<textarea\>、\<select\>**这类表单元素会维持自身状态，并根据用户输入进行更新。不过React中，可变的状态通常保存在组件的**this.state**中，且只能用**setState()**方法进行更新，如：

```

class NameForm extends React.Component {
    constructor (props) {
        super(props);
        this.state = {
            value: ''
        }
        this.handleChange = this.handleChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }
    handleChange (event) {
        this.setState({
            value: event.target.value
        });
    }
    handleSubmit (event) {
        alert('Your name is '+this.state.value);
        event.preventDefault();
    } 
    render () {
        return (
            <form onSubmit={this.handleSubmit}>
            Name: <input type="text" value={this.state.value} onChange={this.handleChange} />
            <input type="submit" value="Submit" />
            </form>
        )
    }
}
```

和HTML中不同的是，React中的**textarea**并不需要写成**\<textarea\>\</textarea\>**的形式，而是写成**\<textarea value="" ... />**的形式便可。而对于HTML中的select标签，通常做法是：


```
	<select>
    <option value="A">A</option>
    <option value="B" selected>B</option>
    <option value="C">C</option>
</select>

```

但是React中，不需要在需要选中的option处加入selected，而只需要传入一个value，就会自动根据value来选中相应的选项，如：

```
<select value="C">
    <option value="A">A</option>
    <option value="B">B</option>
    <option value="C">C</option>
</select>

```

那么如上述例子，C所在的这个**option**就会被选中

##### 2、多个输入的解决办法

通常一个表单都有多个输入，如果我们为每一个输入添加处理事件，那么将会非常繁琐。好的一个解决办法是，使用name，然后根据**event.target.name**来选择做什么。如：

```
	class Form extends React.Component {
    constructor (props) {
        super(props);
        this.state = {
            name: '',
            gender: '男',
            attend: false,
            profile: ''
        };
        this.handleInputChange = this.handleInputChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }
    handleInputChange (event) {
        const target = event.target;
        const value = target.type==='checkbox' ? target.checked : target.value;
        const name = target.name;
        this.setState({
            [name]: value
        });
    }
    handleSubmit (event) {
        this.setState({
            profile: `姓名：${this.state.name}，${this.state.gender}，${this.state.attend ? '参加' : '不参加'}活动`
        });
        event.preventDefault();
    } 
    render () {
        return (
            <form>
            <p>姓名：<input name="name" value={this.state.name} onChange={this.handleInputChange} /></p>
            <p>性别：
                <select name="gender" value={this.state.gender} onChange={this.handleInputChange}>
                    <option value="男">男</option>
                    <option value="女">女</option>
                </select>
            </p>
            <p>是否参加：<input name="attend" type="checkbox" onChange={this.handleInputChange} checked={this.state.attend} /></p>
            <input type="submit" value="Submit" onClick={this.handleSubmit} />
            <p>您的报名信息：{this.state.profile}</p>
            </form>
        )
    }
}

```

##### 3、非受控组件

---

大多数情况下，使用**受控组件**实现表单是首选，在受控组件中，表单数据是交由React组件处理的。如果想要让表单数据由DOM处理（即数据不保存在React的状态里，而是保存在DOM中），那么可以使用**非受控组件**，使用**非受控组件**，可以无需为每个状态更新编写事件处理程序，使用**ref**即可实现，如：

```
class NameForm extends React.Component {
    constrcutor(props) {
        super(props)
    }
    handleSubmit(event) {
        console.log('A name was submitted: ', this.input.value)
        event.preventDefault()
    }
    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <label>
                Name: <input type="text" ref={input => this.input = input} />
                </label>
                <input type="submit" value="submit" />
            </form>
        )
    }
}

```
对于**非受控组件**，如果要指定默认值，那么可以使用**defaultValue**，如：

```
<input type="text" defaultValue="Hello" ref={input => this.input = input} />

```
相应的，**type="checkbox"**和**type="radio"**，则使用**defaultChecked**


#### 13、状态提升
---

当需要几个组件共用状态数据的时候，可以使用状态提升技术。核心思想在于：把数据抽离到最近的共同父组件，父组件管理状态（state），然后通过属性（props）传递给子组件。如实现一个货币转换的组件，可以如下：

##### 1、首先定义转换函数

```
function USD2RMB (amount) {
    return amount * 6.7925;
}

function RMB2USD (amount) {
    return amount * 0.1472;
}

function convert (amount, typeFn) {
    return typeFn(amount);
}

```

##### 2、定义组件

---

我们希望在RMB的输入表单上上输入的时候，USD的输入表单上的数值也同步更新，这种情况下，如果RMB组件自己管理自己的状态，是很难以实现的，因此，我们需要让这个状态提升自父组件进行管理。如下：

```
class CurrencyInput extends React.Component {
    constructor (props) {
        super(props)
        this.handleChange = this.handleChange.bind(this)
    }
    handleChange (event) {
        this.props.onInputChange(event.target.value)
    }
    render () {
        const value = this.props.value
        const type = this.props.type
        return (
            <p>{type}: <input type="text" value={value} onChange={this.handleChange} /></p>
        );
    }
}

```

最后定义一个共同的父组件，如下：

```
class CurrencyConvert extends Component {
    constructor (props) {
        super(props);
        this.state = {
            type: 'RMB',
            amount: 0
        }
        this.handleRMBChange = this.handleRMBChange.bind(this);
        this.handleUSDChange = this.handleUSDChange.bind(this);
    }
    handleRMBChange (amount) {
        this.setState({
            type: 'RMB',
            amount
        });
    }
    handleUSDChange (amount) {
        this.setState({
            type: 'USD',
            amount
        });
    }
    render () {
        const type = this.state.type;
        const amount = this.state.amount;
        const RMB = type==='RMB' ? amount : convert(amount, USB2RMB);
        const USD = type==='USD' ? amount : convert(amount, RMB2USB);
        return (
            <div>
                <p>Please Input:</p>
                <CurrencyInput type="RMB" value={RMB} onInputChange={this.handleRMBChange} />
                <CurrencyInput type="USD" value={USD} onInputChange={this.handleUSDChange} />
            </div>
        );
    }
}

```

#### 14、组合和继承

---

React推崇更多的是使用组合，而非使用继承。对于一些使用场景，React给出的建议如下：

##### 1、包含关系

当父组件不知道子组件可能的内容是什么的时候，可以使用**props.children**，如：

```
function Article (props) {
    return (
        <section>
            <aside>侧边栏</aside>
            <article>{props.children}</article>
        </section>
    );
}

function App () {
    return (
        <Article>这是一篇文章</Article>
    );
}

```

这将渲染得到：

```
<section>
    <aside>侧边栏</aside>
    <article>这是一篇文章</article>
</section>

```

我们还可以自定义名称，因为JSX实际上会被转化为合法的JS表达式，所以，还可以有：

```
function Article (props) {
    return (
        <section>
            <aside>{props.aside}</aside>
            <article>{props.children}</article>
        </section>
    );
}

function App () {
    return (
        <Article aside={
            <h1>这是一个侧栏</h1>
        }>这是一篇文章</Article>
    );
}

```

##### 2、如何使用继承

在Facebook的网站上，使用了数以千计的组件，但是实践证明还没有发现需要使用继承才能解决的情况。
属性和组合为我们提供了清晰的、安全的方式来自定义组件的样式和行为，组件可以接受任意元素，包括：基本数据类型、React元素、函数。
如果要在组件之间复用UI无关的功能，那么应该将其提取到单独的JavaScript模块中，这样子可以在不对组件进行扩展的前提下导入并使用函数、对象、类




