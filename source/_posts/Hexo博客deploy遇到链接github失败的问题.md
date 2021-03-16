---
title: Hexo博客提交失败
date: 2021-03-13 19:34
tags: hexo
categories: hexo
excerpt: Hexo博客提交失败，显示错误errno10054
---

# Hexo博客deploy遇到链接github失败的问题

```
git config --global http.proxy http://127.0.0.1:1080

git config --global https.proxy https://127.0.0.1:1080

git config --global --unset http.proxy

git config --global --unset https.proxy
```

执行一遍上述代码
接着执行如下代码

```
hexo clean
hexo g
hexo d
```

完成！


## 2021/3/16 更新
通过DevSidecar插件代理git：
从gitee下载DevSidecar: [DevSidecar](https://gitee.com/docmirror/dev-sidecar/releases)
安装插件
启动所有服务

记住你的代理端口：
加速服务 - 基本设置 - 代理端口

git 下运行下述两行代码：记得将代理端口(我为1181)换为你的代理端口
git config --global http.proxy http://127.0.0.1:1181

git config --global https.proxy https://127.0.0.1:1181

然后就可快乐的 git push 了