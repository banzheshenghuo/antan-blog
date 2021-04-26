---
title: 阿里云免费SSL证书申请和配置七牛云HTTPS
date: 2021-02-26 22:26:32
tags:
---
# 阿里云免费SSL证书
## 选购证书

进入[SSL证书选购页面](https://www.aliyun.com/product/security/markets/aliyun/product/cas?spm=5176.12825654.h2v3icoap.635.e9392c4aHaG07V)，点击选购证书，然后证书类型选择DV单域名证书完成选购。
![选购证书](https://zqfile.banzheshenghuo.com/20210426220512.png)

## 申请证书

进入[阿里云SSL控制台](https://yundun.console.aliyun.com/?spm=a2c6h.12873639.0.0.6a3e45f8I5eLYa&p=cas#/overview/)，选择之前的免费SSL。
![证书申请](https://zqfile.banzheshenghuo.com/20210426220729.png)

然后在证书列表找到刚创建的证书，点击证书申请，按照提示填写域名、邮箱等信息。

阿里云提供多版本SSL证书，如Tomcat、Apache、Nginx和IIS等。

# 七牛云配置HTTPS

进入[域名管理页面](https://portal.qiniu.com/cdn/domain)，点击需要配置的域名，选择HTTPS配置。
![七牛云](https://zqfile.banzheshenghuo.com/20210426210215.png)

## 本地上传证书

选择本地上传。然后将之前下载的阿里云SSL证书压缩包解压，将以`*.public.crt`和 `*.key`后缀的文件使用文本编辑器或IDE打开，将**全部内容**复制粘贴至证书内容和证书秘钥中。
![添加证书](https://zqfile.banzheshenghuo.com/20210426210403.png)
然后等待几分钟配置生效。

# 参考
[2020阿里云免费SSL证书申请方法流程](https://developer.aliyun.com/article/766913)