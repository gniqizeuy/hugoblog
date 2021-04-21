---
title: "protobuf install"
description: "protobuf 安装"
bookToc: false
---

新版本

google protobuf-3.6.1是现在最新版本，添加了新的特性，看说明

下载地址 https://github.com/protocolbuffers/protobuf/releases

我下载的是protobuf-all-3.6.1.tar.gz 包

新版本不需要执行autogen.sh脚本，直接./configure就行


    tar zvxf protobuf-all-3.6.1.tar.gz
    cd protobuf-3.6.1
    
    ./configure –prefix=/usr/local/
    
    sudo make  # 要编译很久
    sudo make check
    sudo make install

编译安装protobuf的编译器protoc

    wget https://github.com/google/protobuf/releases/download/v3.6.1/protobuf-all-3.6.1.tar.gz
    tar zxvf protobuf-all-3.6.1.tar.gz
    ./autogen.sh
    ./configure
    make
    make install

错误处理

1、./autogen.sh执行报错./autogen.sh: line 38: autoreconf: command not found

    安装autoconf和automake
    yum -y install gcc automake autoconf libtool make

    安装g++:
    yum install gcc gcc-c++