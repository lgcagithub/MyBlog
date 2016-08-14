---
title: 用Hexo和Github Page搭建个人博客
date: 2016-08-14 12:35:38
tags:
- Hexo
- Github Page
- 搭建博客
---

------------------------

## 本地安装、部署、测试预览

- 下载并安装[Git](https://git-scm.com/downloads)，以及[配置ssh key](http://www.cnblogs.com/ayseeing/p/3572582.html)。当然，如果你是Github老司机的话请忽略该步。

- 下载并安装[NodeJS](https://nodejs.org/zh-cn/)。

- 打开命令行工具，输入：
``` bash
npm install -g hexo-cli
```
	这个需要一点时间，看网速。我就等了数分钟。这一步完成就可以开始使用`Hexo`了。
<!-- more -->
- 进入您想要存放博客项目的文件夹，输入以下命令初始化工作区：
``` bash
hexo init
```
	这个步骤也需要几分钟。初始化完成后，得到如下大致的目录结构：
```
node_modules
scaffolds
source
  _posts
    hello-world.md
themes
.gitignore
_config.yml
package.json
```
	若你的没有`node_modules`和`.gitignore`，也不用担心，它们不是重点。`_config.yml`是整个博客的全局配置，`source`是存放生成博客内容所需资源的地方，`source/_posts`是文章的默认存放处，`themes`是博客主题的存放点，`scaffolds`是模板的存放点，这里的模板指的是新建文章的初始内容，Hexo已经默认在此文件夹生成了`post.md` `page.md` `draft.md`这三种模板。

- 此时您的博客处于最初的状态，一切配置的默认值都由`Hexo`设置好了，并且`source`文件夹的`_post`文件夹里面有一篇名为`hello-world.md`的`markdown`文件，这就是`Hexo`为您生成的第一篇示例用的文章。对的，在`Hexo`里就是用`markdown`来写文章的。想看看您的博客最初长啥样？赶快生成内容吧！输入如下命令：
``` bash
hexo generate
```
	这一步完成后，目录里会多了个`public`文件夹，里面就是您的博客内容了。

- 生成了之后怎么看？启动`Hexo`的测试服务器吧。输入如下命令：
``` bash
hexo server
```
	`Hexo`默认指定端口是`4000`，所以打开[http://localhost:4000](http://localhost:4000)就能看到你的博客长啥样了。要是你的4000端口被占用了，可以换个端口：
``` bash
hexo server -p 8888
```

　　至此，您已经成功部署`Hexo`，并成功在本地搭建起个人博客的`Hello World`了，但也仅仅是本地能看到而已，那如何放到`Github`上让大家能看到呢？

-------------------------------

## 发布到Github Page

- `Github Page`有两种类型，一种是个人/企业主页，另一种是项目主页。前者每个`Github`账号只能有一个，后者能有无数个。这里以个人/企业主页为例，创建步骤跟平时在`Github`上创建一般的`Repository`是类似的，只是对仓库的名字有点要求，必须是`你的用户名.github.io`，如：

![仓库名字要这么写](http://obw0x5bwh.bkt.clouddn.com/hexo-and-github-page-build-blog/image/00.png)

- 修改博客文件夹根目录下的`_config.yml`，确保`deploy`部分配置填写正确，这里以`git`推送到`Github`为例：
``` yaml
deploy:
  type: git
  repo: git@github.com:lgcagithub/lgcagithub.github.io.git
```
	`type`就是部署工具的类型，这里就是`git`。`repo`就是你的`Github Page`的`repository`地址，这里填的是我的`Github Page`地址的`ssh`形式（当然，之前要[部署好ssh key](http://www.cnblogs.com/ayseeing/p/3572582.html)）。更多工具和推送类型请参考[官方文档](https://hexo.io/zh-cn/docs/deployment.html)。

- 配置完之后还没完，还要下载一个插件，用来让`Hexo`能用`git`进行推送，命令如下：
``` bash
npm install hexo-deployer-git --save
```
	其实这个命令在背后也是到Github上拉代码了，因为这个插件项目在Github上......

- 现在可以发布了，命令如下：
``` bash
hexo deploy
```
	这一步会在博客根目录创建`.deploy*`文件夹，用`git`的话就变成了`.deploy_git`，然后`hexo`会将之前`generate`出来的`public`文件夹中的内容全部拷贝到`.deploy_git`文件夹，并把它们全都提交到之前在`_config.yml`中指定的`repository`地址。

　　现在，如果你直接打开在`Github`上的博客`repository`的话，出来的还是你的项目文件列表，你必须在浏览器地址栏输入`你的用户名.github.io`才能正确打开博客。从此刻开始，你的博客将可以被世界各地的人访问啦！

--------------------------------

## 创建新文章并发布

- 新建文章的命令格式是这样子的：
``` bash
hexo new [layout] <title>
```
	`layout`是可选参数，但是`title`一定要提供，且`title`若包含空格则需要用双引号括起来。Hexo默认提供3种`layout`，我觉得这个参数叫`layout`不太合适，很容易跟`themes`里面的`layout`搞混，其实它指的是`scaffolds`文件夹内的模板`post` `page` `draft`，除此之外也可以自己在这个文件夹里新增自定义模板。这三种模板生成的文章对应的所在路径为：


|模板|路径|
|:------:|:------|
|post|source/_posts|
|page|source|
|draft|source/_drafts|

自定义模板的路径与post的一样。

- 当你想立刻开写的时候，直接就：
``` bash
hexo new "new post title"
```
	当你想创建草稿的时候，可以这样：
``` bash
hexo new draft "new draft title"
```
	创建的草稿将会存放到`source/_drafts`，这将不会被`hexo generate`进行处理。如果你想发布草稿了，可以输入：
``` bash
hexo publish draft "your draft title"
```

- `hexo`的文章都是用`markdown`来书写的，可以到[Cmd Markdown 编辑阅读器](https://www.zybuluo.com/mdeditor)来边学边用，一点也不难，而且还很有趣，妈妈再也不用担心我不会`Markdown`。比如你们现在看的这篇文章就是我边学边写的。

- 文章写完后，输入`hexo server`启动本地服务器预览，觉得OK就可以输入`hexo deploy`发布啦~

-----------------------------------

## 写在最后

　　至此，您已经知道如何安装Hexo、本地部署测试，如何将博客部署到`Github Page`，以及如何发表新文章。现在就开始您的分享之旅吧！
　　值得一提的是，部署到`Github Page`的只是你已经生成的博客内容，并不是你的博客`Hexo`工程。假如有一天你的电脑坏了或者要换电脑了，只凭`Github Page`中那点内容是还原不回整个工程的，那么你的`Markdown`原稿也就没了，所以**我建议到`Github`上再开一个普通`Repository`来存放您的博客工程**。

　　本文就先唠叨那么多。
