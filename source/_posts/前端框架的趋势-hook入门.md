---
title: 前端框架的趋势-hook入门
date: 2019-06-13 11:17:10
tags: javascript
---

<img src="https://user-gold-cdn.xitu.io/2019/6/13/16b4e61bb6d37766?imageView2/0/w/1280/h/960/format/webp/ignore-error/1">

## 背景

很荣幸在6月8号那天参加了在上海举办的vueconf，其中尤大本人讲解的vue3.0的介绍中，见识到了vue3.0的一些新特性，其中最重要的一项RFC就是 [Vue Function-based API RFC](https://zhuanlan.zhihu.com/p/68477600)，很巧的在不久前正好研究了一下[react hook](https://react.docschina.org/docs/hooks-intro.html)，感觉2者的在思想上有着异曲同工之妙，所以有了一个想总结一下关于hook的想法，同时看到很多人关于hook的介绍都是分开讲的，当然可能和vue3.0对于这个特性的说明刚刚问世也有一定的关系，so，lets begin~

## 什么是hook

首先我们需要了解什么是hook，拿react的介绍来看，它的定义是：
> 它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性

在16.8以前的版本中，我们在写react组件的时候，大部分都都是class component，因为基于class的组件react提供了更多的可操作性，比如拥有自己的state，以及一些生命周期的实现，对于复杂的逻辑来讲class的支持程度是更高的：

```javascript
class Hello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() { // do sth... }

  componentWillUnmount() { // do sth... }
  
  // other methods or lifecycle...
  
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

同时，对于function component来说，react也是支持的，但是function component只能拥有props，不能拥有state，也就是只能实现stateless component：

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

react 并没有提供在函数组件中设置state以及生命周期的一些操作方法，所以那个时候，极少的场景下适合采用函数组件，但是16.8版本出现hook以后情况得到了改变，hook的目标就是--**让你在不编写 class 的情况下使用 state 以及其他的 React 特性**，来看个例子：

```javascript
import React, { useState } from 'react';

function Example() {
  // 声明一个新的叫做 “count” 的 state 变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

useState就是react提供的一个Hook，通过它我们就可以在function组件中设置自己想要的state了，不仅可以使用还可以很方便的去通过setState(注意不是class中的setState，这里指的是上述例子中的setCount)更改，当然，react提供了很多hook来支持不同的行为和操作，下面我们还会再简单介绍，我们在看下vue hook，这是尤大在vueconf上分享的一段代码：

```javascript
import { value, computed, watch, onMounted } from 'vue'

const App = {
  template: `
    <div>
      <span>count is {{ count }}</span>
      <span>plusOne is {{ plusOne }}</span>
      <button @click="increment">count++</button>
    </div>
  `,
  setup() {
    // reactive state
    const count = value(0)
    // computed state
    const plusOne = computed(() => count.value + 1)
    // method
    const increment = () => { count.value++ }
    // watch
    watch(() => count.value * 2, val => {
      console.log(`count * 2 is ${val}`)
    })
    // lifecycle
    onMounted(() => {
      console.log(`mounted`)
    })
    // expose bindings on render context
    return {
      count,
      plusOne,
      increment
    }
  }
}
```
从上面的例子中不难看出，和react hook的用法非常相似，并且尤大也有说这个RFC是借鉴了react hook的想法，但是规避了一些react的问题，然后这里解释一下为什么我把vue的这个RFC也称为是hook，因为在react hook的介绍中有这么一句话，什么是hook--Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数，那么vue提供的这些API的作用也是类似的--可以让你在函数组件里“钩入” value(2.x中的data) 及生命周期等特性的函数，所以，暂且就叫vue-hook吧~


## hook的时代意义

那么，hook的时代意义是什么？我们从头来说，框架是服务于业务的，业务中很难避免的一个问题就是-- 逻辑复用，同样的功能，同样的组件，在不一样的场合下，我们有时候不得不去写2+次，为了避免耦合，后来各大框架纷纷想出了一些办法：
- mixin
- HOC
- slot

各大框架的使用情况：
- react 和 vue都曾用过mixin(react 目前已经废弃),
- Higher-Order-Components(HOC) react中用的相对多一点，vue的话，嵌套template有点。。别扭,
- slot vue中用的多一些，react基本不需要slot这种用法,

上述这些方法都可以实现逻辑上的复用，但是都有一些额外的问题:

- mixin的问题：
    - 可能会相互依赖，相互耦合，不利于代码维护；
    - 不同的mixin中的方法可能会相互冲突;
    - mixin非常多时，组件是可以感知到的，甚至还要为其做相关处理，
这样会给代码造成滚雪球式的复杂性

- HOC的问题：
    - 需要在原组件上进行包裹或者嵌套，如果大量使用HOC，
将会产生非常多的嵌套，这让调试变得非常困难；
    - HOC可以劫持props，在不遵守约定的情况下也可能造成冲突
    - props 也可能造成命名的冲突
    - wrapper hell

**有没有见过这样的dom结构？**

<img src="https://user-gold-cdn.xitu.io/2019/6/12/16b4a8f77e88d56f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1"/>


这就是wrapper hell的典型代表~

所以，hook的出现是划时代的，它通过function抽离的方式，实现了复杂逻辑的内部封装，根据上述我们提出的问题总结了hook的一些优点：

1. 逻辑代码的复用
2. 减小了代码体积
3. 没有this的烦恼

带着这些思想，我们一起看下react和vue分别的实现：


## react hook简介

> Dan 讲解hook的视频在[这里](https://www.youtube.com/watch?v=dpw9EHDh2bM&feature=youtu.be)，如果你看不了这个，可以尝试看[官网介绍](https://react.docschina.org/docs/hooks-intro.html)

我们用同样功能的代码来看react hook，实现一个监听鼠标变化，并实时查看位置的功能，同时我们把位置信息挂到title上面，用class component我们要这样写：

```javascript
import React, { Component } from 'react';

export default class MyClassApp extends Component {
    constructor(props) {
        super(props);
        this.state = {
            x: 0,
            y: 0
        };
        this.handleUpdate = this.handleUpdate.bind(this);
    }
    componentDidMount() {
        document.addEventListener('mousemove', this.handleUpdate);
    }
    componentDidUpdate() {
        const { x, y } = this.state;
        document.title = `(${x},${y})`;
    }
    componentWillUnmount() {
        window.removeEventListener('mousemove', this.handleUpdate);
    }
    handleUpdate(e) {
        this.setState({
            x: e.clientX,
            y: e.clientY
        });
    }
    render() {
        return (
            <div>
                current position x:{this.state.x}, y:{this.state.y}
            </div>
        );
    }
}

```
在线代码演示在[这里](https://codesandbox.io/s/xenodochial-butterfly-gzflh)


同样的逻辑我们换用hook来实现

```javascript
import React, { useState, useEffect } from 'react';

// 自定义hook useMousePostion
const useMousePostion = () => {
    // 使用hookuseState初始化一个state
    const [postion, setPostion] = useState({ x: 0, y: 0 });
    function handleMove(e) {
        setPostion({ x: e.clientX, y: e.clientY });
    }
    // 使用useEffect处理class中生命周期可以做到的事情
    // 注：效果一样，但是实际的原理并不同，有兴趣可以去官网仔细研究
    useEffect(() => {
        // 同时可以处理 componentDidMount 以及 componentDidUpdate 中的事情
        window.addEventListener('mousemove', handleMove);
        document.title = `(${postion.x},${postion.y})`;
        return () => {
            // return的function 可以相当于在组件被卸载的时候执行 类似于 componentWillUnmount
            window.removeEventListener('mousemove', handleMove);
        };
        // [] 是参数，代表deps，也就是说react触发这个hook的时机会和传入的deps有关，内部利用object.is实现
        // 默认不给参数会在每次render的时候调用，给空数组会导致每次比较是一样的，只执行一次，这里正确的应该是给postion
    }, [postion]);
    // postion 可以被直接return，这样达到了逻辑的复用~，哪里需要哪里调用就可以了。
    return postion;
};

export default function App() {
    const { x, y } = useMousePostion(); // 内部维护自己的postion相关的逻辑
    return (
        <div>
            current position x: {x}, y: {y}
        </div>
    );
}

```

在线代码演示在[这里](https://codesandbox.io/s/hookdemo-8zxh2)

可以看出用了hook之后，我们把关于position的逻辑都放到一个自定义的hook--useMousePostion 中，之后复用是很方便的，而且可以在内部进行维护postion独有的逻辑而不影响外部内容，比起class组件，抽象能力更强。

当然，react hook 想要用好不可能这么简单，讲解的文章也很多，这篇文章不深入太多，只是一个抛砖引玉，下面给大家安利几个不错的资源：

入门：
- [官网传送门](https://react.docschina.org/docs/hooks-overview.html)

深入：
- [Dan自己写的关于hook的文章，写的很好强烈安利，深入内部实现](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#tldr)


另外放一个笔者自己关于用hook实现redux的[**最佳实践**](https://codesandbox.io/s/react-hook-redux-zvx57)，注意是笔者自己这么认为的，欢迎大佬们指出问题，参考的[这个文章](https://www.robinwieruch.de/react-hooks-fetch-data/)

## vue hook简介

> 尤大讲解的视频在[这里](https://mp.weixin.qq.com/s/hBqu1A3gIwMVglb-ZWp8oA)

代码因为vue3.0尚未发布，我们还是看尤大给的demo代码：

```javascript
import { value, computed, watch, onMounted } from 'vue'

function useMouse() {
  const x = value(0)
  const y = value(0)
  const update = e => {
    x.value = e.pageX
    y.value = e.pageY
  }
  onMounted(() => {
    window.addEventListener('mousemove', update)
  })
  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })
  return { x, y }
}

// 在组件中使用该函数
const Component = {
  setup() {
    const { x, y } = useMouse()
    // 与其它函数配合使用
    const { z } = useOtherLogic()
    return { x, y, z }
  },
  template: `<div>{{ x }} {{ y }} {{ z }}</div>`
}
```
可以看出来同样我们可以抽离一些需要复用的逻辑到一个单独的函数useMouse中，然后在这个函数里面定义的一些生命周期和value的内容会随着setup函数的调用被“钩入(hook)”到组件上，并且这个函数return出来的数据可以直接被用在模板上，更具体的玩法我们坐等3.0的出现吧。

基础内容不过多介绍，毕竟真实的api还没发布，想了解具体内容的可以来看尤大的[讲解](https://zhuanlan.zhihu.com/p/68477600)

## same & diff Point

看完了2个框架关于hook的实现，我们来做个简单的对比

1. Same Point:
 - 出现的背景，解决的问题是一样的，2个框架都是为了解决逻辑复用过乱，代码体积过大等一些问题，包括this问题，使用function函数我们很少会和this去打交道了。
 - 使用方式类似，都是把可以复用的一些单独的逻辑抽离到一个单独的函数中去，同时返回组件中需要用到的数据，并且内部会自我维护数据的更新，从而触发视图的更新

2. Diff Point:

**实现原理不同**
react hook底层是基于链表实现，调用的条件是每次组件被render的时候都会顺序执行所有的hooks，所以下面的代码会报错
```javascript
function App(){
    const [name, setName] = useState('demo');
    if(condition){
        const [val, setVal] = useState('');
    }
}
```
因为底层是链表，每一个hook的next是指向下一个hook的，if会导致顺序不正确，从而导致报错，所以react是不允许这样使用hook的。

vue hook只会在setup函数被调用的时候被注册一次，react数据更改的时候，会导致重新render，重新render又会重新把hooks重新注册一次，所以react的上手难度更高一些，而vue之所以能避开这些麻烦的问题，根本原因在于它对数据的响应是基于proxy的，这种场景下，只要任何一个更改data的地方，相关的function或者template都会被重新计算，因此避开了react可能遇到的性能上的问题

当然react对这些都有解决方案，想了解的同学可以去看官网有介绍，比如useCallback，useMemo等hook的作用，我们看下尤大对vue和react hook的总结对比：

> 1.整体上更符合 JavaScript 的直觉；

> 2.不受调用顺序的限制，可以有条件地被调用；

> 3.不会在后续更新时不断产生大量的内联函数而影响引擎优化或是导致 GC 压力；

> 4.不需要总是使用 useCallback 来缓存传给子组件的回调以防止过度更新；

> 5.不需要担心传了错误的依赖数组给 useEffect/useMemo/useCallback 从而导致回调中使用了过期的值 —— Vue 的依赖追踪是全自动的。

不得不说，青出于蓝而胜于蓝，vue虽然借鉴了react，但是天然的响应式数据，完美的避开了一些react hook遇到的短板~


## 总结

1. function component 将会是接下来各大框架发展的一个方向，function天然对TS的友好也是一个重要的影响;
2. react hook的上手成本相对于vue会难一些，vue天生规避了一些react中比较难处理的地方;
3. hook一定是大前端的一个趋势，现在才是刚刚开始的阶段：[SwiftUI-Hooks](https://github.com/unixzii/SwiftUI-Hooks), [flutter_hooks](https://github.com/rrousselGit/flutter_hooks)...

> 因为vue3.0的源码尚未发布，有很多实现是猜测的，欢迎提出问题，一起探讨！
