---
layout: post
title: "octopress安装总结"
date: 2013-06-18 08:54
comments: true
categories: octopress
---
# 前言 #

最近学习了github的使用，开始想折腾自己的个人博客，通过在网上查找资料，最终发现octopress比较符合我的需求，博客的风格清爽，而且还具有响应式设计效果，在移动端阅读十分方便。
# 安装 #

安装步骤参考octopress官网的文档说明，见地址：[http://octopress.org/docs/setup/](http://octopress.org/docs/setup/ "octopress安装步骤")

经过我的实践证明，在安装的时候由于某些环境发生变化或者步骤省略，可能会发生一些错误。

1. 执行bundle install错误。由于未安装DevKit而无法满足依赖，请去网址[http://rubyinstaller.org/add-ons/devkit/](http://rubyinstaller.org/add-ons/devkit/ "http://rubyinstaller.org/add-ons/devkit/")下载DevKit，先安装该组件。
2. 注意版本问题。如果下载的版本与文档说明的不一致，可能会安装完毕后运行不正常。文档时针对某个版本编写的，推荐使用如下版本。 

Ruby 1.9.3-p429  DevKit-4.5.2

# 部署到github #

网站部署在octopress官网上也有文档说明，见地址：[http://octopress.org/docs/deploying/github/](http://octopress.org/docs/deploying/github/ "http://octopress.org/docs/deploying/github/") 但是在安装文档操作的时候会碰到问题，特记录下来。

**注意：** 会碰到无权限访问的问题。当执行如下命令：

rake setup_github_pages

会出现提示语句：

Enter the read/write url for your repository

(For example, 'git@github.com:your_username/your_username.github.io)

Repository url:

由于github网站发生变化，url地址不对，如果按照提示语句设置，则会再提交的时候报错。建议直接拷贝github网站上的ssh url地址，比如目前github地址是这样的格式，比如我的用户名是ywbrj042，则完整的地址如下所示：

git@github.com:ywbrj042/ywbrj042.github.com.git





