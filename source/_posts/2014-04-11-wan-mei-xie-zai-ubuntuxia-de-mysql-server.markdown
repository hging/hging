---
layout: post
title: "完美卸载ubuntu下的mysql-server"
date: 2014-04-11 16:41:00 +0800
comments: true
categories: 
---
* **系统**: Ubuntu 13.10

* **删除 mysql**
``` sh
sudo apt-get autoremove --purge mysql-server-5.0
sudo apt-get remove mysql-server
sudo apt-get autoremove mysql-server
sudo apt-get remove mysql-common //这个很重要
```
上面的其实有一些是多余的。

* **清理残留数据**
``` sh
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
```