---

title: 从零开始的博客(一)
date: 2020-11-08 19:22
tags: hexo
categories: hexo

---
# 前言
创建博客的初衷是为了记录和总结自己的学习心得，也希望将自己的一些经历分享给同样在努力奋斗的你。本篇将带着你一步步搭建专属于你的博客网站。在学习之前，你需要具备以下前提：1. 拥有一个专属于你的GitHub 2. 电脑上需安装有Git 3. 需要安装node.js并完成相应的环境配置
<!-- more -->

# 从零开始的博客搭建(一)——GitHub+Hexo
***努力的意义就是以后的日子，放眼望去全部都是自己喜欢的人和事。***

---
创建博客的初衷是为了记录和总结自己的学习心得，也希望将自己的一些经历分享给同样在努力奋斗的你。本篇将带着你一步步搭建专属于你的博客网站。
**在学习之前，你需要具备以下前提：**

**1. 拥有一个专属于你的GitHub**
**2. 电脑上需安装有Git**
**3. 需要安装node.js并完成相应的环境配置**

## 安装Hexo
Hexo 是目前比较常用的静态博客搭建框架，除 Hexo 外，比较常用的还有 jekyll, hugo 等，可以酌情选择，本篇则以 GitHub + Hexo 搭建博客。

### 第一步：在GitHub上创建仓库
登录你的 GitHub 账号，进入该页面：

![](/img/posts_img/20201108195954111_21042.png)

进入 Repository 仓库，点击 New 创建新的仓库，转入下图页面。

![](/img/posts_img/20201108200309207_10161.png)

你需要填写仓库名称**(你的用户名+`.github.io`)**，然后点击创建 Create repository 即可。

### 第二步：安装Hexo
首先在你想要存放博客文件的位置创建一个文件夹，例如我的博客在本机中的路径为 `G:\myBlog`，打开 myBlog 文件夹，鼠标右键打开 Git Bush Here (前提是你要成功安装Git)，输入 npm 命令安装 Hexo：`npm install -g hexo-cli`。
安装完成后，输入 `hexo init` 初始化博客。
其次，输入 `hexo g` 静态部署。
至此，网页已经部署完成了，我们可以输入 `hexo s` 命令查看，此时浏览器输入 `http://localhost:4000` 就可以查看到 Hexo 的初始页面啦。

**小结：**

1. 新建文件夹，在该文件夹下右击打开 Git Bush Here
2. `npm install -g hexo-cli`
3. `hexo init`
4. `hexo g`
5. `hexo s` (可跳过)

> 啥？你问我 `hexo g` 和 `hexo s` 的具体作用？转载一下这篇博客，讲的很清楚啦，后续出现的一些命令这里也有说明：[Hexo 常用命令](https://blog.csdn.net/dxxzst/article/details/76135935)

### 第三步：将Hexo部署到GitHub
我们已经在 myBlog 中安装了 Hexo 框架，但这些仍是在本机上的一些操作，我们需要把它部署到之前创建的 GitHub 仓库上。
同样是在 myBlog 文件夹中，用笔记本的方式打开 `_config.yml` 文件。
> 该文件配置了你博客的相关的内容，关于 `_config.yml` 文件中的配置参数解释，可以参考：[hexo根目录下的_config.yml配置解释](https://blog.csdn.net/zemprogram/article/details/104288872)
> **所有参数中，冒号 `:` 后面都要带上空格。**

将 `_config.yml` 文件下拉至底部，修改 deploy (部署)配置参数，如图：

![](/img/posts_img/20201108202628168_13089.png)

其中 type 为部署的方法，repository 为部署的仓库名称(即我们此前创建的仓库clone地址)，branch 为分支(默认主支 master)
repository 仓库地址直接从 clone 复制，首先进入你创建的仓库，然后按下步操作进行：

![](/img/posts_img/20201108203039449_16182.png)

回到 myBlog 文件夹中，右击 Git Bush Here，安装 Git 部署插件，输入命令 `npm install hexo-deployer-git --save`。
然后分别输入 `hexo clean`, `hexo g`, `hexo d`。
完成上述步骤后，打开浏览器，我们就可以用 `https://your_user_name.github.io` (此处为 `jtwang7.github.io`) 代替 `http://localhost:4000` 打开博客了。

**小结：**

1. 打开 myBlog 目录中的 `_config.yml` 配置参数文件，修改 deploy 参数。
2. 在 myBlog 路径下，`npm install hexo-deployer-git --save`
3. `hexo clean` #清除缓存文件 db.json 和已生成的静态文件 public
4. `hexo g` #生成网站静态文件到默认设置的 public 文件夹(hexo generate 的缩写)
5. `hexo d` #自动生成网站静态文件，并部署到设定的仓库(hexo deploy 的缩写)

### 第四步：购买/解析域名，制作个性化的访问地址
走到上一步，我们已经可以用 `https://jtwang7.github.io` 访问博客了，但是本着对 .com 的执着，在[腾讯云](https://cloud.tencent.com/?fromSource=gwzcw.2212127.2212127.2212127&utm_medium=cpd&utm_id=gwzcw.2212127.2212127.2212127)上购买了一个 .com 的域名，具体购买过程就不细说了，按照网站提示操作就行，除了腾讯云外，也可以在[万网](https://wanwang.aliyun.com/)，[Godaddy](https://sg.godaddy.com/zh/offers/domains/godaddycom?isc=gennbacn07&countryview=1&currencyType=CNY&utm_source=baidu&utm_medium=cpc&utm_term=Title&utm_campaign=zh-cn_corp_sem_x_b_x_bz_001&utm_content=Brandzone%20PC&gclid=CIXh9LjPmecCFdOavAoddDkHcw&gclsrc=ds)上购买。
主要讲一下域名解析的一些过程，以腾讯云为例。
首先进入腾讯云控制台，点击域名注册进入域名控制台管理。

![](/img/posts_img/20201108204915553_22845.png)

第二步点击解析。

![](/img/posts_img/20201108204944145_13319.png)

第三步添加以下两条解析记录，其中 IPV4 地址可以通过ping得到，具体方法是：打开cmd输入下面命令：`ping jtwang7.github.io` #ping + 你的 GitHub 仓库地址

![](/img/posts_img/20201108205233329_15863.png)

至此，在腾讯云上的域名解析已经完成了，我们看到在记录里，有一个记录类型为 CNAME，接下来我们要打开 myBlog 文件夹的 source 文件夹，添加 CNAME 文件，可以先创建一个CNAME.txt文件，打开后写上你的域名，即你购买时申请的域名，(不要加www否则每次访问都必须加www，但如果不带有www，以后访问的时候带不带www都可以访问)，保存后记得要重命名，将.txt删除。

![](/img/posts_img/20201108205644657_21100.png)
![](/img/posts_img/20201108205702960_15023.png)

最后，回到 myBlog 文件夹下，依次输入 `hexo clean`, `hexo g`, `hexo d`。
打开 GitHub，查看 CNAME 文件是否在项目中，如图：

![](/img/posts_img/20201108210014793_7086.png)

若没有该文件，则点击 `Settings`，在 GitHub Pages 中查看域名是否保存，若域名没有自动填写，则手动将其填上并保存，则项目中会出现 CNAME 文件，若还没有 CNAME 文件，则点击 `Add file`，自行添加即可。

![]/img/posts_img/20201108210255514_19277.png)
![]/img/posts_img/20201108210359154_32401.png)
![]/img/posts_img/20201108210335057_9425.png)

现在，你可以用你自己的域名来访问博客了~

**小结：**

1. 购买域名
2. 解析域名
3. 在 myBlog/source/ 中添加 CNAME 文件
4. 回到 myBlog 文件夹，Git Bush Here
5. `hexo clean`
6. `hexo g`
7. `hexo d`
8. 检查 GitHub 是否有 CNAME 文件，若无，则在 GitHub 中添加域名


