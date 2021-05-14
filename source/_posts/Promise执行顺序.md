---
title: Promise执行顺序
date: 2019-12-13 15:42:01
tags:
---
> 面试经常会考 Promise 的执行顺序，这里简单记录我的理解和关键点。

# 基本流程
先简单说下事件循环的执行过程。

1. 首先执行最初的宏任务（一般是脚本的初始化）
2. 在执行过程中
    - 如果有等待执行的微任务就加入当前宏任务的微任务队列
    - 如果等待执行的宏任务则加入宏任务队列
3. 执行完毕后，拉取当前宏任务所属的微任务执行
4. 执行完微任务队列所有微任务后，拉取下一个宏任务执行，循环上述流程

然后说说 `promise` 执行过程中的需要注意的点

# Promise.then 中的回调函数才是微任务

`Promise`在实例化的过程中是同步执行的，只有`Promise`的then方法的回调函数才属于微任务。
```js
// 例子1
console.log('start')
new Promise((resolve)=>{
    console.log('promise')
}).then(()=>{
	console.log('then')
})
console.log('end')

//执行结果
// start
// promise
// end
// then
```

# Promise.then 回调函数何时加入微任务队列

这里要知道三个操作，分别是 `执行`、`加入微任务队列`、`注册then回调函数`。
- 执行  执行操作就是真正去运行代码。
- 加入微任务队列    在执行的过程中，发现状态变更的微任务，那么就直接加入微任务队列（不是发现微任务就立即加入微任务队列）。
- 注册then回调函数  不是所有的then回调都会立即加入微任务队列，`Promise`的状态为`pending`时，只会先进行注册然后等待状态变更后再加入微任务队列。
> 上面说的状态就是`Promise`的状态，只会从 `penging` 变为 `resolved` 或 `rejected`

那么何时加入微任务队列呢？

首先当执行到then方法时，查看`Promise`状态，
- 如果状态已经变更，那么就直接加入微任务队列。
- 如果状态没变，则先进行注册（如果有多个then回调会注册多次，类似队列），等待后续状态变更后再加入微任务队列。

第一种情况可以参考上面的例子1，在实例化时直接调用了`resolve()`，所以执行完当前的宏任务后在执行 then 回调

第二种情况如下
```js
// 例子2
console.log('start')
new Promise((resolve)=>{
    console.log('promise1')
    setTimeout(resolve)
}).then(()=>{
    console.log('then1')
})

new Promise((resolve)=>{
	console.log('promise2')
    resolve()
}).then(()=>{
    console.log('then2')
})
console.log('end')

// 执行顺序
// start
// promise1
// promise2
// end
// then2
// then1
```
可以看到，第一个`Promise` 先于第二个`Promise`执行，但是因为第一个`Promise`的状态延时改变，所以只注册而没有加入微任务队列。而第二个`Promise`在实例化时就变为 `resolved`，所以他的 then回调 直接加入微任务队列早一步执行。

再回看第一个`Promise`，在状态延迟变更后会将之前注册的then回调加入微任务队列。

# then回调的返回值

首先明确一点，then回调函数返回的也是一个`Promise`。

需要注意的是返回值的类型。
- 返回非Promise类型的值  一般会被`Promise.resolve` 进行包裹，表示当前执行结束后立即修改状态为`resolved`。
- 返回Promise类型的值 等待该 `Promise`
	
上面2种情况貌似看起来没啥意义，但是如果后面还有 then回调 就可以看出他们的区别，可以看看下面2个例子。

```js
// 例子3
Promise.resolve().then(()=>{
   console.log('外层第一个then')
   return Promise.resolve().then(()=>{
        console.log('内部第一个then')
    }).then(()=>{
        console.log('内部第二个then')
    })
}).then(()=>{
    console.log('外部第二个then')
})

// 执行
// 外层第一个then
// 内部第一个then
// 内部第二个then
// 外部第二个then
```

```js
// 例子4
Promise.resolve().then(()=>{
    console.log('外层第一个then')
    Promise.resolve().then(()=>{
        console.log('内部第一个then')
    }).then(()=>{
        console.log('内部第二个then')
    })
}).then(()=>{
    console.log('外部第二个then')
})

// 执行
// 外层第一个then
// 内部第一个then
// 外部第二个then
// 内部第二个then
```


