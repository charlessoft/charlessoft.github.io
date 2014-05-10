---
layout: post
title: "V8 Engine编译"
date: 2014-05-10 15:32:28 +0800
comments: true
categories: V8
---

#概述
V8 Engine 用来编译javascript, chrome浏览器就是使用该引擎,V8 引擎是独立的模块,可以直接嵌入到程序中执行js脚本

<!--more-->

##windows 编译

###1. 工具

>V8  

>svn checkout http://v8.googlecode.com/svn/branches/bleeding_edge/ v8

>python  

>svn co http://src.chromium.org/svn/trunk/tools/thirdparty/python26@89111 thirdparty/python26

>cygwin  

>svn co http://src.chromium.org/svn/trunk/deps/thirdparty/cygwin@231940 thirdparty/cygwin

>icu  

>svn co https://src.chromium.org/chrome/trunk/deps/thirdparty/icu46 thirdparty/icu

>gyp  

>svn co http://gyp.googlecode.com/svn/trunk build/gyp




###2. 编译步骤

>把下载后的third_party拷贝到v8目录下,设置python的环境变量,方便直接调用python命令

>cd 到v8目录下,执行命令python build\gyp_v8 生成工程文件

>"c:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\devenv.com" /build Release build\All.sln


###3. 调试步骤 

>使用vs2010打开All.sln编译好的工程,
>可以进行调试,设置启动项为 sample/shell,运行shell print('hello world!');

###4. 参考资料

>[关于V8 JavaScript Engine的使用方法研究(一)](http://blog.chinaunix.net/uid-8272118-id-2033359.html)   

>[如何在程序中嵌入google的V8 Javascript引擎](http://www.cppblog.com/weiym/archive/2012/05/19/175374.html)  

>[编译v8引擎](http://blog.csdn.net/zengraoli/article/details/9178219)  

>[BuildingWithGYP](https://code.google.com/p/v8/wiki/BuildingWithGYP)  

>[V8系列——内存管理(1)](http://hi.baidu.com/hycjk/item/f137d2e5616e64b52f140bd7)  

>[V8系列——内存管理](http://hi.baidu.com/hycjk/item/c79470d1f35fac95260ae7da)  
