---
layout: post
title:	解决ubuntu上sublime text 3不能输入汉字的问题
category: other
tags: [ubuntu, sublime text，输入法，中文]
image:
  feature: article.jpg
comments: true
share: true
---
惯例，文首附参考链接：[http://bbs.chinahtml.com/t319624/](http://bbs.chinahtml.com/t319624/)


解决ubuntu上sublime text 3不能输入汉字的问题。
网络上已经有很完善的解决方案，本文针对该方案进行针对新手的补充说明

目录

* Table of Contents
{:toc}


##Step1：安装输入法

首先我们安装输入法，**依次**使用如下三个命令即可：

	sudo add-apt-repository ppa:fcitx-team/nightly
	sudo apt-get update
	sudo apt-get install fcitx fcitx-googlepinyin

关于开机自动启动的问题，并不影响最终结果，如果需要请看[参考链接](#1)


##Step2：创建一个c文件

将下面的c代码，保存为sublime-imfix.c（注意此处为**连字符**，不是下划线），保存的位置为sublime的安装目录。  
**有的朋友可能找不到安装目录，请直接打开sublime选择preferences（首选项），然后选择browse packages...然后就会打开一个文件夹，向上几步就可以到sublime的根目录。**



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

##Step3：安装C/C++的编译环境和gtk libgtk2.0-dev

直接使用如下命令：

	sudo apt-get install build-essential  
	sudo apt-get install libgtk2.0-dev

##Step4：编译成共享库

	gcc -shared -o libsublime-imfix.so sublime-imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC


**在这一步的时候，如果遇到问题，比如说出现未定义的变量，文件名，结构体等等问题，请检查之前的c代码，有时候在复制过程中会出现乱码。请重新复制或者对照修改。**


##Step5：修改启动图标
首先，cd到sublime的根目录，找到`libsublime-imfix.so`，使用命令

	mv libsublime-imfix.so /usr/lib
把`libsublime-imfix.so`移动到/usr/lib

然后使用如下命令  

	cd /usr/share/applications
	sudo vi sublime_text.desktop
进入applications修改sublime-text-2.desktop

原文作者在这里使用了vi作为文本编辑器，大家也可以用gedit作为编辑器：
	sudo gedit sublime_text.desktop

运行命令后会打开文件sublime_text.desktop，修改    
`Exec=/opt/sublime_text/sublime_text %F`  
为    
`Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /opt/sublime_text/sublime_text' %F`  

修改  
`Exec=/opt/sublime_text/sublime_text -n`  
为  
`Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /opt/sublime_text/sublime_text' -n`

至此就完成了所有的操作，注销系统再进入，就可以使用了。

<span id="jump">
参考链接：
</span>
[http://bbs.chinahtml.com/t319624/](http://bbs.chinahtml.com/t319624/)