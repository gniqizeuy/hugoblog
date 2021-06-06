---
title: "mutagen"
bookToc: false
---

[mutagen.io](https://mutagen.io/)


#### Installation

{{< tabs "uniqueid" >}}
{{< tab "Linux" >}}
`wget https://github.com/mutagen-io/mutagen/releases/download/v0.11.8/mutagen_linux_amd64_v0.11.8.tar.gz`

`tar zxvf mutagen_linux_*.tar.gz -C /usr/local/bin`

`mutagen daemon start` #启动
{{< /tab >}}
{{< tab "Windows" >}}
github issue: [issue](https://github.com/NixOS/nixpkgs/issues/81219)

download tar.gz: [amd64.v0.11.8.tar.gz](https://github.com/mutagen-io/mutagen/releases/download/v0.11.8/mutagen_windows_amd64_v0.11.8.tar.gz) ; add to path

设置 `MUTAGEN_SSH_PATH` 变量指向 SSH 可执行文件的目录，暂不支持 Windows 的 SSH 客户端

采用 git for windows 注意 git 安装地址为 C: 默认地址

将 `MUTAGEN_SSH_PATH` 指向 `GIT_INSTALL_ROOT\usr\bin`
{{< /tab >}}
{{< /tabs >}}


#### create session

`mutagen sync create`

#### list session

`mutagen sync session`

#### monitor session

`mutagen sync monitor`