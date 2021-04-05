---
title: 使用 MutationObserver 监听DOM变更
date: 2019-06-08 17:10:20
tags: Web API
---
> 今天在写 tampermonkey 脚本需要监听DOM的变更，想到了之前写的事件循环中的微任务 MutationObserver 就是用来监听DOM变更的，然后学习了一下如何使用并写了这篇博客作为记录。

# 简述
MutationObserver 接口可以监听DOM树的变更。

## 基础示例
```js
// 选择需要观察变动的节点
const targetNode = document.getElementById('some-id');

// 观察器的配置（需要观察什么变动）
const config = { attributes: true, childList: true, subtree: true };

// 当观察到变动时执行的回调函数
const callback = function(mutationsList, observer) {
    // Use traditional 'for loops' for IE 11
    for(let mutation of mutationsList) {
        if (mutation.type === 'childList') {
            console.log('A child node has been added or removed.');
        }
        else if (mutation.type === 'attributes') {
            console.log('The ' + mutation.attributeName + ' attribute was modified.');
        }
    }
};

// 创建一个观察器实例并传入回调函数
const observer = new MutationObserver(callback);

// 以上述配置开始观察目标节点
observer.observe(targetNode, config);

// 之后，可停止观察
observer.disconnect();
```

# 构造函数
使用`MutationObserver`构造函数，创建一个观察器实例，需要传递一个函数作为DOM改变后触发的回调函数。
```js
const observer = new MutationObserver(callback);
```

## callback
当指定的DOM元素变动时会触发该回调函数。

callback有2个参数，第一个是监听的DOM改变后返回的 mutationLIst（描述DOM节点的数组），第二个是创建的观察器实例对象。
示例如下：
```js
function callBack(mutationList, observer){}
```


# 实例方法
`MutationObserver`构造函数实例化后创建的观察器实例，拥有以下实例方法：

## observe(targetNode, configs)
- targetNode  指定监听的DOM节点
- configs  配置项，描述了DOM的某些变化才能被监听并触发callBack
	- attributeFilter ?: <string>  监听DOM节点指定属性的变更，如果没有设置该属性则默认为所有属性
	- attributeOldValue ?: boolean  当值为true时，记录DOM节点属性变更之前的值
	- attributes ?: boolean  值为true则监听DOM节点的属性变更，默认为false
	- characterData ?: boolean  值为true则监听DOM节点包含的字符变化，默认为false
	- characterDataOldValue ?: boolean 值为true则记录DOM节点字符变化之前的值
	- childList ?:boolean 值为true则监听目标节点添加或删除子节点
	- subtree ?: boolean 当值为true时，将监听的范围扩展为以目标节点为根节点的整个DOM树，即目标节点及其子孙节点
		
> 注意，在调用`observe`时，`attributes`、`characterData`、`childList`必须有一个为true。
TODO 插入例子
TODO 替换为table

## disconnect()
停止观察器对DOM节点的监听，可以调用`observe`来重启监听。

如果被观察的元素被从DOM中移除，然后被浏览器的垃圾回收机制释放，此MutationObserver将同样被删除。

## takeRecords()
返回已检测但是尚未被观察器的回调函数处理的`MutationRecord`对象列表，使变更队列保持为空。
一般用在断开观察器之前，获取所有的变更记录。

# MutationRecord 对象
当监听的DOM发生变化时，每一次变化都会生成一组`MutationRecord`对象。

- type  按照DOM的实际变化，值为`attributes`（属性变化）,值为`characterData`（字符变化）,值为`childList`（子节点变化） 
- target  实际变化的节点
- addedNodes  返回添加的节点列表
- removedNodes  返回删除的节点列表
- previousSibling  返回添加或删除之前的兄弟节点，没有则为null
- nextSibling  返回添加或删除之后的兄弟节点，没有则为null
- attributeName  返回被修改的属性名，没有则为null
- oldValue  返回改变之前的属性值（attributeOldValueweitrue）或字符数据（characterDataOldValue为true）

# 应用场景和示例
监听元素和DOM节点的变化，可以参考下面示例

```js
function queryNode(value) {
  return document.querySelector.call(document, value);
}

// * 更新attribute
queryNode(".btn-1").addEventListener("click", () => {
  const displayVal = queryNode(".child-1").style.display;

  queryNode(".child-1").style.display =
    displayVal === "none" ? "block" : "none";
});

// * 更新DOM
queryNode(".btn-2").addEventListener("click", () => {
  const parentNode = queryNode(".parent-2");

  const hasChild = !!parentNode.firstChild;
  if (hasChild) {
    parentNode.removeChild(queryNode(".child-2"));
  } else {
    parentNode.innerHTML = '<div class="child-2">测试 dom节点</div>';
  }
});

function callback(mutationRecords, mutationObserver) {
  for (let mutationRecord of mutationRecords) {
    if (mutationRecord.type === "attributes") {
      console.log("更新attributes产生的mutationRecord", mutationRecord);
    } else if (mutationRecord.type === "childList") {
      console.log("更新DOM产生的mutationRecord", mutationRecord);
    }
  }
}

const observer = new MutationObserver(callback);

observer.observe(queryNode("#root"), {
  attributes: true,
  childList: true,
  subtree: true,
  attributeOldValue: true,
  characterDataOldValue: true
});
```

[code sandbox 示例](https://codesandbox.io/s/mutationobserver-demo-v2lwz?file=/src/index.js)

# 扩展
## mutation observer 调用时机
mutation observer 调用属于微任务，在改变监听DOM节点时会将callback添加至微任务队列，执行顺序见下图

![执行顺序](http://zqfile.banzheshenghuo.com/20210405175823.png)

# 参考
[MDN MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)
[《JavaScript 标准参考教程（alpha）》阮一峰](https://javascript.ruanyifeng.com/dom/mutationobserver.html)
