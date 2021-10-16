# nginx源代码阅读

## 1.阅读的方法

~~~ sh
根据 nginx.conf 配置文件内的worker_processes 开始
worker_cpu_affinity 00000010 00001000 00010000 10000000;(nginx.conf 绑定CPU指定的核数,0~7号cpu)
worker_rlimit_nofile 4096;(让单个进程处理更多的并发数，与umlimit -a 的数量保持一致)
event{
	worker_connections 65536; 设置单个工作进程最大并发连接数
	use epoll; 使用select、poll、epoll事件驱动，只能设置再events模块中
	accept_mutex_on; on为同一时刻一个请求轮流由work进程处理，而防止被同时唤醒所有worker，避免多个睡眠进程被唤醒的设置，默认为off，新请求会唤醒所有worker进程，此进程也称为“惊群”，因此nginx刚安装完以后要进行适当的优化，建议设置为on；
	multi_accept on;on时nginx服务器的每个工作进程可以同时接受多个新的网络连接。此指令默认为off，既默认为一个工作进程只能一次接受一个新的网络连接，打开后几个同时接受多个，建议设置为on；
}
 
~~~

