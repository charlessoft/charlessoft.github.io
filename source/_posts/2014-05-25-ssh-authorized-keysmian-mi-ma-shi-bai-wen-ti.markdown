---
layout: post
title: "authorized_keys权限导致登陆失败问题"
date: 2014-05-25 13:34:22 +0800
comments: true
categories: Linux
---

##概述
authorized_keys 是将用户的公钥,保存在登陆$HOME/.ssh/authorized_keys,公钥是一段字符串,只要把它追加在authorized_keys文件的末尾就行了,下次用户可以免密码登陆.

经过测试免密码登陆`CentOs Linux 6.5 X64位`系统与`authorized_keys`文件权限有关系


{% img /../images/ssh_authorized/ssh_authorized_key_1.png %}  
有上图看出authorized_keys 权限是`-rw-rw-r-`
即使把客户端的id_rsa.pub 保存到服务器的authorized_keys上也连接不了,需要把属性修改成`-rw-r--r--`或者`-rw------`

修改命令:

```
chmod 644 authorized_keys
或
chmod 600 authorized_keys
```

{% img /../images/ssh_authorized/ssh_authorized_key_2.png %}  
这样就可以连接上



