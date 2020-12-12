---
title: hexo博客备份
date: 2020-12-08 22:06
tags: [Hexo]
categories: [hexo]
excerpt: 博客备份，博客多地部署。
---

# hexo博客备份
参考链接：
[使用 Hexo-Git-Backup 插件备份你的 Hexo 博客 ](https://www.itrhx.com/2019/09/29/A53-hexo-backup/)
[使用hexo-git-backup插件备份博客源文件](https://blog.csdn.net/qq_41793001/article/details/103151182)
[hexo-git-backup](https://github.com/coneycode/hexo-git-backup)

---
利用 `hexo-git-backup` 插件进行备份

## 步骤
1. 安装备份插件
```
$ npm install hexo-git-backup@0.0.91 --save  //Hexo version = 2.x.x
$ npm install hexo-git-backup --save  //Hexo version > 3.x.x
```
2. 到 Hexo 博客根目录的 `_config.yml` 配置文件里添加以下配置：(我使用了fluid主题，所以需要在`_config.fluid.yml`中同步配置)
```
backup:
  type: git
  theme: fluid
  message: Back up my blog
  repository:
    github: git@github.com:coneycode/hexo-git-backup.git,hexo
```
参数解释：
theme：你要备份的主题名称
message：自定义提交信息
repository：仓库名，注意仓库地址后面要添加一个分支名，比如我创建了一个 hexo 分支
3. 在github仓库新建分支，分支名与上述配置对应
4. 此时github仓库中应存在 master(或main)和hexo ，在提交的时候只需在 master 中使用以下命令备份你的博客即可：
```
$ hexo backup
```
或者使用以下简写命令也可以：
```
$ hexo b
```

