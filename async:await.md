###### 什么是async

async 函数是 Generator 函数的语法糖。使用 关键字 async来表示，在函数内部使用 await 来表示异步。想较于 Generator，async函数的改进在于下面四点：

+	内置执行器：Generator 函数的执行必须依靠执行器，而async函数自带执行器，调用方式跟普通函数调用一样。
+ 	更好的语义：async 和 await 相对于 * 和 yield 更加语义化。
+  更广的适用性：co模块约定，yield命令后面只能是Thunk函数或Promise对象。而async 函数的await 命令后面则可以是Promise或者 原始类型的值（Number ， string ，boolenan ，但这时等同于同步操作）
+  返回值是Promise。async函数返回值是Promise对象，比Generator函数返回Iterator对象方便，但这时等同于同步操作）

async 是ES7新出的特性，表明当前函数时异步函数，不会阻塞线程导致后续代码停止运行。

async定义的函数内部会默认返回一个promise对象，如果函数内部抛出异常或者是返回reject，都会使函数的promise状态为失败reject


async 函数接收到返回的值，发现不是异常或者reject，则判定成功，这里可以return各种数据类型的值，false，Nan，undefined，都是resolve，但是返回如下结果会使async函数判定失败reject

```
1. 内部含有直接使用并且并未声明的变量或函数
2. 内部抛出一个错误throw new Error或者返回reject状态return Promise.reject（‘执行失败’）
3. 函数方法执行出错，等等
```

###### 什么是await

await的意思是saync wait（异步等待），这个关键字只能在使用async定义的函数里面使用，任何async函数都会默认返回Promise，并且这个promise解析的值将会是这个函数的返回值，而async函数必须等到内部所有的await命令的Promise对象执行完，才会发生状态改变。

```
很多人以为await会一直等待之后的表达式执行完之后才会继续执行后面的代码，实际上await是一个让出线程的标志。await后面的函数会先执行一遍，然后就会跳出整个async函数来执行后面js栈的代码。等本轮事件循环执行完了之后又会跳回到async函数中等待await后面表达式的返回值，如果返回值为非promise则继续执行async函数后面的代码，否则将返回的promise放入Promise队列（Promise的Job Queue）

链接：https://juejin.im/post/5b1ffff96fb9a01e345ba704
```

async 函数如果有多个await函数的时候，如果其中任一一个跑出异常或者报错了，都会导致函数停止执行，直接reject

那么怎么处理呢，可以用try/catch，遇到函数时，可以将错误抛出，并且继续往下执行。

```
let last;
async functiomn throwError(){
	try{
		await Promise.reject('error');
		last = await '这里不会执行'
	}catch（error）{
		console.log('has Error stop')
	}
}

throwError().then(success=>{console,log('成功'，last)})
			.catch(error=>{console.log('失败'，last)})
```
