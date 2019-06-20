---
title: 手写async函数
date: 2019-03-05 21:58:18
tags: javascript
---

![1327f9baa13e86641c721d111e51bc1e.jpeg](evernotecid://2E1F95D0-6D94-423F-BC95-0D422C913EFE/appyinxiangcom/22692071/ENResource/p24)

## js的异步历史

##### 我们都知道JavaScript是单线程的，避开了操作多线程的上锁和复杂的状态同步问题，单线程是无法充分利用cpu的，不过早期的JavaScript只是作为浏览器的脚本工具实现的，因此采用单线程是最方便最省事的，作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

## 水深火热的callback

##### 受制于单线程的原因，JavaScript在处理异步任务的时候就出现了比较尴尬的局面，比如AJAX请求，如果事情只能一件一件的来，当用户到服务器上去请求的时候，必须干等到结果回来才能继续后面的动作，这显而易见是不能被接受的，因此callback顺势而出，到后来的nodejs崛起，实现IO读取等异步的方式都是采用了callback的方式，也诞生了臭名昭著的callback hell，场面开始失控...

```javascript
function (param, cb){
    false.readFile(param, function(err, data){
        if(err) return cb(err);
        async1(data, function(err, data){
            if(err) return cb(err);
            async2(data, function(err, data){
                if(err) return cb(err);
                // asyncN... 不知道会有多少
            })
        })
    })
}
```

## 拯救callback的promise

>我的意中人是盖世英雄，有一天他会踏着七色云彩前来拯救我 

##### es6将promise纳入了JavaScript的标准，promise代表着“承诺”，初始化了一个“承诺”之后，你只要在then中定义好这个承诺若是在未来达成要做什么事情，在catch中定义好这个承诺失败了需要做什么事情，就ok了

```javascript
const somePromiseObject = return new Promise((resolve, reject) => {
    const result = doSomeThing();
    if(result) {
        return resolve(result); // handle success
    }
    reject('err') // handle error
})

somePromiseObject
    .then(data => {
        // success callback  
    },
    err => {
        // err callback
    })
```

##### promise的出现完美的解救了callback的回调地狱，原因是promise允许链式调用，并且可以统一到最外层去做错误的catch~

```javascript
somePromiseObject
    .then(data => {
        // do someThings..
    })
    .then(data => {
        // do someThings..
    })
    .then(data => {
        // do someThings..
    })
    .catch(err => {
        // handle errs here
    })
```

#### but 当我们认为promise就是JavaScript解决异步的最优方案的时候，ES2017 标准又引入了async 函数，使得异步操作变得更加方便

## 异步终极利器async函数

>我猜中了前头可我猜不中这结局 --promise

##### 没错就在es6将promise加入标准后的不久，es7又新加入一种新的异步解决方案（号称终极解决方案）-- async函数

##### 假设有个异步函数

```javascript
ajax1().then(() => {
    ajax2().then(() => {
        ajax3().then((data) => {
            console.log(data) // success
        }, err => {
            console.log(err)
        })
    },
    err => {
        console.log(err)
    })
}, err => {
    console.log(err)
})
```

##### 好吧我们赶紧试试用async来实现

```javascript
const asyncFun = async () => {
    try{
        await ajax1();
        await ajax2();
        const data = await ajax3();
        console.log(data) // success
    }catch(err){
        console.log(err) // handle errs here
    }
}
```

##### 可以看到明显的看到，async的写法比promise的链式更舒服，它允许我们像写同步的语法一样去写嵌套的异步，实际的效果也会按照async 被 await 的顺序来运行，而且对开发最友好的是所有中间出现的错误都能被最外层的catch给捕获，所以说很多人都认为async将是js异步的终极解决方案，下面我们先来认识一下实现async的函数的一个基础函数，generator函数~

## 什么是generator

* 可以先看下阮一峰老师的[概念理解](http://es6.ruanyifeng.com/#docs/generator)
* generator函数最神奇的地方在于它可以交出函数的控制权，什么意思呢，我们正常情况下的函数一旦被执行就会全部执行结束，除非发生错误，是不能在执行中间停下来的，而generator函数就不一样了，它可以用yeild关键字来将函数的执行暂停，generator函数执行后会返回一个迭代器(也可以理解为指针)，这个指针对象拥有一个关键的next方法，用来移动当前指针所在的位置，每次调用next返回一个对象，包含2个内容：value：当前generator函数中yeild语句后面表达式的值，done：bool值，标识当前迭代器是否结束迭代了，也就是说是否所有的yeild语句都走完了；
* genarator 不仅仅可以暂停一个函数的执行，还可以在执行的时候送入数据给函数，改变函数中yeild关键字后面表达式的值；
```javascript
    function* gen(x) {
        const y = yield x + 2;
        const y1 = yield y + 3;
        return y1;
    }
    
    var g = gen(1);
    console.log(g.next()); // { value: 3, done: false }
    console.log(g.next(2)); // { value: 5, done: false }
    console.log(g.next()); // { value: undefined, done: true }
```
* 可以看到运行g.next(2)之前y的值已经是3了，+3应该是6，实际返回的确实5，因为我们给入的值会替换掉yeild关键字后面的表达式，也就是y=2了，所以next(2)后的值是5而不是6；

##### 既然generator可以暂停，如果我们每次遇到promise就暂停，等拿到promise.then()执行的结果在返回data，岂不是就是async函数的表现形式？dei，这确实就是async实现的基本原理，下面我们一起来手撸一个模拟async的函数--"myAsync"，盘它！

## 盘它

1. 根据async函数的表现，async关键字后面跟的是一个generator函数，我们用函数来模拟，如下结果：
```javascript
    // 函数A(正常情况);
const test = async function myGenerator(){
    const data = await Promise.resolve("success");
    console.log(data);
}

// 函数B(模拟情况);
function myAsync(myGenerator) {
    // handle...
}
function* myGenerator() {
    const data = yield Promise.resolve("success");
    console.log(data); // success
}
const test = myAsync(myGenerator);
```
2. ok，然后我们来实现myAsync函数，这个函数会接受一个generator函数，我们知道generator函数调用后才会生成迭代器，拿到迭代器后，我们肯定需要调用next()来获取下一个yeild的值，然后如果这个值是一个promise，那我们就调用then拿到结果后再next到下一次的迭代中，否则，我们直接把拿到的数据不做处理直接给到next--核心思想：generator的迭代器结果，是promise就等异步完成，否则就直接返回数据，然后递归调用handle函数处理下一个迭代，直到迭代器的done是true，返回，看代码：

```javascript
// 函数B(模拟情况);
function myAsync(myGenerator) {
    const gen = myGenerator(); // 生成迭代器
    const handle = genResult => {
        if (genResult.done) return; // 如果迭代器结束了，直接返回；
        return genResult.value instanceof Promise // 判断当前迭代器的value是否是Promise的实例
            ? genResult.value
                  // 如果是，则等待异步完成后继续递归下一个迭代，并把resolve后的data带过去
                  .then(data => handle(gen.next(data)))
                  .catch(err => gen.throw(err)) // gen.throw 可以抛出一个允许外层去catch的err
            : handle(gen.next(genResult.value)); // 如果不是promise，就可以直接递归下一次迭代了
    };
    try {
        handle(gen.next()); // 开始处理next迭代
    } catch (err) {
        throw err;
    }
}
function* myGenerator() {
    const data = yield Promise.resolve("success");
    console.log(data); // success
}
const test = myAsync(myGenerator);
```

>到现在其实我们的核心功能handle函数就完成了，现在去调用next()方法，已经看到可以打印success了，但是，除了正常的promise，我们还要考虑下面的情况

```javascript
// 函数A(正常情况);
const a = {
    then: () => {
        console.log("then");
        return 123123;
    }
};

const test0 = async function() {};

const test1 = async function() {
    return 123;
};
const test2 = async function() {
    console.log(123);
};
const test4 = async function() {
    return a;
};

test0().then(console.log); // undefined
test1().then(console.log); // 123
test2().then(console.log); // 123 undefined
test4().then(console.log); // then
```

>也就是说在async接受到的函数不是generator函数的情况下：

1. async函数默认返回一Promise.resolve(undefined)
2. 当async接受到的函数有返回值，并且返回值不是promise的情况下，async函数默认用promise包装这个返回结果
3. 当async接受到函数没有返回值，async会直接运行函数，并返回一个Promise.resolve(undefined)
4. 考虑到thenable这种promise的鸭子类型函数的特殊性，我尝试去返回了一个这种类型的对象，果然，发现then被执行了~

##### 那么，我们来完善边界情况的处理

>所以现在的代码变成

```javascript
// 函数B(模拟情况);
function myAsync(myGenerator) {
    // 判断接受到的参数不是一个generator函数
    if (
        Object.prototype.toString.call(myGenerator) !==
        "[object GeneratorFunction]"
    ) {
        // 如果是一个普通函数
        if (
            Object.prototype.toString.call(myGenerator) === "[object Function]"
        ) {
            return new Promise((resolve, reject) => {
                // 默认返回一个promise对象
                try {
                    const data = myGenerator();
                    return resolve(data); // 尝试运行这个函数，并把结果resolve出去
                } catch (err) {
                    return reject(err); // 失败处理
                }
            });
        }
        // 如果参数含有then这个方法--thenable 鸭子类型
        if (typeof myGenerator.then === "function") {
            return new Promise((resolve, reject) => {
                try {
                    // 运行这个对象的then函数，并resolve出去
                    const data = myGenerator.then();
                    return resolve(data);
                } catch (err) {
                    return reject(err); // 失败处理
                }
            });
        }
        // 剩下的情况，统一resolve出去给的参数
        return Promise.resolve(myGenerator);
    }
    const gen = myGenerator(); // 生成迭代器
    const handle = genResult => {
        if (genResult.done) return; // 如果迭代器结束了，直接返回；
        return genResult.value instanceof Promise // 判断当前迭代器的value是否是Promise的实例
            ? genResult.value
                  .then(data => handle(gen.next(data))) // 如果是，则等待异步完成后继续递归下一个迭代，并把resolve后的data带过去
                  .catch(err => gen.throw(err)) // gen.throw 可以抛出一个允许外层去catch的err
            : handle(gen.next(genResult.value)); // 如果不是promise，就可以直接递归下一次迭代了
    };
    try {
        handle(gen.next()); // 开始处理next迭代
    } catch (err) {
        throw err;
    }
}

const a = {
    then: () => {
        console.log("then");
        return 123123;
    }
};

const test0 = myAsync(function() {});

const test1 = myAsync(function() {
    return 123;
});

const test2 = myAsync(function() {
    console.log(123);
});
const test4 = myAsync(function() {
    return a;
});

test0.then(console.log); // undefined
test1.then(console.log); // 123
test2.then(console.log); // 123 undefined
test4.then(console.log); // then
```

>有的同学可能会说，咦，es6的 async 函数返回的那个是个函数，需要执行才能拿到结果，比如test0 应该是 test() 返回的是promise，但是模拟出来的怎么直接就拿到返回的promise了，其实很简单，你只要再用一个函数(比如myAsyncWrapper)把这个函数包装一下就ok~

##### 到此，我们的模拟就结束了，现在让我们来尽情的实验一下~

```javascript
myAsync(function*() {
    try {
        const data1 = yield new Promise(res => {
            setTimeout(() => {
                res(1234);
                console.log("step 1"); // step 1
            }, 1000);
        });
        console.log(data1); // step 1 打印1s后 打印 123
        const data2 = yield new Promise(res => {
            setTimeout(() => {
                res(12342);
                console.log("step 2"); // step 2
            }, 1000);
        });
        console.log(data2); // step 2 打印1s后打印 12342
    } catch (err) {
        console.log(888, err); 
    }
});
```

>完美啊有么有，再来试试reject

```javascript
myAsync(function*() {
    try {
        yield Promise.reject(123);
        const data1 = yield new Promise(res => {
            setTimeout(() => {
                res(1234);
                console.log("step 1");
            }, 1000);
        });
        console.log(data1);
        const data2 = yield new Promise(res => {
            setTimeout(() => {
                res(12342);
                console.log("step 2");
            }, 1000);
        });
        console.log(data2);
    } catch (err) {
        console.log(888, err); // 888 123
    }
});
```

>没问题直接输出了888 123

##### 完美结束


## 总结

##### 1.async 函数是generator+promise的语法糖，可能es6具体的实现一定有一定的优化和处理，但是个人觉得理论上偏差不大

##### 2.想要用的得心应手还是建议去猜测和理解一下实现的原理

#### 3.特此声明：以上内容基本都是模拟实现，并不是真实的实现原理，没有写测试，因此可能有漏洞的存在，欢迎各位大佬批评指正，希望大家能够共同进步~


> [代码demo传送门](https://github.com/jinggk/FrontEndCode/blob/master/async/demo.js)
