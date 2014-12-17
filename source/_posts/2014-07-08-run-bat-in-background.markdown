---
layout: post
title: "run .bat in background"
date: 2014-07-08 16:20:58 +0800
comments: true
categories: 
---
在.bat文件同级目录，新建一个.vbe，输入内容如下
```
set ws=wscript.createobject("wscript.shell")
ws.run "start.bat /start",0
```
其中 `start.bat` 为.bat文件的文件名