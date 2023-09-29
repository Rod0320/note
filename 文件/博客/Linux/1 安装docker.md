# 查询可用版本

```shell
[root@rod bin]# yum list -y docker*
可安装的软件包
docker.x86_64                                     2:1.13.1-209.git7d71120.el7.centos          
```

# 安装docker

```shell
yum -y install docker.x86_64
```

# 查询版本

```shell
[root@rod bin]# docker -v
Docker version 1.13.1, build 7d71120/1.13.1
```

# 设置docker服务开机自启动

```shell
# docker 服务开机自启动命令
systemctl enable docker.service

# 关闭docker 服务开机自启动命令
systemctl disable docker.service
```

- 重启生效

```shell
reboot
```

