### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go.
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++.
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查.

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 包和模块

当工程规模变大时，把代码写到一个甚至几个文件中，都是不太聪明的做法，可能存在以下问题：

1. 单个文件过大，导致打开、翻页速度大幅变慢
2. 查询和定位效率大幅降低，类比下，你会把所有知识内容放在一个几十万字的文档中吗？
3. 只有一个代码层次：函数，难以维护和协作，想象一下你的操作系统只有一个根目录，剩下的都是单层子目录会如何：disaster.
4. 容易滋生 Bug

同时，将大的代码文件拆分成包和模块，还允许我们实现代码抽象和复用：将你的代码封装好后提供给用户，那么用户只需要调用公共接口即可，无需知道内部该如何实现。

因此，跟其它语言一样，Rust 也提供了相应概念用于代码的组织管理：

* 项目(Packages)：一个 Cargo 提供的 feature，可以用来构建、测试和分享包.
* 包(Crate)：一个由多个模块组成的树形结构，可以作为三方库进行分发，也可以生成可执行文件进行运行.
* 模块(Module)：可以一个文件多个模块，也可以一个文件一个模块，模块可以被认为是真实项目中的代码组织单元.

#### 包 和 Package

* 项目(Package)：可以用来构建、测试和分享包.
* 工作空间(WorkSpace)：对于大型项目，可以进一步将多个包联合在一起，组织成工作空间.
* 包(Crate)：一个由多个模块组成的树形结构，可以作为三方库进行分发，也可以生成可执行文件进行运行.
* 模块(Module)：可以一个文件多个模块，也可以一个文件一个模块，模块可以被认为是真实项目中的代码组织单元.

* 包 Crate

对于 Rust 而言，包是一个独立的可编译单元，它编译后会生成一个可执行文件或者一个库。

一个包会将相关联的功能打包在一起，使得该功能可以很方便的在多个项目中分享。例如标准库中没有提供但是在三方库中提供的 rand 包，它提供了随机数生成的功能，我们只需要将该包通过 use rand; 引入到当前项目的作用域中，就可以在项目中使用 rand 的功能：`rand::XXX`。

同一个包中不能有同名的类型，但是在不同包中就可以。例如，虽然 rand 包中，有一个 Rng 特征，可是我们依然可以在自己的项目中定义一个 Rng，前者通过 `rand::Rng` 访问，后者通过 Rng 访问，对于编译器而言，这两者的边界非常清晰，不会存在引用歧义。


#### 项目 Package

rust中包的名称被 crate 占用，库的名称被 library 占用，所以将 Package 翻译成项目，你也可以理解为工程、软件包。

由于 Package 就是一个项目，因此它包含有独立的 `Cargo.toml` 文件，以及因为功能性被组织在一起的一个或多个包。

一个 Package 只能包含一个库(library)类型的包，但是可以包含多个二进制可执行类型的包。

* 二进制 Package

让我们来创建一个二进制 Package：
```markdown
> cargo new my-project
     Created binary (application) `my-project` package
> ls my-project
Cargo.toml
src
> ls my-project/src
main.rs
```

这里，Cargo 为我们创建了一个名称是 my-project 的 Package，同时在其中创建了 `Cargo.toml` 文件，可以看一下该文件，里面并没有提到 `src/main.rs` 作为程序的入口，原因是 Cargo 有一个惯例：`src/main.rs` 是二进制包的根文件，该二进制包的包名跟所属 Package 相同，在这里都是 my-project，所有的代码执行都从该文件中的 `fn main()`函数开始。

使用 `cargo run` 可以运行该项目，输出：`Hello, world!`。

* 库 Package

再来创建一个库类型的 Package：
```markdown
> cargo new my-lib --lib
     Created library `my-lib` package
> ls my-lib
Cargo.toml
src
> ls my-lib/src
lib.rs
```
首先，如果你试图运行 my-lib，会报错：
```markdown
> cargo run
error: a bin target must be available for `cargo run`
```

原因是库类型的 Package 只能作为三方库被其它项目引用，而不能独立运行，只有之前的二进制 Package 才可以运行。

与 `src/main.rs` 一样，Cargo 知道，如果一个 Package 包含有 `src/lib.rs`，意味它包含有一个库类型的同名包 my-lib，该包的根文件是 `src/lib.rs`。


注意： 

这里有个混淆的概念：

Package 和包容易混淆，因为用 cargo new 创建的 Package 和它其中包含的包是同名的！

不过，只要你牢记 Package 是一个项目工程，而包只是一个编译单元，基本上也就不会混淆这个两个概念了：`src/main.rs` 和 `src/lib.rs` 都是编译单元，因此它们都是包。


#### 典型的 Package 结构

上面创建的 Package 中仅包含 `src/main.rs` 文件，意味着它仅包含一个二进制同名包 my-project。如果一个 Package 同时拥有 `src/main.rs` 和 `src/lib.rs`，那就意味着它包含两个包：库包和二进制包，这两个包名也都是 my-project —— 都与 Package 同名。

一个真实项目中典型的 Package，会包含多个二进制包，这些包文件被放在 `src/bin` 目录下，每一个文件都是独立的二进制包，同时也会包含一个库包，该包只能存在一个 src/lib.rs`：
```markdown
.
├── Cargo.toml
├── Cargo.lock
├── src
│   ├── main.rs
│   ├── lib.rs
│   └── bin
│       └── main1.rs
│       └── main2.rs
├── tests
│   └── some_integration_tests.rs
├── benches
│   └── simple_bench.rs
└── examples
    └── simple_example.rs
```

```markdown
* 唯一库包：src/lib.rs
* 默认二进制包：src/main.rs，编译后生成的可执行文件与 Package 同名
* 其余二进制包：src/bin/main1.rs 和 src/bin/main2.rs，它们会分别生成一个文件同名的二进制可执行文件
* 集成测试文件：tests 目录下
* 基准性能测试 benchmark 文件：benches 目录下
* 项目示例：examples 目录下
```

这种目录结构基本上是 Rust 的标准目录结构，在 GitHub 的大多数项目上，你都将看到它的身影。


