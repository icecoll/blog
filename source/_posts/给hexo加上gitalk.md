---
title: 给hexo加上gitalk
date: 2018-08-06 14:41:14
tags: "hexo"
---

### Step 1. Register a new OAuth application
传送门: [Goooooooooooooooooooo](https://github.com/settings/applications/new)

示例:
```
Application name
blog

Homepage URL
https://blog.lianming.tk

Application description
lianming's blog.

Authorization callback URL
https://blog.lianming.tk
```

### Step 2. Config

主题的配置中添加: 
```
# themes/jacman/_config.yml
gitalk:
  enable: true
  clientID: ''
  clientSecret: ''
  repo: 'blog.lianming.tk' #写一个自己的github repo名称,比如我直接用自己的gitpage repo
  owner: 'icecoll'         #自己的github user name
  admin: ['icecoll']
  distractionFreeMode: 'true
```

### Step 3. Add gitalk code

把gitalk相关的代码加入到页面文件中
```ejs
# themes/jacman/layout/_partial/post/comment.ejs
<% if (theme.gitalk && theme.gitalk.enable){ %>
  <section id="gitalk-container" class="comment"></section>
  <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
  <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
  <script>
    var gitalk = new Gitalk({
      clientID:  '<%= theme.gitalk.clientID %>',
      clientSecret: '<%= theme.gitalk.clientSecret %>',
      repo: '<%= theme.gitalk.repo %>',
      id: window.location.pathname,
      owner: '<%= theme.gitalk.owner %>',
      admin: '<%= theme.gitalk.admin %>',
      distractionFreeMode: '<%= theme.gitalk.distractionFreeMode %>',
    });
    gitalk.render('gitalk-container');
  </script>
<% } %>

```



