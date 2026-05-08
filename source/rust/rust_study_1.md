#### rust编译介绍

> cargo new cargoCode --创建 cargoCode的项目
>
> cargo build --编译 cargoCode的项目
>
> cargo check --检查项目
>
> cargo build --release --构建cargoCode的项目（发布构建）
>
> cargo new --lib libname --创建一个库文件

> rustc --version 查看rust的版本
>
> rustup doc 打开本地的rust文档

#### rust知识点

1. 所有权机制
    - 所有权是rust中最为独特的一个功能；
    - 正是所有权的概念和相关工具的引入，rust才能够在没有垃圾回收机制的前提下保障内存安全；
2. 借用和生命周期（内存安全、线程安全）
3. 类型系统与trait
4. 突破抽象范式
5. Unsafe Rust
6. 对于rust这样的系统级编程语言来说，一个值被存储在栈上还是被存储在堆上会极大的影响到语言的行为；(所有权会涉及栈与堆)
7. 存储在栈上的是大小确定的，存储在堆上是大小不确定的；

#### rust代码管理：

> - 包（package）：一个用于构建、测试并分享单元包的Cargo功能；
> - 单元包（crate）：一个用于生成库或可执行文件的树形模块结构
> - 模块（module）及use关键字：它们被用于控制文件结构、作用域及路径的私有性；
> - 路径（path）：一种用于命名条目的方法，这些条目包括结构体、函数和模块等；

#### rust守护进程

用户级 systemd（推荐，无需 root）
普通用户可以使用 systemctl --user 管理自己的服务：

```shell
# 1. 创建用户级服务目录
mkdir -p ~/.config/systemd/user/

# 2. 创建服务文件
cat > ~/.config/systemd/user/runstDaemo.service << 'EOF'
[Unit]
Description=Rust Daemon Service

[Service]
Type=simple
ExecStart=/home/dong/win_share_file/rustCode/enterprise_system/target/release/runstDaemo
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
EOF

# 3. 启动服务（无需 sudo）
systemctl --user daemon-reload
systemctl --user start runstDaemo
systemctl --user enable runstDaemo

#停服务
systemctl --user stop runstDaemo

# 4. 查看状态
systemctl --user status runstDaemo

# 5. 查看日志
journalctl --user -u runstDaemo -f #此种方法需要特定权限

#在 ~/.config/systemd/user/rustDaemo.service 文件中加入日志打印
StandardOutput=append:/home/dong/log/rustDaemo.log
StandardError=append:/home/dong/log/rustDaemo.log

# 重载并重启
systemctl --user daemon-reload
systemctl --user restart rustDaemo
```

#### Cargo.toml文件配置

```
resolver 是 Cargo 的 依赖解析器版本 设置，有1，2，3版本
```



#### rust路线

```rust
Rust基础
  ↓
async/await 基本语法
  ↓
Tokio 入门
  ↓
spawn / join / select
  ↓
异步 IO / TCP / HTTP
  ↓
channel / Mutex / Semaphore
  ↓
axum Web 开发
  ↓
Stream / FuturesUnordered
  ↓
Send / Sync / 'static
  ↓
Future / Poll / Waker
  ↓
Pin / Unpin
  ↓
性能优化与系统设计
```

