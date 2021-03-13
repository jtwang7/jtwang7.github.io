---
title: Hexo博客deploy失败(显示 errno 10054)
date: 2021-03-13 19:34
tags: hexo
categories: hexo
excerpt: Hexo博客--hexo d--提交失败，显示错误openssl ssl_read: connection was reset, errno 10054
---

# Hexo博客deploy遇到链接github失败的问题

```
git config --global https.proxy http://127.0.0.1:1080

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

