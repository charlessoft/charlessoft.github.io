---
layout: post
title: "jenkins_install"
date: 2014-05-07 11:57:41 +0800
comments: true
categories: 
---

#可持续集成 Jenkins

##概述


##安装手册

###1. 环境
ububtu 13.4  
###2. 软件
2.1. Jenkins  
2.2. Subversion(可选)  
2.3. Apache(可选)  
2.4. Jdk6  
2.5. Tomcat(可选)  
2.7. Python Jenkins模块  

###3. 安装步骤

####3.1. Jdk 安装和配置  
下载jdk  
jdk-6u38-ea-bin-b04-linux-i586-31_oct_2012.bin(x32)  
jdk-6u38-ea-bin-b04-linux-amd64-31_oct_2012.bin(x64)  
下载地址：https://jdk6.java.net/download.html  
进入到jdk存放目录

```
>mkdir /usr/local/java
>mv ./jdk-6u38-ea-bin-b04-linux-i586-31_oct_2012.bin /usr/local/java
>cd /usr/local/java
>chmod 755 ./jdk-6u38-ea-bin-b04-linux-i586-31_oct_2012.bin
./jdk-6u38-ea-bin-b04-linux-i586-31_oct_2012.bin
```

设置环境变量 vi /etc/profile

```
>export JAVA_HOME=/usr/local/java/jdk1.6.0_38
>export JAVA_BIN=/usr/local/java/jdk1.6.0_38/bin
>export PATH=$PATH:$JAVA_HOME/bin
>export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar export JAVA_HOME JAVA_BIN PATH CLASSPATH
```
使 /etc/profile立即生效

```
>source /etc/profile
```

####3.2. 安装 apache
3.2.1. 安装apr-1.3.6.tar.gz  
下载地址：
https://archive.apache.org/dist/apr/apr-1.3.6.tar.gz  

```
>tar zxvf apr-1.3.6.tar.gz
>cd apr-1.3.6
>./configure
>make
>make install
```

3.2.1. 安装apr-util  
下载地址:https://archive.apache.org/dist/apr/apr-util-1.3.8.tar.gz  

```
>tar zxvf apr-util-1.3.8.tar.gz
>cd apr-util-1.3.8
>./configure --with-apr=/usr/local/apr
>make
>make install
```
3.2.3. 安装apache  
下载地址:http://pkgs.fedoraproject.org/repo/pkgs/httpd/httpd-2.2.9.tar.gz/80d3754fc278338 033296f0d41ef2c04/httpd-2.2.9.tar.gz

```
>tar zxvf httpd-2.2.9.tar.gz
>cd http-2.2.9
>./configure --prefix=/usr/local/apache2.2.9 --enable-dav --enable-so --enable-maintainer-mode --with-apr=/usr/local/apr/bin/apr-1-config --with-apr-util=/usr/local/apr/bin/apu-1-config
>make
>make install
>ln -s apache2.2.9 apache
```
####3.3. 安装subversion

下载地址:https://archive.apache.org/dist/subversion/subversion-1.8.5.tar.gz

```
>apt-get install zlib1g-dev
>tar zxvf subversin-1.8.5.tar.gz
>unzip sqlite-amalgamation.zip -d subversion-1.8.5
>cd subversion-1.8.5
>mv sqlite-amalgmation-3071501 sqlite-amalgamation
>./configure --prefix=/opt/svn --with-apxs=/usr/local/apache2.2.9/bin/apxs --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr
>make
>make install
```

配置svn  
建立svn版本库目录

```
>mkdir -p /home/charles/svndata/repos
```

建立svn版本库

```
>svnadmin create /home/charles/svndata/repos
```

修改配置文件
vi /home/charles/svndata/repos/conf/svnserve.conf  
去掉 passwd-db = passwd 的注释


{% img /../images/jenkins_install/jenkins_install_pic_1.png %}  

修改 passwd 文件,增加用户和密码

{% img /../images/jenkins_install/jenkins_install_pic_2.png %}  

启动svn服务

```
svnserve -d -r /home/charles/svndata/repos
```

####3.4. 安装maven

```
cp apache-maven-3.2.1-bin.zip /usr/local cd /usr/local/
unzip apache-maven-3.2.1-bin.zip
ln -s apache-maven-3.2.1 apache-maven 配置环境变量
vi /etc/profile
export JAVA_HOME=/usr/local/ jdk1.6.0_38
export JAVA_BIN=/usr/local/ jdk1.6.0_38/bin
export M2_HOME=/usr/local/ apache-maven
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar export JAVA_HOME JAVA_BIN PATH CLASSPATH
```

立即生效配置

```
source /etc/profile
```

####3.5. 安装tomcat
下载地址:http://tomcat.apache.org/download-70.cgi#7.0.2

```
cp apache-tomcat-7.0.52.tar /usr/local/ tar zxvf apache-tomcat-7.0.52.tar
ln -s apache-tomcat-7.0.52 apache-tomcat cd apache-tomcat/bin
./startup.sh
```

{% img /../images/jenkins_install/jenkins_install_pic_3.png %}  

打开web页面 http://localhost:8080

{% img /../images/jenkins_install/jenkins_install_pic_4.png %}  

修改utf8编码
vi /etc/local/java/apache-tomcat-7.0.52/conf/server.xml

{% img /../images/jenkins_install/jenkins_install_pic_5.png %}  

####3.6. 安装jenkins  

下载地址:http://mirrors.jenkins-ci.org/war-stable/latest/jenkins.war

方法1:直接运行  
java -jar jenkins.war

方法2:复制jenkins.war到/usr/local/apache-tomcat/webapps下

```
cp jenkins.war /usr/local/apache-tomcat/webapps
```

重启tomcat
打开url http://localhost:8080/jenkins

####3.7. 安装jenkins插件

Publish Over SSH -->用来控制发布程序后执行脚本



###4. 配置

####4.1. Linux slave  
4.1.1. 安装jdk(同 3.1 Jdk安装和配置)  

4.1.2. 安装maven(同 3.4 安装maven)  

4.1.3. 安装ssh服务

```
sudo /usr/sbin/useradd -m jenkins -d /home/Jenkins echo “jenkins:jenkins” | chpasswd
su - jenkins 切换到 jenkins 用户
java --version 确保 java 安装正确
ssh-keygen 生成 ssh key 信息,按三次回车,表示把 key 存储在 /home/jenkins/.ssh/id_rsa 下,不设置密码
cd .ssh
cat id_rsa.pub > authorized_keys
chmod 700 authorized_keys
将 id_rsa(相当于 privatekey)拷贝到 jenkins master 机器上,例如/root/.jenkins/id_rsa 下
```

4.1.4 在jenkins页面增加slave
{% img /../images/jenkins_install/jenkins_install_pic_6.png %}  
增加证书  
{% img /../images/jenkins_install/jenkins_install_pic_7.png %}  
设置 publish over ssh  
{% img /../images/jenkins_install/jenkins_install_pic_8.png %}  

####4.2. windows slave  

4.2.1. 安装cygwin  

选择安装ssh服务,cygwin默认是不安装OpenSSH的,需要手动选,在Netl类别下选上OpenSSH和OpenSSL两项
{% img /../images/jenkins_install/jenkins_install_pic_9.png %}  
等到下载完并完成安装,设置环境变量,把C:/cygwin/bin;C:/cygwin/usr/bin 加 入到系统环境变量的 Path 中
