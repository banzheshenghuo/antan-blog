---
title: 老项目改造（一）：使用webpack替换roadhog
date: 2020-01-21 14:44:51
tags: 
- webpack
- 优化
categories:
- 项目重构和优化
---

> 先说下基本情况，这个项目是2017年的老项目，采用了`roadhog `作为构建工具，支持零配置打包的同时提供了一系列配置参数简化webpack的配置难度。
> 但是到2020年的现在，无论是开发体验还是打包速度越来越让我无法接受，而`roadhog`最后一次`release`的时间是2018年8月，所以和组里的小伙伴谈论后决定用webpack替换roadhog。

# 卸载roadhog并安装webpack及其相关依赖

```
npm uninstall roadhog

npm install webpack webpack-cli -D

npm i webpack-dev-server webpack-merge babel-loader style-loader css-loader file-loader less-loader -D
```

# 安装babel和相关插件

> 由于`roadhog`自身依赖了`babel`，将`roadhog`卸载后需要重新添加依赖。
```
npm install babel-core babel-preset-env -D
```

添加`.babelrc`
```
{
  "presets": [["env"]]
}
```

# 创建webpack配置文件

在项目根目录下创建`webpack`配置文件，结构如下：
```
|-- src
|-- xxx
|-- webpack
        |-- webpack.common.js
        |-- webpack.dev.js
        |-- webpack.prod.js
```

`webpack.dev.js`用于日常开发，`webpack.prod.js`用于生产打包，`webpack.common.js`用来管理通用配置。


> 以下为`webpack`部分配置，可作为参考
# 通用配置

```js
// webpack.common.js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',

  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../src/index.html'),
      hash: true,
    }),
  ],
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: ['babel-loader'],
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[name]--[hash:base64:5]',
              },
            },
          },
        ],
        exclude: /(node_modules)|(src\/assets)/,
      },
      {
        test: /\.less$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[name]--[hash:base64:5]',
              },
            },
          },
          {
            loader: 'less-loader',
            options: {
              options: {
                javascriptEnabled: true,
              },
            },
          },
        ],
        exclude: /node_modules/,
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: ['file-loader'],
        exclude: /node_modules/,
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: ['file-loader'],
        exclude: /node_modules/,
      },
    ],
  },
};

```

- `clean-webpack-plugin` 用于打包成功时清空原输出目录文件。
- `html-webpack-plugin`  生成`html` 文件且适配`webpack`

# 开发配置

```js
const path = require('path');
const merge = require('webpack-merge');
const common = require('./webpack.common.js');
const webpack = require('webpack');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'inline-source-map',

  plugins: [new webpack.HotModuleReplacementPlugin()],
  output: {
    filename: '[hash].js',
    path: path.resolve(__dirname, '../dist'),
  },
  devServer: {
    contentBase: './dev/dist',
    hot: true,
  },
});

```

# 生产配置

```js
const path = require('path');
const merge = require('webpack-merge');

const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'production',
  output: {
    filename: '[hash].js',
    path: path.resolve(__dirname, '../dist'),
    publicPath: './',
    chunkFilename: '[name].[chunkhash:8].async.js',
  },
});
```

# 迁移过程中的问题
## esm和cjs混用
使用webpack打包时发现该问题，是由于混用esm和cjs导致的。具体来说就是，使用 `export default`导出，使用 `require`引入。
> 所以建议不要混合使用，只使用`esm`，还可以配合`tree shaking`减少代码体积。

```js
// 示例

// 导出模块 b.js
function Test(){
	return <div>xxx</div>
}

export default Test

// 导入模块	a.js
const Test = require('./b')

// 直接使用Test
```

从代码上来看是有问题的。此时从 `b.js`导出的Test组件是存在 `default`属性上，不能直接使用才对，可奇怪的是该项目一直保持稳定运行。

后面看了看`roadhog`依赖库和配置，发现引入了 `babel-plugin-add-module-exports`。这个插件是为了恢复`babel@5`的行为，在使用`require`引入 `export default`导出的`esm`模块时，不需要额外加上`.default`属性，就可以直接获取到导出模块。

```js
// 源码
// 导出模块
export default Test

// 以下webpack编译后代码

// 未添加 babel-plugin-add-module-exports
// __webpack_exports__ = module.exports
// Test 赋值给default属性
__webpack_exports__["default"] = (Test);


// 添加 babel-plugin-add-module-exports
// 直接导出Test
exports.default = Test;
module.exports = exports.default;
```

当然安装了 `babel-plugin-add-module-exports`后还需要在`babelrc`上配置
```js
{
  ...,
  "plugins": ["add-module-exports"]
}
```