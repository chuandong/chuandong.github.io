linux core的设置方法

### 1. 检查当前core文件限制
首先查看系统当前的core文件大小限制：

```shell
ulimit -c
```

如果输出为0，表示系统禁用了core文件生成。

### 2. 临时启用core文件（当前shell有效）

设置当前shell会话的core文件大小为无限制：

```shell
ulimit -c unlimited
```

### 3. 永久启用core文件（系统级设置）

要让设置永久生效，需要修改配置文件：

#### 操作一：修改limits.conf

编辑/etc/security/limits.conf文件，添加以下行：

```shell
*               soft    core            unlimited
*               hard    core            unlimited
```

#### 操作二：修改profile或bashrc

在/etc/profile或~/.bashrc中添加：

```shell
ulimit -c unlimited
```

然后执行source /etc/profile或source ~/.bashrc使设置生效。

### 4. 配置core文件生成路径和命名

默认情况下，core文件会生成在程序运行的当前目录，命名为core。可以通过修改core_pattern来自定义：

查看当前配置：

```shell
cat /proc/sys/kernel/core_pattern
```

设置自定义格式（例如包含进程ID）--这步没设置，使core文件哪运行，哪生成：

```shell
echo "core.%p" > /proc/sys/kernel/core_pattern
```

要永久保存这个设置，编辑/etc/sysctl.conf文件，添加：

```shell
kernel.core_pattern = core.%p
```

然后执行sysctl -p使设置生效。