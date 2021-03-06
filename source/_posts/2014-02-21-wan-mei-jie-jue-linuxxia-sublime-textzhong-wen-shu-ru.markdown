---
layout: post
title: "完美解决Linux下Sublime Text中文输入"
date: 2014-02-21 12:59:55 +0800
comments: true
categories: 
---
### 参考Sublime Text官方论坛的一个[回复](http://www.sublimetext.com/forum/viewtopic.php?f=3&t=7006&start=10#p41343)，通过以下方法完美解决Sublime Text 2中文输入的问题。

* **系统**：Ubuntu 13.10
* **输入法**：fcitx 4.2.8.1

<!-- more -->

__1、保存下面代码为sublime_imfix.c：__

```c
/*
sublime-imfix.c
Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
By Cjacker Huang <jianzhong.huang at i-soft.com.cn>

gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
LD_PRELOAD=./libsublime-imfix.so sublime_text
*/
#include <gtk/gtk.h>
#include <gdk/gdkx.h>
typedef GdkSegment GdkRegionBox;

struct _GdkRegion
{
  long size;
  long numRects;
  GdkRegionBox *rects;
  GdkRegionBox extents;
};

GtkIMContext *local_context;

void
gdk_region_get_clipbox (const GdkRegion *region,
            GdkRectangle    *rectangle)
{
  g_return_if_fail (region != NULL);
  g_return_if_fail (rectangle != NULL);

  rectangle->x = region->extents.x1;
  rectangle->y = region->extents.y1;
  rectangle->width = region->extents.x2 - region->extents.x1;
  rectangle->height = region->extents.y2 - region->extents.y1;
  GdkRectangle rect;
  rect.x = rectangle->x;
  rect.y = rectangle->y;
  rect.width = 0;
  rect.height = rectangle->height; 
  //The caret width is 2; 
  //Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
  if(rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
        gtk_im_context_set_cursor_location(local_context, rectangle);
  }
}

//this is needed, for example, if you input something in file dialog and return back the edit area
//context will lost, so here we set it again.

static GdkFilterReturn event_filter (GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
{
    XEvent *xev = (XEvent *)xevent;
    if(xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
       GdkWindow * win = g_object_get_data(G_OBJECT(im_context),"window");
       if(GDK_IS_WINDOW(win))
         gtk_im_context_set_client_window(im_context, win);
    }
    return GDK_FILTER_CONTINUE;
}

void gtk_im_context_set_client_window (GtkIMContext *context,
          GdkWindow    *window)
{
  GtkIMContextClass *klass;
  g_return_if_fail (GTK_IS_IM_CONTEXT (context));
  klass = GTK_IM_CONTEXT_GET_CLASS (context);
  if (klass->set_client_window)
    klass->set_client_window (context, window);

  if(!GDK_IS_WINDOW (window))
    return;
  g_object_set_data(G_OBJECT(context),"window",window);
  int width = gdk_window_get_width(window);
  int height = gdk_window_get_height(window);
  if(width != 0 && height !=0) {
    gtk_im_context_focus_in(context);
    local_context = context;
  }
  gdk_window_add_filter (window, event_filter, context); 
}
```

__2、编译动态库：__

```sh
gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
```

需要自行安装编译环境以及GTK的dev包，GTK的dev包安装方式如下：

```sh		
sudo apt-get install build-essential libgtk2.0-dev
```

__3、设置LD_PRELOAD并启动Sublime Text：__

```sh
LD_PRELOAD=./libsublime-imfix.so subl
```

为了不用每次启动Sublime Text都打如上代码，可以修改Sublime Text的启动脚本，默认路径为/usr/bin/subl，修改为如下内容，这样就可以在linux下使用Sublime Text进行中文文档编辑了：

```sh
#!/bin/bash

SUBLIME_HOME="/opt/sublime_text"
LD_LIB=$SUBLIME_HOME/libsublime-imfix.so
sh -c "LD_PRELOAD=$LD_LIB $SUBLIME_HOME/sublime_text $@"
```