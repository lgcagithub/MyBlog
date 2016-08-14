---
title: 使用多说给Hexo生成的博客增加评论功能
date: 2016-08-15 00:10:04
tags:
- Hexo
- 多说
---

------------------------------

## 前言

　　文章的价值除了只是利用博客展示给他人阅读之外，更多地来自于阅读者之间的讨论。所以给自己的博客文章加入评论功能是非常必要的。然而`Hexo`只是个纯静态网站生成器，只生成浏览器可以解析的网页内容，并不涉及到服务端的东西，而自己也不想费时费力去搞一个评论功能的服务端逻辑。即使有服务端代码，`Github`也不允许你运行。所以对于像我这种纠结的人来说，使用国内的“多说”最好不过了╮(╯▽╰)╭。那为嘛不使用`Hexo`自带的`Disqus`呢？因为它的服务商在墙外呀，看着别人的博客用了`Disqus`，等了半天都加载不出来，2333333333。
　　废话不多说，立刻开始。

<!-- more -->

## 创建多说站点

　　如果是首次使用多说，你必须申请多说账号。如果是首次使用多说的插件，你必须创建多说站点。申请多说账号并登陆之后，点主页中的“我要安装”进入创建站点页面。

![创建站点页面](http://obw0x5bwh.bkt.clouddn.com/add-comments-for-blog-using-duoshuo/image/00.png)

“站点名称”随便填入一个好记的名称就行了，比如你博客的`Title`。“站点地址”就填入你博客的地址，注意分清是`http`还是`https`。“多说域名”里那个红箭头指的框框中填入的东西等下是要在博客配置中引用的。接下来的邮箱、QQ号、手机号啥的你自己搞定。

## 博客配置

　　在博客根目录下的`_config.yml`的最后面加入：
``` yml
duoshuo_shortname: lgcagithub
```
`lgcagithub`就是之前红箭头指的内容。

## 修改博客主题模板

　　找到博客根目录下的`themes/landscape/layout/_partial/article.ejs`，将里面的：
``` html
<% if (!index && post.comments && config.disqus_shortname){ %>
<section id="comments">
  <div id="disqus_thread">
    <noscript>Please enable JavaScript to view the <a href="//disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
  </div>
</section>
<% } %>
```
改为：
``` html
<% if (!index && post.comments && config.duoshuo_shortname){ %>
<section id="comments">
<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="<%= post.layout %>-<%= post.slug %>" data-title="<%= post.title %>" data-url="<%= page.permalink %>"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:'<%= config.duoshuo_shortname %>'};
  (function() {
    var ds = document.createElement('script');
    ds.type = 'text/javascript';ds.async = true;
    ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
    ds.charset = 'UTF-8';
    (document.getElementsByTagName('head')[0] 
     || document.getElementsByTagName('body')[0]).appendChild(ds);
  })();
  </script>
<!-- 多说公共JS代码 end -->
</section>
<% } %>
```
上述代码来源于[http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9) 。

　　至此，你已经完成给博客添加评论功能的工作。保存，生成，发布到`Github Page`吧。点进任意一篇文章，拉到最下面就能看到你的成果了，评论、点赞、喜欢、分享样样都行，是不是很爽~\\(≧▽≦)/~。

## 代码分析

　　上面那段代码居然有那么强大的功能，怎么做到的？让我们来分析分析。
- 首先是`<% %>`括起来的代码是ejs的逻辑代码，这里主要判断了文章评论功能是否开启以及博客配置中是否有`duoshuo_shortname`这个属性，我们刚刚就配置了。判断通过就继续格式化处理接着的代码。
- 跟着来的是`<section>`标签，这其实是跟原来`Disqus`的代码是一样的，我们改动的只是`<section>`标签对括起来的东西。
- 首先是`<div>`标签对，属性`class="ds-thread"`代表这是评论框的所在，根据官方文档的说明，属性`data-thread-key`要求赋予当前文章的ID，没ID咋办？反正就给它一点能独一无二标志的值就行了，这里填的是`post`的布局名+横杠+文章`Markdown`文件名，属性`data-title`填入的就是文章的标题咯，属性`data-url`是为了让有分页的文章能显示同样的评论内容，这里赋予了文章的永久链接。
- 接着是一段`javascript`代码，声明了一个字典，字典中的`short_name`属性引用了博客配置中的`duoshuo_shortname`，接着是一个自调用函数，该函数又创建了一个`javascript`标签，该标签引用到多说官方提供的插件js代码......所以，不是这小段代码实现了评论框，最终还是官方给力啊~官方提供的js利用配置中的shorname来得知是谁在用多说，然后所有东西都在那个js里实现了。多么好的一条龙服务。→▽→
