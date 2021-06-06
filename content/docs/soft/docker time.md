---
title: "docker timezone"
bookToc: false
---

**查看容器时间**

    docker exec -it xxx date

**查看本地时区文件**

    ls -l /etc/localtime
    
**拷贝本地时区文件**

    docker cp xxx xxx:/etc/localtime

