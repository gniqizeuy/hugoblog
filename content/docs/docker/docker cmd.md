+++
title = "docker command"
description = "docker 命令"

date = "2021-04-19"
categories = [
"Docker"
]

menu = "main"

+++

### Docker 子命令分类

| 子命令分类       | 子命令                                                       |
| ---------------- | ------------------------------------------------------------ |
| Docker 环境信息  | info、version                                                |
| 容器生命周期管理 | Create、exec、kill、pause、restart、rm、run、start、stop、unpause |
| 镜像仓库命令     | login、logout、pull、push、search                            |
| 镜像管理         | build、images、import、load、rmi、save、tag、commit          |
| 容器运维操作     | attach、export、inspect、port、ps、rename、stats、top、wait、cp、diff、update |
| 容器资源管理     | volume、network                                              |
| 系统日志信息     | events、history、logs                                        |

**docker ps 命令**

docker ps 命令可以查看容器的相关信息，默认只显示只在运行的容器的信息，  
可以查看到的信息包括 

- CONTAINER ID
- NAMES
- IMAGE
- STATUS
- 容器启动后执行的 COMMAND
- 创建时间 CREATED
- 绑定的端口 PORTS

docker ps 命令最常用的功能就是查看容器的 CONTAINER ID，以便对特定容器进行操作

```shell
docker ps [OPTIONS]
```

常用的选项有 -a 和 -l。

- -a 参数查看所有容器，包括停止的容器；

- -l 选项只查看最新创建的容器，包括不在运行中的容器
