---
title: 使用hexo创建博客并部署至Github
date: 2020-08-12 11:21:13
tags: hexo
---
# hexo
## 使用hexo创建博客
1. 全局安装 hexo 脚手架
`npm install hexo-cli -g  `
2. 创建hexo项目
`hexo init blog`
3. 进入项目，启动本地`server`
`cd blog && hexo server`

## 常用指令
`hexo new [layout] <filename>`
- filename: 文件名，使用引号包裹
- layout：post | draft | page
> post为正式的可见文章。
> draft为草稿文件，默认不可见。

`hexo publish [layout] <filename>`
发表草稿文件，比如上面使用`hexo new draft xx`创建的草稿文件转换成正式文件。

`hexo generate`
在hexo项目的public目录中生成静态文件。

`hexo clean`
清除缓存文件和已经生成的静态文件。

## 主题
如果你对hexo默认的主题不感兴趣，可以选择[官方推荐的主题](https://hexo.io/themes/)。
### 使用方式
一般将主题仓库`clone`至`hexo`项目的`themes`文件夹下，然后在hexo项目的`_config.yml`文件内修改`theme`为已添加的主题。

具体细节和定制可以查看主题仓库的 `README.md`。

# 部署到 GitHub Pages
## 使用Travis CI 
> Travis CI  只有对开源的repository是免费的，如果需要部署私有项目可以使用下面的一键部署或Github Action


## 一键部署
1. 安装 hexo-deployer-git
	`npm install hexo-deployer-git --save`
2. 在 _config.yml（如果有已存在的请删除）添加如下配置：
```
deploy:
  type: git
  repo: https://github.com/<username>/<project>
  # example, https://github.com/hexojs/hexojs.github.io
  branch: gh-pages
```

3. 运行 hexo clean && hexo deploy 。
4. 查看 `[username].github.io` 上的网页是否部署成功。


# 参考
- [_hexo官方文档_](https://hexo.io/zh-cn/)