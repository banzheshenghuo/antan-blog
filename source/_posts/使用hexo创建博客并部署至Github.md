---
title: 使用hexo创建博客并部署至Github
date: 2020-08-12 11:21:13
tags: hexo
categories: 工具
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

# 将hexo项目推送至Github
在Github上创建仓库并推送，仓库名可以自定义，例如：`my-blog`

# 创建Github Pages（已经创建可以忽略这部分内容）
不了解Github Pages的可以看看[官方介绍](https://docs.github.com/cn/pages)。

简单来说就是依托于Github创建个人的静态站点。

首先为站点创建仓库，仓库名必须是`<userName>.github.io`，[参考文档](https://docs.github.com/cn/pages/getting-started-with-github-pages/creating-a-github-pages-site)
> 仓库必须选择public

也可以将博客项目部署到其他项目（Project page），访问地址是`https://<userName>.github.io/[projectName]`。

# 部署到 GitHub Pages
## Github Actions 部署
简单的说 Github Actions 是 Github 提供的持续集成服务，可以帮助我们抓取代码、运行测试、登录远程服务器，发布到第三方服务等等。

这里使用 Github Actions 来监听maste的push事件，创建静态文件并推送至远程的操作。

1. 首先检查自己`package.json`文件是否有hexo的打包指令，没有就加上。
```json
{
  //xxx
  "scripts": {
    "build": "hexo generate", // 没有就加上
    "clean": "hexo clean",
    "server": "hexo server"
  },
  //xx
}
```
2. 在hexo项目的根目录添加 `.github/workflows/deploy.yml`文件，内容如下：
```yml
name: deploy to Github Pages # 工作流 name

on:            # 触发条件
  push:        # push事件
    branches:  # 指定分支
      master   # master分支

jobs:          # 任务，一个workflow可以有多个job
  deploy:      # 任务名为 deploy
    runs-on: ubuntu-latest  # 运行环境
    steps: 
      - uses: actions/checkout@v2
      - name: Use Node.js 12.x
        uses: actions/setup-node@v1
        with:
          node-version: '12.x'
      - name: Cache NPM dependencies  
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies  # 安装hexo依赖
        run: npm install
      - name: Build   # 执行打包命令，生成静态文件
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public  # 静态文件的资源路径
          publish_branch: gh-pages  # 部署的分支
```

3. 将修改内容推送至远程仓库，然后修改 GitHub Pages 的部署分支为 gh-pages

## 一键部署
1. 安装 `hexo-deployer-git`
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
> 在GitHub 中对需要部署的 repository 进行设置，修改 GitHub Pages 的部署分支为 gh-pages。

## 使用Travis CI 
> Travis CI  只有对开源的repository是免费的

# 问题和注意事项
## 设置Github Pages
打开需要部署的项目的Settings，找到 GitHub Pages 然后将`gh-pages`设为默认源。
![设置源](https://zqfile.banzheshenghuo.com/20210427155505.png)
> 注意：GitHub Pages 免费版只能用在 public 项目上，如果想在 private 使用需要升级GitHub Pages服务。


##  一键部署中出现以下报错
fatal: unable to access `'https://github.com/[githubname]/[projectname]/'`: LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to `github.com:443`
```
FATAL {
  err: Error: Spawn failed
      at ChildProcess.<anonymous> (blog/node_modules/hexo-util/lib/spawn.js:51:21)
      at ChildProcess.emit (events.js:315:20)
      at Process.ChildProcess._handle.onexit (internal/child_process.js:277:12) {
    code: 128
  }
}
```
原因和方案：网络问题，翻墙即可。

## 样式图片资源未加载
在`_config.yml`中修改 `url`为预览地址。
![修改url](https://zqfile.banzheshenghuo.com/20210427154811.png)

# 参考
- [_hexo官方文档_](https://hexo.io/zh-cn/)
- [_Github Pages_](https://docs.github.com/cn/pages/getting-started-with-github-pages)