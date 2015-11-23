---
title: Github pages + jekyll搭建博客踩过的坑
tagline: ""
last_updated: 2015-11-23
category : 技术
layout: post
tags: [blog, jekyll, github]
---
{% include JB/setup %}

本文记录第一次用github pages + jekyll搭建博客踩过的坑。

<!-- more -->

## 选模板

建议新手不要花太多时间选模板，越fancy功能越复杂的模板，配置起来越麻烦，配置失败后越有挫败感。我一开始选的是这个[模板](http://painterlin.com/)，个性化配置的时候屡屡受挫，于是放弃，换了这个极简的模板[scribble](http://scribble.muan.co/)。

## User, Organization Pages vs  Project Pages

换成极简模板后，还是不幸采坑了。修改仓库根目录下的[about.md](https://github.com/Yinying/scribble/blob/gh-pages/about.md)后，从浏览器打开about页面，内容还是没变。等了十几分钟后再试，没生效。清除浏览器缓存，依然没生效。尝试修改_post文件夹中的文章，从浏览器查看，修改不生效。

抓狂中，我注意到[scribble模板](https://github.com/Yinying/scribble)和我之前fork的[其他模板](https://github.com/Yinying/Yinying.github.io)略有不同。首先，它的branch是gh-pages，而不是我们常见的master；其次，repository的命名就是scribble，而不是常见的**.github.io。

google后发现的确略有不同，最直接的不同是以下两点

type of pages | User, Organization Pages | User, Organization Pages
------------ | ------------- | ------------
repository命名 |username.github.io   | project name(如scribble)
访问url | http(s)://\<username>.github.io  | http(s)://\<username>.github.io/\<projectname>

这篇[github帮助文档](https://help.github.com/articles/user-organization-and-project-pages/)有更详细的介绍，另外，在本地安装部署jekyll服务时也略有差异，详见[这篇文档](https://help.github.com/articles/using-jekyll-with-pages/)。

了解这些差异后，我把repository的名字从yinying.github.io改为scribble，在浏览器通过http://yinying.github.io/scribble访问，博客能够正常访问，但是我修改过的内容依然没有生效。

我预感到在Project Pages这条路上继续走下去会很坎坷，于是放弃了，挑选了另一个极简风格的User/Organization Pages[模板](http://www.thomaszhao.cn/2015/01/08/how-do-i-build-this-jekyll-blog/)重新fork折腾。


## 本地安装jekyll

有了之前折腾的经验后，这次很快就把自己的测试文章发布了。不过最后在把_post文件夹中原作者的文章删除时又碰到问题了。fork,clone后_posts有5篇原作者的文章，我执行git rm，commit，push删除其中一篇后，删除在网页端并没有生效。看到issue中两位开智同学都反映git push后约十分钟后在网页端才能看到修改的效果。为了提高调试效率，于是我下决心在本地安装jekyll。

### 坑一：更新gem
Github Pages的帮助页面里有提到如何在[本地安装Jekyll](https://help.github.com/articles/using-jekyll-with-pages/)，如果你本机已经安装了版本足够高的ruby和gem，按照官方教程一步步操作应该会很顺利。不过我在执行`gem install bundler`这步时卡住了，google看到另一位[博主](http://mark311.github.io/jekyll/update/2014/10/12/using-github-pages.html)也遇到了同样的问题，并给出了解决方案

>Jekyll是一个Ruby程序，帮助文档推荐使用bundler来安装，我完全按照它的建议一步步来做。但在安装bundler时，执行gem install bundler后一直没有返回，添加了-V参数来打开verbose——gem install -V bundler，结果程序卡在了请求rubygem.org的某个URL上，输出错误信息：HTTP Response 302。Google了这个问题，说要更新gem的版本。我本地的gem是2.0，而当前最新的是2.4，于是果断更新了，更新是通过下载gem的tarball，然后执行ruby setup.rb来安装的。

具体步骤：

* 到[rubygems官网](ttps://rubygems.org/pages/download)下载安装包，有tar，zip等格式，随意选一个
* 解压缩到任意路径下
* cd到该路径下
* 执行`ruby setup.rb`
* 执行`gem -v`确认更新后的版本

### 坑二：网络问题

成功将gem从2.0更新到2.5后，执行`gem install bundler`仍然失败，加-V参数查看具体错误信息

```
➜ gem install -V bundler                                                                                                                                                                                                                1 ↵
HEAD https://api.rubygems.org/api/v1/dependencies
200 OK
GET https://api.rubygems.org/api/v1/dependencies?gems=bundler
200 OK
Getting SRV record failed: DNS result has no information for _rubygems._tcp.api.rubygems.org
GET https://api.rubygems.org/quick/Marshal.4.8/bundler-1.10.6.gemspec.rz
302 Moved Temporarily
ERROR:  While executing gem ... (Gem::RemoteFetcher::FetchError)
    Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://api.rubygems.org/quick/Marshal.4.8/bundler-1.10.6.gemspec.rz)
``` 

我怀疑是网络问题，于是选择美国服务器连上VPN，果然立马就能正常下载安装bundler了。

### 坑三：权限问题

跳过坑二后，执行`gem install bundler`报错

```
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory.
```
这个问题对程序员或稍微折腾过linux的同学不是问题，类似坑一中执行`ruby setup.rb`，需要admin/root权限才能执行，这时只需要在命令前加上`sudo`，即执行`sudo gem install bundler`，接下来按提示输入密码即可。

### 坑四：Liquid Exception

成功安装bundler后，安装jekyll

```
sudo gem install jekyll
```
进入clone的模板仓库路径，启动jekyll本地的server，启动后就可以在http://localhost:4000看到网页修改的效果了。

```
cd yinying.github.io
bundle exec jekyll serve
```

启动后，报错

`
 Liquid Exception: Could not find post "/myblog/2015-01-08-how-do-i-build-this-jekyll-blog" in tag 'post_url'. Make sure the post exists and the name is correct. in _posts/core-samples/2015-01-07-hello_world2.md
`



至此，终于找到删除的文章仍然能在网页端显示的原因了。推测是由于删除"/myblog/2015-01-08-how-do-i-build-this-jekyll-blog" 后，另一篇文章中的tag‘post_url'找不到本该链接到资源，抛异常了（具体为啥我没想明白）。于是，我索性把两片文章都删了，重新启动jekyll服务，在浏览器访问http://localhost:4000，看到两篇文章都删除了。接着commit,push，访问yinying.github.io，删除在网页端也生效了。
  
  
  End





