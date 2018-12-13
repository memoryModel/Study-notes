# tomcat启动

## 初次启动

> 1、进入tomcat的bin目录下
>
>
>
> Last login: Mon Dec  5 11:05:02 on ttys000
>
> cxhdeMacBook-Air:~ cxh$ cd /Users/cxh/Documents/apache-tomcat-7.0.73/bin
>
> 2、输入pwd，回车
>
> cxhdeMacBook-Air:bin cxh$ pwd
>
> /Users/cxh/Documents/apache-tomcat-7.0.73/bin
>
> 3、输入sudo chmod 755 *.sh，回车
>
> cxhdeMacBook-Air:bin cxh$ sudo chmod 755 *.sh
>
> 4、输入sudo sh ./startup.sh，回车
>
> cxhdeMacBook-Air:bin cxh$ sudo sh ./startup.sh

## 参数说明

> - *sudo为系统超级管理员权限.*
> - *chmod 改变一个或多个文件的存取模式*
> - *755代表用户对该文件拥有读、写、执行的权限，同组的其他人员拥有执行和读的权限，没有写的权限，其它用户的权限和同组人员一样.*
> - *777代表，user,group ,others ,都有读写和可执行权限.*
> - *chmod -R 777 folername,获取文件夹权限.*

## 整体终端界面如下：

> Last login: Mon Dec  5 11:05:02 on ttys000
>
> cxhdeMacBook-Air:~ caoxiaohong$ cd /Users/cxh/Documents/apache-tomcat-7.0.73/bin
>
> cxhdeMacBook-Air:bincxh$ pwd
>
> /Users/cxh/Documents/apache-tomcat-7.0.73/bin
>
> cxhdeMacBook-Air:bincxh$ sudo chmod 755 *.sh
>
> cxhdeMacBook-Air:bincxh$ sudo sh ./startup.sh
>
> Using CATALINA_BASE:   /Users/cxh/Documents/apache-tomcat-7.0.73
>
> Using CATALINA_HOME:   /Users/cxh/Documents/apache-tomcat-7.0.73
>
> Using CATALINA_TMPDIR: /Users/cxh/Documents/apache-tomcat-7.0.73/temp
>
> Using JRE_HOME:        /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
>
> Using CLASSPATH:       /Users/cxh/Documents/apache-tomcat-7.0.73/bin/bootstrap.jar:/Users/cxh/Documents/apache-tomcat-7.0.73/bin/tomcat-juli.jar
>
> Tomcat started.
>
> cxhdeMacBook-Air:bincxh$ 

## 说明：

> 当第一个打开成功后，以后再打开就可以操作2步即可启动tomcat。
>
> 1、打开startup.sh目录。终端输入：cd /Users/cxh/Documents/apache-tomcat-7.0.73/bin
>
> cxhdeMacBook-Air:~ cxh$ cd /Users/cxh/Documents/apache-tomcat-7.0.73/bin
>
> 2、执行startup.sh批处理文件。终端输入：sudo ./startup.sh
>
> cxhdeMacBook-Air:bin cxh$ sudo ./startup.sh