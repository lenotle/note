# Centos 开放端口

```shell
systemctl start/stop/status firewalld 
# 查看开放端口
firewall-cmd --list-ports 
# 开放5672端口
firewall-cmd --zone=public --add-port=5672/tcp --permanent 
#关闭5672端口
firewall-cmd --zone=public --remove-port=5672/tcp --permanent 
```

