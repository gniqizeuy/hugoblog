---
title: "vscode remote"
description: "vscode remote ssh"
bookToc: false
---

**安装 remote-ssh 插件**

**配置免密登录**

打开git bash 执行 ssh-keygen 命令

生成文件在 ~/.ssh 下

    id_rsa id_rsa.pub
    私钥    公钥

使用 ssh-copy-id 命令添加本机密钥

    ssh-copy-id <user>@<host>

**编辑 C:\User\xxx\.ssh\config**

    Host centos             # 服务器名称
        HostName xxx        # 远程服务器的 IP 地址或域名
        User xxx            # 用户名
        IdentityFile xxx    # 私钥路径