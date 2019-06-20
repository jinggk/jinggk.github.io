---
title: proxy 简介
date: 2019-06-06 14:23:42
tags: javascript
---

## 什么是Proxy

proxy翻译过来的意思就是”代理“，ES6对Proxy的定位就是target对象(原对象)的基础上通过handler增加一层”拦截“，返回一个新的代理对象，之后所有在Proxy中被拦截的属性，都可以定制化一些新的流程在上面，先看一个最简单的例子

```javascript
const target = {}; // 要被代理的原对象
// 用于描述代理过程的handler
const handler = {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
}
// obj就是一个被新的代理对象
const obj = new Proxy(target, handler);
obj.a = 1 // setting a!
console.log(obj.a) // getting a!
```

上面的例子中我们在target对象上架设了一层handler，其中拦截了针对target的get和set，然后我们就可以在get和set中间做一些额外的操作了


> 注意：对Proxy对象的赋值操作也会影响到原对象target，同时对target的操作也会影响Proxy，不过直接操作原对象的话不会触发拦截的内容~

```javascript
obj.a = 1; // setting a!
console.log(target.a) // 1 不会打印"getting a!"
```

> 注意：如果handler中没有任何拦截上的处理，那么对代理对象的操作会直接通向原对象

```javascript
const target = {};
const handler = {};
const obj = new Proxy(target, handler);
obj.a = 1;
console.log(target.a) // 1
```

> 既然proxy也是一个对象，那么它就可以做为原型对象，所以我们把obj的原型指向到proxy上后，发现对obj的操作会找到原型上的代理对象，如果obj自己有a属性，则不会触发proxy上的get，这个应该很好理解

```javascript
const target = {};
const handler = {
    get: function(target, key){
		console.log(`get ${key} from ${JSON.stringify(target)}`);
		return Reflect.get(target, key);
	}
}
const proxy = new Proxy(target, handler);
const obj = Object.setPrototypeOf(obj, proxy);
proxy.a = 1;
console.log(obj.a) // get a from {"a": 1}
```


## ES6的Proxy实现了对哪些属性的拦截？

> 通过上面的例子了解了Proxy的原理后，我们来看下ES6目前实现了哪些属性的拦截，以及他们分别可以做什么？
> 下面是 Proxy 支持的拦截操作一览，一共 13 种
1. get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo'];
2. set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值;
3. has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
4. deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值;
5. ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性;
6. getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象;
7. defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值;
8. preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值;
9. getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象;
10. isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值;
11. setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截;
12. apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...);
13. construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args);

以上是目前es6支持的proxy，具体的用法不做赘述，有兴趣的可以到[阮一峰老师的es6入门](http://es6.ruanyifeng.com/#docs/proxy)去研究每种的具体用法，其实思想都是一样的，只是每种对应了一些不同的功能~

## 实际场景中 Proxy 可以做什么？

### 实现私有变量
js的语法中没有private这个关键字来修饰私有变量，所以基本上所有的class的属性都是可以被访问的，但是在有些场景下我们需要使用到私有变量，现在业界的一些做法都是使用”_变量名“来”约定“这是一个私有变量，但是如果哪天被别人从外部改掉的话，我们还是没有办法阻止的，然而，当Proxy出现后，我们可以用代理来处理这种场景，看代码：

```javascript
const obj = {
    _name: 'nanjin',
    age: 19,
    getName: () => {
        return this._name;
    },
    setName: (newName) => {
        this._name = newName;
    }
}

const proxyObj = obj => new Proxy(obj, {
    get: (target, key) => {
        if(key.startsWith('_')){
            throw new Error(`${key} is private key, please use get${key}`)
        }
        return Reflect.get(target, key);
    },
    set: (target, key, newVal) => {
        if(key.startsWith('_')){
            throw new Error(`${key} is private key, please use set${key}`)
        }
        return Reflect.set(target, key, newVal);
    }
})

const newObj = proxyObj(obj);
console.log(newObj._name) // Uncaught Error: _name is private key, please use get_name
newObj._name = 'newname'; // Uncaught Error: _name is private key, please use set_name
console.log(newObj.age) // 19
console.log(newObj.getName()) // nanjin
```

可见，通过proxyObj方法，我们可以实现把任何一个对象都过滤一次，然后返回新的代理对象，被处理的对象会把所有_开头的变量给拦截掉，更进一步，如果有用过mobx的同学会发现mobx里面的store中的对象都是类似于这样的

<img src="https://user-gold-cdn.xitu.io/2019/6/6/16b2b95a4f3b0c99?imageView2/0/w/1280/h/960/format/webp/ignore-error/1">

有handler 和 target，说明mobx本身也是用了代理模式，同时加上Decorator函数，在这里就相当于把proxyObj使用装饰器的方式来实现，Proxy + Decorator 就是mobx的核心原理啦~


### 实现vue的双向绑定数据响应

VUE的双向绑定涉及到模板编译，响应式数据，订阅者模式等等，有兴趣的可以看[这里](https://github.com/KieSun/Dream/issues/7)，因为这篇文章的主题是proxy，因此我们着重介绍一下数据响应式的过程。

#### 2.x版本
在当前的vue2.x的版本中，在data中声名一个obj后，vue会利用Object.defineProperty来递归的给data中的数据加上get和set，然后每次set的时候，加入额外的逻辑。来触发对应模板视图的更新，看下伪代码：

```javascript
const defineReactiveData = data => {
	Object.keys(data).forEach(key => {
		let value = data[key];
		Object.defineProperty(data, key, {
         get : function(){
            console.log(`getting ${key}`)
            return value;
         },
         set : function(newValue){
            console.log(`setting ${key}`)
            notify() // 通知相关的模板进行编译
            value = newValue;
         },
         enumerable : true,
         configurable : true
		})
	})
}
```

这个方法可以给data上面的所有属性都加上get和set，当然这只是伪代码，实际场景下我们还需要考虑如果某个属性还是对象我们应该递归下去，来试试：

```javascript
const data = {
	name: 'nanjing',
	age: 19
}
defineReactiveData(data)
data.name // getting name  'nanjing'
data.name = 'beijing';  // setting name
```
可以看到当我们get和set触发的时候，已经能够同时触发我们想要调用的函数拉，Vue双向绑定过程中，当改变this上的data的时候去更新模板的核心原理就是这个方法，通过它我们就能在data的某个属性被set的时候，去触发对应模板的更新。

现在我们在来试试下面的代码：

```javascript
const data = {
    userIds: ['01','02','03','04','05']
}
defineReactiveData(data);
data.userIds // getting userIds ["01", "02", "03", "04", "05"]
// get 过程是没有问题的，现在我们尝试给数组中push一个数据
data.userIds.push('06') // getting userIds 
```

what ? setting没有被触发，反而因为取了一次userIds所以触发了一次getting~，
不仅如此，很多数组的方法都不会触发setting，比如：push,pop,shift,unshift,splice,sort,reverse这些方法都会改变数组，但是不会触发set，所以Vue为了解决这个问题，重新包装了这些函数，同时当这些方法被调用的时候，手动去触发notify()；看下源码：

```javascript
// 获得数组原型
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)
// 重写以下函数
const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse',
]
methodsToPatch.forEach(function(method) {
  // 缓存原生函数
  const original = arrayProto[method]
  // 重写函数
  def(arrayMethods, method, function mutator(...args) {
    // 先调用原生函数获得结果
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    // 调用以下几个函数时，监听新数据
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // 手动派发更新
    ob.dep.notify()
    return result
  })
})
```

上面是官方的源码，我们可以看下实现push的伪代码

```javascript
const push = Array.prototype.push;
Array.prototype.push = function(...args){
	console.log('push is happenning');
	return push.apply(this, args);
}
data.userIds.push('123') // push is happenning
```
通过这种方式，我们可以监听到这些的变化，但是vue官方文档中有这么一个[注意事项](https://cn.vuejs.org/v2/guide/list.html#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
> 由于 JavaScript 的限制，Vue 不能检测以下变动的数组：
> - 当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue
> - 当你修改数组的长度时，例如：vm.items.length = newLength
这个最根本的原因是因为这2种情况下，受制于js本身无法实现监听，所以官方建议用他们自己提供的内置api来实现，我们也可以理解到这里既不是defineProperty可以处理的，也是包一层函数就能解决的，这就是2.x版本现在的一个问题，下面我们回到这篇文章的主题，Proxy，vue官方会在3.x的版本中使用proxy来代替defineProperty处理响应式数据的过程，我们先来模拟一下实现，看看能否解决当前遇到的这些问题；

#### 3.x版本

我们先来通过proxy实现对data对象的get和set的劫持，并返回一个代理的对象，注意，我们只关注proxy本身，所有的实现都是伪代码，也不考虑递归的过程，有兴趣的同学可以自行完善

```javascript
const defineReactiveProxyData = data => new Proxy(data, 
	{
        get: function(data, key){
            console.log(`getting ${key}`)
            return Reflect.get(data, key);
        },
        set: function(data, key, newVal){
            console.log(`setting ${key}`)
			if(typeof newVal === 'object'){
                // 如果不是基本数据类型，递归设置代理
				return Reflect.set(data, key, defineReactiveProxyData(newVal));
			}
            return Reflect.set(data, key, newVal);
        }
    })
const data = {
    name: 'nanjing',
    age: 19
};
const vm = defineReactiveProxyData(data);
vm.name // getting name  nanjing
vm.age = 20; // setting age  20
```

看起来我们的代理已经起作用啦，之后只要在setting的时候加上notify()去通知模板进行编译就可以了，然后我们来尝试设置一个数组看看；

```javascript
vm.userIds = [1,2,3] //  setting userIds
vm.userIds.push(1);
// getting userIds 因为我们会先访问一次userids
// getting push 调用了push方法，所以会访问一次push属性
// getting length 数组push的时候 length会变，所以需要先访问原来的length
// setting 3 通过下标设置的，所以set当前的index是3
// setting length 改变了数组的长度，所以会set length
// 4 返回新的数组的长度
```

ok 2.x版本中遇到的第一个问题，需要重新包装Array.prototype上的一些方法，使用了proxy后不需要了，解决了~，继续看下一个问题

```javascript
vm.userIds.length = 2
// getting userIds 先访问
// setting length 在设置
vm.userIds[1] = '123'
// getting userIds 先访问
// setting 1 设置index=1的item
// "123"
```

从上面的例子中我们可以看到，不管是直接改变数组的length还是通过某一个下标改变数组的内容，proxy都能拦截到这次变化，这比defineProperty方便太多了，2.x版本中的第二个问题，在proxy中根本不会出现了。

### 总结1

通过上面的例子和代码，我们看到Vue的响应模式如果使用proxy会比现在的实现方式要简化和优化很多，很快在即将来临的3.0版本中，大家就可以体验到了。不过因为proxy本身是有兼容性的，比如ie浏览器，所以在低版本的场景下，vue会回退到现在的实现方式。

### 总结2

回归到proxy本身，设计模式中有一种典型的代理模式，proxy就是js的一种实现，它的好处在于，我可以在不污染本身对象的条件下，生成一个新的代理对象，所有的一些针对性逻辑放到代理对象上去实现，这样我可以由A对象，衍生出B,C,D...每个的处理过程都不一样，从而简化代码的复杂性，提升一定的可读性，比如用proxy实现数据库的ORM就是一种很好的应用，其实代码很简单，关键是要理解背后的思想，同时能够举一反三~


### 扩展：

1. Proxy.revocable()
这个方法可以返回一个可取消的代理对象
```javascript
const obj = {};
const handler = {};
const {proxy, revoke} = Proxy.revocable(obj, handler);
proxy.a = 1
proxy.a // 1
revoke();
proxy.a // Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
```

一旦代理被取消了，就不能再从代理对象访问了

<img src="https://user-gold-cdn.xitu.io/2019/6/6/16b2b9693f300f3f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1">
打印proxy 可以看到IsRevoked变为true了


1. 代理对象的this问题
因为new Proxy出来的是一个新的对象，所以在如果你在target中有使用this，被代理后的this将指向新的代理对象，而不是原来的对象，这个时候，如果有些函数是原对象独有的，就会出现this指向导致的问题，这种场景下，建议使用bind来强制绑定this

看代码：
```javascript
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);

proxy.getDate(); // Uncaught TypeError: this is not a Date object.
```
因为代理后的对象并不是一个Date类型的，不具有getDate方法的，所以我们需要在get的时候，绑定一下this的指向

```javascript
const target = new Date();
const handler = {
    get: function(target, key){
        if(typeof target[key] === 'function'){
            return target[key].bind(target) // 强制绑定
            this到原对象
        }
        return Reflect.get(target, key)
    }
};
const proxy = new Proxy(target, handler);

proxy.getDate(); // 6
```
这样就可以正常使用this啦，当然具体的使用还要看具体的场景，灵活运用吧！



> 伪代码部分都是笔者揣摩写的，如有问题，欢迎指正~
