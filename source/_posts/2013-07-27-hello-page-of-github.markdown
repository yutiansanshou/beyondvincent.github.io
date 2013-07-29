---
layout: post
title: "你好！github页面"
date: 2013-07-27 14:44
comments: true
categories: github
---
{% img center /images/2013/07/octopress.jpg %}

###1、发表并部署博文

```
$ rake new_post["New Post"]
$ rake generate
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source
$ rake deploy
```


