Nginx理论知识

Nginx适用于哪些场景：

```
静态资源服务:通过本地文件系统提供服务
反向代理服务：Nginx的强大性能、缓存、负载均衡
API服务：OpenResty
```

Nginx的优点：

```sh
高并发，高性能
可扩展性好
高可靠性
热部署（不停服务，部署新的服务）
BSD许可证(开源的)
```

Nginx的组成：

```
Nginx二进制可执行文件：由各模块源码编译出的一个文件
Nginx.conf配置文件：控制Nginx的行为
access.log访问日志：记录每一条http请求信息
error.log错误日志：定位问题
```

Nginx安装配置说明：

```sh
在 /home/dong/soft/nginx-1.19.9 目录下，执行 ./configure --help | more 能看到配置文件的安装说明：
  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --modules-path=PATH                set modules path
  --conf-path=PATH                   set nginx.conf pathname
  --error-log-path=PATH              set error log pathname
  --pid-path=PATH                    set nginx.pid pathname
  --lock-path=PATH                   set nginx.lock pathname

  --user=USER                        set non-privileged user for
                                     worker processes
  --group=GROUP                      set non-privileged group for
                                     worker processes

  --build=NAME                       set build name
  --builddir=DIR                     set build directory

  --with-select_module               enable select module
  --without-select_module            disable select module
  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module

  --with-threads                     enable thread pool support

  --with-file-aio                    enable file AIO support

  --with-http_ssl_module             enable ngx_http_ssl_module
  --with-http_v2_module              enable ngx_http_v2_module
  --with-http_realip_module          enable ngx_http_realip_module
  --with-http_addition_module        enable ngx_http_addition_module
```

​	加with是将这个加入进行编译； without 将这个模块移除

