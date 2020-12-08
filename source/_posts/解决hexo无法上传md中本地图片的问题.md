---
title: 从零开始的博客
date: 2020-11-08 22:06
tags: hexo
categories: hexo

---
# 前言
编辑完 .md 文件后，发现 .md 文件中的图片 markdown 书写方法引用的是相对路径，推送到博客上导致无法显示图片，故在网上寻找相应的解决方法，并将其记录下来。

<!-- more -->

# 解决hexo无法上传md中本地图片的问题
***努力的意义就是以后的日子，放眼望去全部都是自己喜欢的人和事。***

---
## 问题描述
编辑完 .md 文件后，发现 .md 文件中的图片 markdown 书写方法引用的是相对路径，推送到博客上导致无法显示图片，故在网上寻找相应的解决方法。
**参考链接：(大佬写的很详细，可详读)**
[hexo引用本地图片无法显示](https://blog.csdn.net/xjm850552586/article/details/84101345)

## 可能原因

1. 本地图片没有有效上传至github仓库中，导致引用无效
解决方案：安装插件(见下文解决方案)
2. 本地图片没有存放在同名文件夹中
解决方案：将需要引用的本地图片存放在与文章名相同的文件夹中
3. 图片路径出错
解决方案：打开 `_config.yml` 修改 url 配置参数，将 url 改为 github 仓库地址或者域名 (如 `jtwang7.github.io` 或 `wangjintian.com`)
4. .md 文件中图片引用相对路径没有更换
解决方法：通常我们将 .md 文件的图片存于与该文件同名的文件夹中，然后一同放到 `myBlog/source/_posts/` 中( _posts 文件夹用于存储文章及文章所包含的图片)，因此，原本 .md 文件中的图片相对路径会发生改变，需要对相应路径进行调整，同时，VNote 引用的图片不要做大小调整(即不可用=...px)！！！否则会导致加载不出来。

## 解决方法
### 第一步：安装hexo-asset-image插件
具体方法：
首先需要安装一个图片路径转换的插件，插件名为**hexo-asset-image**。
进入 myBlog 文件夹(即你的博客根目录)，右击 Git Bush Here，输入命令：
```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

### 第二步：替换hexo-asset-image插件的index.js文件内容
打开`myBlog/node_modules/hexo-asset-image/index.js`，将内容更换为下面的代码(不更改会出Bug):
```
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

var version = String(hexo.version).split('.');
hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
    	var link = data.permalink;
	if(version.length > 0 && Number(version[0]) == 3)
	   var beginPos = getPosition(link, '/', 1) + 1;
	else
	   var beginPos = getPosition(link, '/', 3) + 1;
	// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
	var endPos = link.lastIndexOf('/') + 1;
    link = link.substring(beginPos, endPos);

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];
 
      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
		if ($(this).attr('src')){
			// For windows style path, we replace '\' to '/'.
			var src = $(this).attr('src').replace('\\', '/');
			if(!/http[s]*.*|\/\/.*/.test(src) &&
			   !/^\s*\//.test(src)) {
			  // For "about" page, the first part of "src" can't be removed.
			  // In addition, to support multi-level local directory.
			  var linkArray = link.split('/').filter(function(elem){
				return elem != '';
			  });
			  var srcArray = src.split('/').filter(function(elem){
				return elem != '' && elem != '.';
			  });
			  if(srcArray.length > 1)
				srcArray.shift();
			  src = srcArray.join('/');
			  $(this).attr('src', config.root + link + src);
			  console.info&&console.info("update link as:-->"+config.root + link + src);
			}
		}else{
			console.info&&console.info("no src attr, skipped...");
			console.info&&console.info($(this));
		}
      });
      data[key] = $.html();
    }
  }
});
```

### 第三步：修改_config.yml文件配置
打开_config.yml文件，修改下述内容(可`Ctrl+F`调出查找，搜索 post_asset_folder)：
```
post_asset_folder: true
```

