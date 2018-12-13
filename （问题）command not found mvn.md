

# 问题：command not found: mvn



## 问题描述

> 在Mac下一般配置了Maven的环境变了一般都不会提示，但是如果使用zsh的扩展之后，系统默认的环境变量配置文件会发生变化，尤其使用Eclipse打开终端时，默认不会去读取用户目录下的~/.bashrc文件，那么就必须在~/.zshrc文件下再次添加环境变量。

## 解决方案

> 打开~/.zshrc文件，添加如下环境变量即可：

``` xml
export MAVEN_HOME=/usr/local/maven3
export PATH=${PATH}:${MAVEN_HOME}/bin
```

## 最终效果

![img](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180827161039453.png)



[原文地址](http://www.cnblogs.com/EasonJim/p/7382072.html)

