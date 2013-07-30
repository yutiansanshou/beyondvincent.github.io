---
layout: post
title: "你好！github页面"
date: 2013-07-27 14:44
comments: true
categories: github
---


{% img /images/2013/07/octopress.jpg %}


#进入写博新时代
我写博文经历了这些平台：
百度空间->[devdiv](http://www.devdiv.com/home.php?mod=space&uid=12&do=blog&view=me&from=space)>[新浪博客](http://blog.sina.com.cn/beyondvincent)->[CSDN](http://blog.csdn.net/beyondvincent)->wordpress->octopress(现在)

<!-more->

###1、发表并部署博文

```
$ rake new_post["New Post"]
$ rake generate
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source
$ rake deploy
```

###2、添加多说评论功能
####A 获取short_name
去多说网注册账号，获取站点的short_name
####B 在_config.yml文件中添加如下内容

```
# duoshuo comments
duoshuo_comments: true
duoshuo_short_name: yourname
```
####C 在source/_layouts/post.html中添加多说评论模块

```
｛% if site.duoshuo_short_name and site.duoshuo_comments == true and page.comments == true %｝
  <section>
    <h1>Comments</h1>
    <div id="comments" aria-live="polite">｛% include post/duoshuo1.html %｝</div>
  </section>
｛% endif %｝
```
####D 创建source/_includes/post/duoshuo.html，并填入如下内容

```
<!-- Duoshuo Comment BEGIN -->
<div class="ds-thread"></div>
<script type="text/javascript">
  var duoshuoQuery = {short_name:"{{ site.duoshuo_short_name }}"};
  (function() {
    var ds = document.createElement('script');
    ds.type = 'text/javascript';ds.async = true;
    ds.src = 'http://static.duoshuo.com/embed.js';
    ds.charset = 'UTF-8';
    (document.getElementsByTagName('head')[0] 
    || document.getElementsByTagName('body')[0]).appendChild(ds);
  })();
</script>
<!-- Duoshuo Comment END -->
```
####E 发布到站点

```
$ rake generate
$ git add .
$ git commit -am "添加多说评论" 
$ git push origin source
$ rake deploy
```

