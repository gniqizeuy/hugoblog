---
title: "ssh"
bookToc: false
---

### ssh 服务异常

ssh failed to start OpenSSH Server daemon

```shell
sshd -t
```

提示服务无法加载ssh_host_rsa_key,ssh_host_ecdsa_key 推测估计是权限问题

尝试以下方法
```shell
chmod 600 /etc/ssh/ssh_host_rsa_key
chmod 600 /etc/ssh/ssh_host_ecdsa_ke
service sshd star

# 方法2：或者尝试操作如下
chown -R root.root /var/empty/sshd
chmod 744 /var/empty/sshd
service sshd restart
```