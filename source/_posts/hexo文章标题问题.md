---
title: hexo发布文章报错问题记录
date: 2021-03-10 23:31
tags: [Hexo]
categories: [hexo]
excerpt: 博客发布出错，显示can not read a block mapping entry; a multiline key may not be an implicit key at ...
---

# hexo发布文章报错问题记录
hexo g 生成博客时报错，报错原因如下：
`can not read a block mapping entry; a multiline key may not be an implicit key at ...`

原因记录：
没有严格按照 yaml 语法编辑博客头部配置

**注意：** 博客头部配置中，
1. 变量和值中间必须有空格，如`title: xxxxx`
2. 不能出现英文 `""`，用中文双引号或者尽量避免使用双引号
3. 多标签配置：`- xxx` 或者 `[xx,xx,xx]` 。 `-` 后面要加空格，按回车会自动生成 `-` 表示书写正确。
