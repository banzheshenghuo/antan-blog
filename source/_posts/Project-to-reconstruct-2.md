---
title: 老项目改造（二）：webpack构建优化
date: 2020-02-01 14:50:54
tags:
- webpack
- 优化
categories:
- 项目重构和优化
---
> 上次说到使用webpack替换roadhog，由于只添加了基础配置，性能方面肯定谈不上有多好，这篇文章着重谈谈如何进行优化。

# 性能优化的指标
说到性能优化，首先需要确定优化的方向，例如速度或体积。其次需要就当前维度进行分析，获取初始的数据进行分析再进行优化，最后拿优化后的数据来对比进行检验。

推荐使用`webpack`的速度分析插件（`speed-measure-webpack-plugin`）和体积分析插件（`webpack-bundle-analyzer`）。

我们先看看未优化的情况。
# 分析
![打包后有44M](https://zqfile.banzheshenghuo.com/20210523152257.png)
> webpack生成的bundle大小为44M。由于使用按需加载，生成了多个chunk，可以看到很多chunk包使用了相同的npm依赖（brace、antd等）。

![未优化的打包速度](https://zqfile.banzheshenghuo.com/20210523153204.png)
构建时间可以看到是3分43秒，耗时还是很长的。

# 体积优化

## 使用split打包公共模块

配置如下：
```js
// webpack.common.js
{
xxx, 
splitChunks: {
      minSize: 30000,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      name: true,
      cacheGroups: {
        vendors: {
          name: 'vendors',
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
        },
        commons: {
          name: 'commons',
          minChunks: 2,
          chunks: 'initial',
        },
      },
    },
}
```

在体积图中看到许多chunk引用了相同依赖，使用splitChunk对引入模块的代码进行分离，打包为公共模块，避免重复依赖。

## 使用体积更小的第三方库进行替换

老项目使用`moment`来处理日期时间戳的转换，实际上使用的场景并不多，但`moment`的大小有280kb且不支持`tree shaking`，所以我们决定干掉他。

为了尽量减少业务代码改动和以后也能方便的使用日期工具库，所以最后选择使用`Day.js`替换`moment`。
> `Day.js`几乎拥有和`moment`一样的API，而且大小只有2kb。

先安装`Day.js`，`npm install dayjs`，然后添加`resolve.alias`配置。
```js
// webpack.common.js
{
	resolve: {
    	alias: {
      		moment: 'dayjs',
    	},
  	},
}
```
> 设置别名，减少业务代码修改。在业务代码中`require`或`import`引入的`moment`实际上指向`dayjs`。

# 速度优化
## 多线程打包
使用`thread-loader`。大致原理是通过维护一个`worker pool`，每次解析模块时分配一个独立的线程，因此可以大大提高构建速度。
> `HappyPack`也是类似的。

使用很简单，只需要将`thread-loader`作为模块的首个`loader`就可以开启多线程构建。
```js
{
 module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: ['thread-loader', 'babel-loader'],
      },
    ]
  }
}
```

## 添加缓存
首次打包会添加缓存，后续打包会利用缓存资源，大大提高打包速度。
> 对于本地打包然后手动上传提升很大，后来使用`Jenkins`执行构建部署就没什么意义了。

使用方式很简单，在`babel-loader`后面添加`?cacheDirectory=true`，示例如下：
```js
{
 module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: ['thread-loader', 'babel-loader?cacheDirectory=true'],
      },
    ]
  }
}
```
设置`cacheDirectory=true`后，首次构建较平时慢一些，会将`babel`编译后产物缓存至`node_modules/.cache/babel-loader`，待下次构建会先访问缓存资源。

# 优化后的体积和耗时

![](https://zqfile.banzheshenghuo.com/20210525210649.png)
![](https://zqfile.banzheshenghuo.com/20210525211032.png)
速度从3分40秒缩短到43秒， 速度提高了80%。
体积从44M减小至6.5M，减少了88%。

# 参考
[webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)
[_speed-measure-webpack-plugin_](https://github.com/stephencookdev/speed-measure-webpack-plugin)
[_thread-loader_](https://github.com/webpack-contrib/thread-loader)
[_babel-loader_](https://github.com/babel/babel-loader)
