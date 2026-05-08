### redis

配置文件：

```
sudo vi /etc/redis/redis.conf
```

测试redis连接：

```
redis-cli ping
```

防火墙配置：

```
sudo firewall-cmd --permanent --add-port=6379/tcp
sudo firewall-cmd --reload
```

redis命令：

```
sudo systemctl start redis
sudo systemctl enable redis
sudo systemctl restart redis
```

