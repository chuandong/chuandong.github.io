#### 1. rust编译介绍

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

#### 2.rust知识点

1. 所有权机制
    - 所有权是rust中最为独特的一个功能；
    - 正是所有权的概念和相关工具的引入，rust才能够在没有垃圾回收机制的前提下保障内存安全；
2. 借用和生命周期（内存安全、线程安全）
3. 类型系统与trait
4. 突破抽象范式
5. Unsafe Rust
6. 对于rust这样的系统级编程语言来说，一个值被存储在栈上还是被存储在堆上会极大的影响到语言的行为；(所有权会涉及栈与堆)
7. 存储在栈上的是大小确定的，存储在堆上是大小不确定的；

#### 3.rust代码管理：

> - 包（package）：一个用于构建、测试并分享单元包的Cargo功能；
> - 单元包（crate）：一个用于生成库或可执行文件的树形模块结构
> - 模块（module）及use关键字：它们被用于控制文件结构、作用域及路径的私有性；
> - 路径（path）：一种用于命名条目的方法，这些条目包括结构体、函数和模块等；