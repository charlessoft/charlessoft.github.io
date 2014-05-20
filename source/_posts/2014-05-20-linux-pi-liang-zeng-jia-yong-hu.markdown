---
layout: post
title: "linux 批量增加用户"
date: 2014-05-20 23:10:05 +0800
comments: true
categories: linux
---
##概述
linux下经常需要增加用户进行执行脚本,下文介绍使用shell脚本增加用户,可以实现批量增加用户.
<!--more-->

##步骤
###相关命令
1. newusers --新建用户
2. chpasswd --修改用户密码

建立一个和/etc/passwd 一样的文本内容userlist.txt
```
ubuntu00:x:520:520::/home/ubuntu00:/bin/bash
ubuntu01:x:521:521::/home/ubuntu01:/bin/bash
ubuntu02:x:522:522::/home/ubuntu02:/bin/bash
ubuntu03:x:523:523::/home/ubuntu03:/bin/bash
ubuntu04:x:524:524::/home/ubuntu04:/bin/bash
ubuntu05:x:525:525::/home/ubuntu05:/bin/bash
ubuntu06:x:526:526::/home/ubuntu06:/bin/bash
ubuntu07:x:527:527::/home/ubuntu07:/bin/bash
ubuntu08:x:528:528::/home/ubuntu08:/bin/bash
ubuntu09:x:529:529::/home/ubuntu09:/bin/bash

```

建立一个password.txt 文件,保存用户名对应的密码,格式如下
```
ubuntu00:123456
ubuntu01:123456
ubuntu02:123456
ubuntu03:123456
ubuntu04:123456
ubuntu05:123456
ubuntu06:123456
ubuntu07:123456
ubuntu08:123456
ubuntu09:123456
```

###使用命令
```
newusers userlist.txt
chpasswd < password.txt
```
就添加完成了,su - ubuntu00 输入密码123456,就切换到该用户下,可以输入bash,使用bash.



