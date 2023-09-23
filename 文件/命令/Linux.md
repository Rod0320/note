```shell
查看防火墙状态 systemctl status firewalld
开启防火墙 systemctl start firewalld  
关闭防火墙 systemctl stop firewalld
开启防火墙 service firewalld start 
若遇到无法开启
先用：systemctl unmask firewalld.service 
然后：systemctl start firewalld.service

添加指定需要开放的端口：firewall-cmd --add-port=8090/tcp --permanent
重载入添加的端口：firewall-cmd --reload
查询指定端口是否开启成功：firewall-cmd --query-port=8099/tcp
移除指定端口：firewall-cmd --permanent --remove-port=123/tcp
查看防火墙的开放的端口: firewall-cmd --permanent --list-ports

telnet ip 端口号
[root@centos7-127 ~]# telnet 192.168.87.128 22
Trying 192.168.87.128...
Connected to 192.168.87.128.  # 表示端口已经开放
```

