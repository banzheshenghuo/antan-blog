---
title: js中的深拷贝和浅拷贝
date: 2017-03-19 12:11:59
tags: deep clone
categories: javascript
---
之所有写这篇文章，是因为前几天在项目中被ES6的assign坑了，后来自己查阅了些资料，觉得有必要总结下。

### 浅拷贝
平常我们把字符串，数字的值赋值给新变量，相当于把值完全复制过去，新变量的值改变不会影响旧变量，

```js
let num = 123
let num2 = num
num2 = 321
console.log(num) //123
console.log(num2) //321
```

但是对象却不同

```js
let obj = { name: '程序猿' }
let obj2 = obj
obj2.name = '单身狗'
console.log(obj) //{ name: '单身狗' }
```

因为都是引用的同一个地址，所以你变他也变（0.0 真是调皮）

ES6出来后，新增了Object.assign 用来复制对象，嗯~~~来试试

```js
let obj = { name: '程序猿', age:{child: 12} }
let copy = Object.assign({}, obj);
copy.name = '单身狗'
copy.age.child = 24
console.log(obj) // { name: '程序猿', age:{child: 24} }
```

嗯~~~不错，程序猿还没变成单身狗，纳尼child怎么变成24了，我怕是用了假的ES6吧
查了查MDN，才发现自己对Object.assign的一点都不了解，就乱用，真是自己作死

![warning for deep clone](https://zqfile.banzheshenghuo.com/20210402153607.png)

简单说就是如果拷贝的对象的属性值也是一个引用（对象），就只会把引用给拷贝过来。而且这货用的最多的地方不是浅拷贝而是将多个对象的属性合并到一个对象上

```js
Object.assign( target, obj1, obj2 )
```

第一个参数是目标对象，后面的都是源对象，将源对象的属性复制到目标对象中
###深拷贝
既然没有现成的深拷贝的方法，那有哪些方式可以达到深拷贝呢？

对象的赋值会相互影响，而数字，字符串之类的不会，我们将对象逐渐遍历，在数字，字符串将其对应赋值，这就是一般深拷贝的方式

```js
function cloneDeep(obj) {
  if (typeof obj !== "object" || Object.keys(obj).length === 0) {
    return obj
  }
  let resultData = {}
  return recursion(obj, resultData)
}

function recursion(obj, data = {}) {
  for (key in obj) {
    if (typeof obj[key] == "object" && Object.keys(obj[key].length > 0)) {
      data[key] = recursion(obj[key])
    } else {
      data[key] = obj[key]
    }
  }
  return data
}
let obj = { name: "程序猿", age: { child: 20 } }
let obj2 = cloneDeep(obj)
obj.name = "单身狗"
obj2.age.child = 24
console.log(obj) // {name:'程序猿',age:{child:20}}
```

一长串的看起来好麻烦啊，我只是想深拷贝个对象啊
这里有一种取巧的方式就是使用json转换，

``` js
let h = JSON.parse(JSON.stringify(a));
```

这里是将对象转化为json字符串又转换为json对象，之前什么引用的全变了样儿，**不过弊端也是有的，继承的属性会丢失**

这里推荐使用Lodash类库

参考资料
- [MDN assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign )
- [Lodash](https://lodash.com/)