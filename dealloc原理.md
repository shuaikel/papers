#### dealloc原理

我们在NSObject.mm 可以找到dealloc函数

```
- (void)dealloc {
	_objc_rootDealloc(self);
}
```

点击进入\_objc_rootDealloc函数


```
_objc_rootDealloc(id obj)
{
	assert(obj);

	obj->rootDealloc();
}
```

里面还是没有我们想要的，点击rootDealloc


```
objc_object::rootDealloc()
{
	if (isTaggedPointer()) return;
	object_dispose((id)this);
}

```

这个里面有我们想要的信息：

+ 首先判断对象是不是isTaggedPointer，如果是TaggedPointer那么没有采用引用计数技术，所以直接return

+ 不是TaggedPointer，就去销毁这个对象object_dispose

我们继续点击object_dispose

```
id 
object_dispose(id obj)
{
	if (!obj) return nil;
	
	objc_destructInstance(obj);    
	free(obj);
	
	return nil;
}
```

里面仅仅是简单的判断，我们还需要继续找我们需要的东西，点击objc_destructInstance函数


```
void *objc_destructInstance(id obj) 
{
	if (obj) {
	// Read all of the flags at once for performance.
	bool cxx = obj->hasCxxDtor();
	bool assoc = obj->hasAssociatedObjects();
	
	// This order is important.
	if (cxx) object_cxxDestruct(obj);
	if (assoc) _object_remove_assocations(obj);//清除成员变量
	obj->clearDeallocating(); //将指向当前对象的弱引用指针置为nil
	}
	
	return obj;
}
```

#### 主要步骤

1、 首先判断对象是不是isTaggedPointer，如果是TaggedPointer那么没有采用引用计数技术，所以直接return


2、不是TaggedPointer

	+ 1、清除成员变量
	+ 2、将指向当前对象的弱引用指针置为nil