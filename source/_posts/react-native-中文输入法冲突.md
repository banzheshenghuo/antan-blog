---
title: react-native 中文输入法冲突
date: 2018-10-22 20:30:10
tags: react-native
---
最近一直在做RN开发，遇到了不少坑，昨天碰到一个中文输入冲突。

## 场景
我用的RN版本是：0.54.0，先上代码,最常用 最简单的写法

``` js
<TextInput
    defaultValue={this.state.text}
    onChangeText={(text) => {
      this.setState({ text });
  }}
/>
```

通过onChangeText 获取到输入值，然后赋值给state，组件重新render，页面重绘，嗯~~ web端常用的方法。
但是RN上切换中文输入法，发现无法输入中文。因为每输入一个字母，没等到输入中文，页面就开始重绘。(IOS端会出现此问题)

## 使用onEndEditing代替 onChangeText
翻阅RN文档发现textInput 有 onEndEditing 方法，文档解释该方法为 当文本输入结束后调用此回调函数。
修改后的代码

``` js
<TextInput
    defaultValue={this.state.text}
    onEndEditing={(evt) => {
      this.setState({ text:evt.nativeEvent.text });
    }}
/>
```
