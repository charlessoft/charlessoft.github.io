---
layout: post
title: "git远程分支"
date: 2014-07-01 23:10:10 +0800
comments: true
categories: git
---

#git 远程分支命令

git 版本控制管理工具,比较强大,刚开始用的时候感觉不适应,特别是很多命令行,分支等等,感觉和svn比起来,还是svn用的比较舒服,但是用久了git,就会发觉git的强大,git采用分布式版本库做法,不需要服务器端软件,只要安装了git本地就可以运作版本控制,网上有很多git和svn对比,这里就不列举了,这里只列一下自己感受比svn好的地方.  

1.随时提交代码到本地版本库,特别适合没网络的环境.  
2.git 切换分支很方便,个人感觉比svn强大很多.  
git 可以在同一个目录下随时切换分支,svn(好像不行).  

以下列举下git一些分支命令  
`git checkout -b dev1` -->新生成分支并且切换到dev1分支  
`git push origin dev1` -->上传到远程分支  
`git checkout -b dev1 origin/dev1` -->下载远程分支  


git 常用命令  
`git init` -->初始化版本库  
`git add -u` -->增加更新的文件到版本中  
`git commit -m "log 内容"` -->提交到版本库  
`git push origin master` -->更新到远程主干中  
`git reset --hard commit_id` -->彻底回退到某个版本，本地的源码也会变为上一个版本的内容    
`git reset --mixed commit_id` -->回退到指定的commit_id,保留修改后的源码,可以使用`git add -u`和`git commit -m"xxx"`提交到版本库  
`git reset --soft commit_id` -->会退到指定的commit_id,只回退了commit信息,可以使用`git commit -m"xxx"`再次提交  
`强制提交到远程版本库`->有时候需要删除远程版本库中前几次提交,可以使用命令  
`git reset --hard commit_id`然后使用`git push origin HEAD --force`  
`git tag v0.1 `->打标签,push 到远程上需要用`git push origin tag v0.1`
`git diff --name-status xxx yyy `->对比两个日志,生成有修改的文件名
  
    
  
  
