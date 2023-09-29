# 防火墙

```shell
systemctl status firewalld # 查看防火墙状态 
systemctl start firewalld  # 开启防火墙  
systemctl stop firewalld # 关闭防火墙 
```

# 查看防火墙的开放的端口

```shell
firewall-cmd --permanent --list-ports
```

# 开放端口

```shell
firewall-cmd --add-port=3306/tcp --permanent # 添加指定需要开放的端口
firewall-cmd --reload # 重载入添加的端口
firewall-cmd --query-port=3306/tcp # 查询指定端口是否开启成功
```

# 移除指定端口

```shell
firewall-cmd --permanent --remove-port=3306/tcp
```

# windos测试端口

```shell
[root@centos7-127 ~]# telnet 192.168.31.111 3306
Trying 192.168.87.128...
Connected to 192.168.87.128.  # 表示端口已经开放
```

