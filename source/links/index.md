---
title: links
date: 2020-11-09 20:12:19
type: links
---
<style>.links-content{margin-top:1rem}.link-navigation::after{content:" ";display:block;clear:both}.card{width:130px;font-size:1rem;padding:0;border-radius:4px;transition-duration:.15s;margin-bottom:1rem;display:block;float:left;box-shadow:0 2px 6px 0 rgba(0,0,0,.12);background:#f5f5f5}.card{margin-left:16px}@media(max-width:567px){.card{margin-left:16px;width:calc((100% - 16px)/2)}.card:nth-child(2n+1){margin-left:0}.card:not(:nth-child(2n+1)){margin-left:16px}}@media(min-width:567px){.card{margin-left:16px;width:calc((100% - 32px)/3)}.card:nth-child(3n+1){margin-left:0}.card:not(:nth-child(3n+1)){margin-left:16px}}@media(min-width:768px){.card{margin-left:16px;width:calc((100% - 48px)/4)}.card:nth-child(4n+1){margin-left:0}.card:not(:nth-child(4n+1)){margin-left:16px}}@media(min-width:1200px){.card{margin-left:16px;width:calc((100% - 64px)/5)}.card:nth-child(5n+1){margin-left:0}.card:not(:nth-child(5n+1)){margin-left:16px}}.card:hover{transform:scale(1.1);box-shadow:0 2px 6px 0 rgba(0,0,0,.12),0 0 6px 0 rgba(0,0,0,.04)}.card .thumb{width:100%;height:0;padding-bottom:100%;background-size:100% 100%!important}.posts-expand .post-body img{margin:0;padding:0;border:0}.card .card-header{display:block;text-align:center;padding:1rem .25rem;font-weight:500;color:#333;white-space:normal}.card .card-header a{font-style:normal;color:#2bbc8a;font-weight:700;text-decoration:none;border:0}.card .card-header a:hover{color:#d480aa;text-decoration:none;border:0}</style><div><div class="links-content"><div class="link-navigation" id="links1"></div></div></div>

------

<div style="text-align:center;"><span class="with-love" id="animate1">
    <i class="fa fa-heart"></i>
  </span>留言添加友链<span class="with-love" id="animate2">
    <i class="fa fa-heart"></i>
  </span></div>
  
------

{% note success %}

## 友链格式

- 网站名称：王锦添的博客
- 网站地址：[https://wangjintian.com](https://wangjintian.com)
- 网站描述：努力的意义就是以后的日子，放眼望去全部都是自己喜欢的人和事。
- 网站Logo/头像：[https://wangjintian.com/images/avatar.webp](https://wangjintian.com/images/avatar.webp)

{% endnote %}

link = {
    init: function() {
        var that = this;
        //这里设置的是刚才的 linklist.json 文件路径
        $.getJSON("/links/linklist.json",
        function(data) {
            that.render(data);
        });
    },
    render: function(data) {
        var html, nickname, avatar, site, li = "";
        for (var i = 0; i < data.length; i++) {
            nickname = data[i].nickname;
            avatar = data[i].avatar;
            site = data[i].site;
            li += '<div class="card">' + '<a href="' + site + '" target="_blank">' + '<div class="thumb" style="background: url( ' + avatar + ');">' + '</div>' + '</a>' + '<div class="card-header">' + '<div><a href="' + site + '" target="_blank">' + nickname + '</a></div>' + '</div>' + '</div>';
  
        }
        $(".link-navigation").append(li);
    }
  }
  link.init();