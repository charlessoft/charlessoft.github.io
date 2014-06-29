---
layout: post
title: "jenkins_plugin_邮件通知"
date: 2014-06-03 00:05:29 +0800
comments: true
categories: ci
---
##概述
使用jenkins持续编译程序后,可以发送编译的结果给特定的人员,以便邮件方式告知编译或测试结果
jenkins 默认有自带了邮件通知功能,但是使用起来不是很方便,本文使用Email-ext plugin进行发送邮件

##插件名称:  
**Email-ext plugin**  

如果手工安装插件还需要安装 Token macro Plugin

安装好插件后在jenkins->系统管理->系统配置里头就会看到`Extended E-email Notification`选项
设置以下选项.就可以发送邮件

{% img /../images/jenkins_plugin_email/jenkins_plugin_email_pic_1.png %}

##使用插件
在项目配置中选择`增加构建后操作步骤`->`E-email Notification`
{% img /../images/jenkins_plugin_email/jenkins_plugin_email_pic_2.png %}
当项目编译后.就会发送邮件到指定的用户

##指定邮件模板
```

```

对邮件中字段详细说明待续再更新--

