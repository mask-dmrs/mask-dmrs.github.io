---
title: Hexo集成gitment异常处理
tags:
  - 插件
  - gitment
  - Hexo
  - 规范开发
categories: Hexo
permalink: hexo-integrated-gitment-exception-handling
date: 2017-11-09 00:27:57
---
#### 问题
集成`Gitment`按照官方的文档配置报错`Cannot read property 'firstChild' of null`
#### 查找
-  通过页面的异常进入js发现是获取某个对象为空`gitment-container`
-  讲这个关键词在插件里面搜索发现,这个一个装载留言内容的div,于是在异常的页面查看源码搜索,没有找到这个div.
-  通过查看源码发现`themes\next\layout\_third-party\comments`是一个留言的插件集合,包含多种留言插件`js`文件
```javascript
{% include 'duoshuo.swig' %}
{% include 'disqus.swig' %}
{% include 'hypercomments.swig' %}
{% include 'youyan.swig' %}
{% include 'livere.swig' %}
{% include 'changyan.swig' %}
{% include 'gitment.swig' %}
{% include 'valine.swig' %}
```
`themes\next\layout\_partials`是留言插件`html`文件`comments.swig`
有一段代码引起我的注意
```javascript
{% elseif theme.changyan.appid and theme.changyan.appkey %}
    <div class="comments" id="comments">
      <div id="SOHUCS"></div>
    </div>

  {% elseif theme.gitment.enable %}
    <div class="comments" id="comments">
      {% if theme.gitment.lazy %}
        <div onclick="showGitment()" id="gitment-display-button">{{ __('gitmentbutton') }}</div>
        <div id="gitment-container" style="display:none"></div>
      {% else %}
        <div id="gitment-container"></div>
      {% endif %}
    </div>
```
对照`themes\next\_config.yml`文件
```yml
changyan:
  enable: false
  appid: XX
  appkey: XX
gitment:
  enable: true
  # RECOMMEND, A mint on Gitment, to support count, language and proxy_gateway
  mint: false 
```
要命的是之前是用畅言测试过,但是这里不是根据`changyan.enable`去判断,于是导致`ifelse`进入了畅言这个留言插件的div判断.
#### 解决
>清空`themes\next\_config.yml`文件中的`changyan.appid`和`changyan.appkey`,
>`themes\next\layout\_partials\comments.swig`文中,`theme.changyan.appid and theme.changyan.appkey`修改为`theme.changyan.enable`

#### 总结
- 规范开发
- 善于关键词查找分析解决问题
