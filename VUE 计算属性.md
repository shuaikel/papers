###### VUE 计算属性

:端口查询：lsof -i tcp:[port端口号]
:关闭端口占用程序：kill [port端口号]

Vue里的Computed属性非常频繁的被使用到，但并不是很清楚它的实现原理，比如：计算属性如何与属性建立依赖关系，属性发生变化又如何通知到计算属性重新计算？

###### data属性初始化 getter setter

```
// src/observer/index.js

// 从这里开始转换data的getter、setter，原始值已存入到__ob__属性中

Object.defineProperty(obj,key,{
	enumerable : true,
	configurable : true,
	get :function reactiveGetter(){
		const value = getter ? getter.call(obj) : val
		<!--判断是否处于依赖收集状态-->
		if (Dep.target){
		<!--建立依赖关系-->
		dep.depend()
		}
		return value
	},
	set : function reactiveSetter(newVal){
	...
	// 依赖发生变化，通知到计算属性重新计算
	dep.notify()
	}
})

```

######  computed 计算属性初始化

```
// src/core/instance/state.js
// 初始化计算属性

function initComputed(vm:Component , computed : Object){
	...
	// 遍历computed计算属性
	for (const key in computed){
	// 创建watcher实例
	watchers[key] = new Watcher(vm,getter || noop , noop , computedWatcherOptions)
	
	// 创建属性vm.reversedMessage,并将提供的函数用作属性vm.reversedMessage的getter，最终computed与data会一起混合到vm下，所以当computed与data存在重名属性时会抛出警告
	defineComputed(vm,key,userDef)
	}
}

export function defineComputed(target:any,key:string,userDef:Object| function){
	// 创建get set方法
	sharePropertyDefinition.get = createComputedGetter(key)
	sharePropertyDefinition.set = noop
	
	// 创建属性vm.reversedMessage ,并初始化getter setter
	Object.defineProperty(target , key ,sharePropertyDefinition)
}

function createComputedGetter(key){
	return function computedGetter(){
		const watcher = this._computedWATCHERS && this._computedWatchers[key]
		if (watcher){
			if (wacther.dirty){
				// watcher 暴露evaluate方法用于取值
				watcher.evaluate()
			}
			// 同第一步，判断是否处于依赖收集状态
			if (Dep.target){
				// 
				watcher.depend()
			}
			return watcher.value
		}
	}
}
```

###### 无论是属性还是计算属性，都会生成一个watcher实例

```
// 将当前watcher实例暂存到Dep.target，表示开启了依赖收集任务
get(){
pushTarget(this)

let value

const vm = this.vm

try{
// 在触发vm.reversedMessage的函数调用时，会触发属性和计算属性的getter，在这个执行过程中，就可以收集到vm.reversedMessage的依赖了
	value = this.getter.call(vm,vm)
}catch(e){
	//
	if (this.user){
		handleError(e,vm,'getter for watcher "${this.expression}"')
	}else{
		throw e
	}
}finally{
	if(this.deep){
		traverse(value)
	}
	
	// 结束依赖收集任务
	popTarget()
	this.cleanupDeps()
	}
	return value
}
```

###### 上面多次提到了dep.depend , dep.notify , Dep.target 那么Dep究竟是什么呢？
Dep承担着非常重要的依赖收集环节

```
// src/core/observer/dep.js

export default class Dep {
	static target : ? Watcher;
	id : number;
	subs : Array<Watcher>;
	
	constructor(){
		this.id = uid++;
		this.subs = [];
	}
	
	addSub (sub : Watcher){
		this.subs.push(sub)
	}
	
	removeSub(sub : Watcher){
		remove(this.subs ,sub)
	}
	
	depend(){
		if(Dep.target){
			Dep.target.addDep(this)
		}
	}
	
	notify(){
		const subs = this.subs.slice()
		for (leti = 0;  l = subs.length; i < l; i++){
			// 更新watcher的值，与watcher.evaluate()类似
			// 但update是给依赖变化时使用的，包含对watch的处理
			subs[i].update()
		}
	}
}

// 当首次计算computed属性的值时，Dep将会在计算期间对依赖进行收集

Dep.target = null
const targetStack = []

export function pushTarget(_target : Watcher){
	// 在一次依赖收集期间，如果有其他依赖收集任务开始（比如：当前的computed嵌套其他的computed属性）
	// 那么将会把当前target暂存到targetStack，先进行其他target的依赖收集
	if(Dep.target)targetStack(Dep.target)
	Dep.target = _target
}

export function popTarget(){
   // 当收集的依赖收集任务完成后，将target恢复为上一层的watcher，并继续做依赖收集
	Dep.target = targetStack.pop()
}

```

###### 总结一下依赖收集、动态计算的流程：
```
1. data属性初始化getter setter
2. computed计算属性初始化，提供的函数将用作属性vm.reversedMessage 的 getter
3. 当首次获取reversedMessage计算属性的值时，Dep开始依赖收集
4. 在执行message getter方法时，如果Dep处于依赖收集状态，则判定message 为 reversedMessage 的依赖，并建立依赖关系
5. 当 message 发生变化时，根据依赖关系，触发 reverseMessage 的重新计算
```