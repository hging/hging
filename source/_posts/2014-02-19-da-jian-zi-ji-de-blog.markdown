---
layout: post
title: "Bulit a blog by Octopress"
date: 2014-02-19 15:59:45 +0800
comments: true
categories: 
---
1、安装ruby
-------

* 使用rvm安装ruby，国内首先修改访问路径，以保证安装过程中网络不会因为大墙所中断导致安装异常：

```sh
sed -i 's!ftp.ruby-lang.org/pub/ruby!ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db
```
* 之后可以使用rvm命令安装对应版本的ruby，本文使用Ruby 2.0.0

```sh
rvm install 2.0.0
```