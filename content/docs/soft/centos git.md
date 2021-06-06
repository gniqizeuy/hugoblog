---
title: "centos update git"
description: "更新 git"
bookToc: false
---

系统自带 git 版本较低

卸载旧版本的Git

    yum remove git -y

要升级Git，首先要安装一些依赖包；
安装基本的依赖包

    yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel nss  gcc perl-ExtUtils-MakeMaker -y

下载最新的git源码包：

    https://mirrors.edge.kernel.org/pub/software/scm/git/

解压、编译安装：

    wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.xz 

    tar xf git-2.9.5.tar.xz 
    cd git-2.9.5 
    ./configure --prefix=/usr/local/git 
    make && make install

配置环境变量：

    export PATH=$PATH:/usr/local/git/bin 
    source /etc/profile

查看版本

    git version 
    git version 2.9.5

> 注意：但是如果缺少了nss包的话，在拉取以https开头的URL地址时就会有以下的几个异常信息；

    fatal: Unable to find remote helper for ‘https’
    fatal: unable to access ‘https://github.com/fatih/vim-go.git/‘: SSL connect error

可以执行安装:

    yum install nss -y

