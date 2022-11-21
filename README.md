# For-Learn-Rust

Rust 是一门系统级编程语言，被设计为保证内存和线程安全，并防止段错误。作为系统级编程语言，它的基本理念是 “零开销抽象”。理论上来说，它的速度与C 或者 C++同级。

Rust 可以被归为通用的、多范式、编译型的编程语言，类似 C 或者 C++。与这两门编程语言不同的是，Rust 是线程安全的！

Rust 编程语言的目标是，创建一个高度安全和并发的软件系统。它强调安全性、并发和内存控制。尽管 Rust 借用了 C 和 C++ 的语法，它不允许空指针和悬挂指针，二者是 C 和 C++ 中系统崩溃、内存泄露和不安全代码的根源。

Rust中有诸如 if else和循环语句 for 和 while 的通用控制结构。和 C 和 C++ 风格的编程语言一样，代码段放在花括号中。

Rust 使用实现（implementation）、特征（trait）和结构化类型（structured type）而不是类（class）。这点，与基于继承的OO语言 C++, Java 有相当大的差异。而跟 Ocaml, Haskell 这类函数式语言更加接近。

Rust 做到了内存安全而无需 `.NET` 和 `Java` 编程语言中实现自动垃圾收集器的开销，这是通过所有权/借用机制、生命周期、以及类型系统来达到的。

下面是一个代码片段的例子，经典的 Hello Rust 应用：

```
fn main() {
  println!("hello, Rust");
}
```

影响了 Rust 的流行的编程语言包括 C, C++, C#, Erlang, Haskell, OCaml, Ruby, Scheme 和 Swift 等等。 Rust也影响了C# 7, Elm, Idris, Swift。

Rust 提供了安装程序，你只需要从官网下载并在相应的操作系统上运行安装程序。安装程序支持 Windows、Mac 和 Linux（通过脚本）上的32位和64位 CPU 体系架构，适用 Apache License 2.0 或者 MIT Licenses。

Rust 运行在以下操作系统上：Linux, OS X, Windows, FreeBSD, Android, iOS。

简单提一下 Rust 的历史。Rust 最早是 Mozilla 雇员 Graydon Hoare 的一个个人项目，从 2009 年开始，得到了 Mozilla 研究院的支助，2010 年项目对外公布。2010 ～2011 年间实现的自举。从此以后，Rust 经历了巨大的设计变化和反复（历程极其艰辛），终于在 2015 年 5 月 15日发布了 1.0 版。在这个研发过程中，Rust 建立了一个强大活跃的社区，形成了一整套完善稳定的项目贡献机制（这是真正的可怕之处）。

Rust 现在由 [Rust项目开发者社区维护](https://github.com/rust-lang/rust)。

自 15 年 5月1.0发布以来，涌现了大量优秀的项目（可以 github 上搜索 Rust 查找）, 大公司也逐渐积极参与 Rust 的应用开发，以及回馈开源社区。

#### Rust 安装 

Rust 支持主流的操作系统，Linux，Mac 和 windows, 这里我用的是Mac OS系统，以Mac OS系统为例进行安装。

Rust 为 mac 用户提供了两种安装方式：

1. 直接下载安装包：

直接下载安装包的话需要检查一下你当前操作系统是64位还是32位，分别下载对应的安装包。查看操作系统请在终端执行如下命令:

直接下载安装包的话需要检查一下你当前操作系统是64位还是32位，分别下载对应的安装包。查看操作系统请在终端执行如下命令:

```bash
> uname -a
Darwin MacBook-Pro.local 19.6.0 Darwin Kernel Version 19.6.0: Tue Nov 10 00:10:30 PST 2020; root:xnu-6153.141.10~1/RELEASE_X86_64 x86_64
```

可以看到是`x86_64`是64的系统，可以在这里下载[rust安装包](https://static.rust-lang.org/dist/rust-1.5.0-x86_64-apple-darwin.pkg)。

2. 命令行一键安装：

Rust 提供简单的一键安装，命令如下：

```bash
> curl -sSf https://static.rust-lang.org/rustup.sh | sh
```

3. 使用mac os系统自带的brew安装

安装Rust不要直接Rust语言本身，例如使用`brew install rust`就只是安装了rust语言本身而已，应该安装的是rustup,rustup是rust官方版本的管理工具，是安装rust的首选。

它的主要特点是：

* 管理Rust二进制文件
* 配置Rust工具链
* 管理Rust相关组件
* 只依赖bash,curl和常见的unix工具
* 支持多平台

3. 验证安装：

如果你完成了上面任意一个步骤，请执行如下命令：

```bash 
> rustc --version
rustc 1.53.0
```
看到这个信息，表明你安装成功。

如果提示没有 rustc 命令，那么请回顾你是否有某个地方操作不对，请回过头来再看一遍文档。

这里需要注意下：

除了稳定版之外，Rust 还提供了 Beta 和 Nightly 版本[下载地址](https://www.rust-lang.org/zh-CN/other-installers.html)。

如果你不想安装 Rust 在你的电脑上，但是你还是像尝试一下 rust，那么这里有一个在线的环境：[play-rust](http://play.rust-lang.org/) 。

中国科学技术大学镜像源包含[rust-static](http://mirrors.ustc.edu.cn/rust-static/)。


#### Rust使用工具

在使用Rust开发过程中常常是用到的工具有`rustc`, `rust-src`,`cargo`，这些都可以使用rustup进行管理。

* cargo是Rust项目管理的工具，提供了一系列的工具，从项目的建立，构建到测试，运行到部署，都为Rust项目的管理提供尽可能完成的手段。

* rustc是rust语言的编译器。

* rust-src是rust标准库。


cargo在创建项目的时候，cargo 默认就创建 bin 类型的项目，Rust 项目主要分为两个类型：bin 和 lib，前者是一个可运行的项目，后者是一个依赖库项目。

#### 运行项目

有两种方式可以运行项目：

* cargo run
```markdown
> sudo cargo run main.rs
    Blocking waiting for file lock on package cache
    Finished dev [unoptimized + debuginfo] target(s) in 48.02s
     Running `/Users/keke/rust/step/target/debug/step main.rs`
hello rust
```

`cargo run` 对项目进行编译，然后再运行，实际它上等同于运行了两个指令,等同于手动编译运行。

* 手动编译和运行项目
```markdown
> cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
```
运行:
```markdown
> /target/debug/hello_rust
hello, rust!
```
从这里可以看出我们运行的是 debug 模式，在这种模式下，代码的编译速度会非常快，但是呢运行速度就慢了. 原因是，在 debug 模式下，Rust 编译器不会做任何的优化，只为了尽快的编译完成，让你的开发流程更加顺畅。

所以呢我们也可以选择:
```markdown
cargo run --release
cargo build --release
```
试着运行一下我们高性能的 release 程序：
```markdown
> ./target/release/hello_rust
hello, rust!
```

#### cargo check

`cargo check` 是我们在代码开发过程中最常用的命令，它的作用很简单：快速的检查一下代码能否编译通过。因此该命令速度会非常快，能节省大量的编译时间。
```markdown
> cargo check         
    Checking hello_rust v0.1.0 (/Users/keke/rust/hello_rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
```

#### Cargo.toml 和 Cargo.lock

`Cargo.toml` 和 `Cargo.lock` 是 cargo 的核心文件，rust的所有活动均基于这两个配置。

`Cargo.toml` 是 cargo 特有的项目数据描述文件。它存储了项目的所有元配置信息，如果 Rust 开发者希望 Rust 项目能够按照期望的方式进行构建、测试和运行，那么，必须按照合理的方式构建 `Cargo.toml`。

`Cargo.lock` 文件是 cargo 工具根据同一项目的 toml 文件生成的项目依赖详细清单，因此我们一般不用修改它，只需要对着 Cargo.toml 文件添加就好。

#### package 配置段落

package 中记录了项目的描述信息，典型的如下：
```markdown
[package]
name = "hello_rust"
version = "0.1.0"
edition = "202w"
```
`name` 字段定义了项目名称，
`version` 字段定义当前版本，新项目默认是 0.1.0，edition 字段定义了我们使用的 Rust 大版本。

定义项目依赖
使用 cargo 工具的最大优势就在于，能够对该项目的各种依赖项进行方便、统一和灵活的管理。

在 `Cargo.toml` 中，主要通过各种依赖段落来描述该项目的各种依赖项：

* 基于 Rust 官方仓库 `crates.io`，通过版本说明来描述
* 基于项目源代码的 git 仓库地址，通过 URL 来描述
* 基于本地项目的绝对路径或者相对路径，通过类 Unix 模式的路径来描述

具体如下:
```markdown
[dependencies]
rand = "0.3"
hammer = { version = "0.5.0"}
color = { git = "https://github.com/bjz/color-rs" }
geometry = { path = "crates/geometry" }
```

#### Rust实战开始

Rust这块主要从基础数据结构，语法和使用，然后结合damo一起来学习！


#### 参考

* [rust权威](https://course.rs/basic/variable.html)