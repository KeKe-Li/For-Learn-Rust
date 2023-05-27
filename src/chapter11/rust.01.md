### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go.
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++.
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查.

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 返回值和错误处理

Rust 中的错误主要分为两类：

* 可恢复错误，通常用于从系统全局角度来看可以接受的错误，例如处理用户的访问、操作等错误，这些错误只会影响某个用户自身的操作进程，而不会对系统的全局稳定性产生影响。

* 不可恢复错误，刚好相反，该错误通常是全局性或者系统性的错误，例如数组越界访问，系统启动时发生了影响启动流程的错误等等，这些错误的影响往往对于系统来说是致命的。

#### panic 深入剖析

让我们先做一个假设：文件读取操作发生在系统启动阶段。那么可以轻易得出一个结论，一旦文件读取失败，那么系统启动也将失败，这意味着该失败是不可恢复的错误，无论是因为文件不存在还是操作系统硬盘的问题，这些只是错误的原因不同，但是归根到底都是不可恢复的错误(梳理清楚当前场景的错误类型非常重要)。

对于这些严重到影响程序运行的错误，触发 panic 是很好的解决方式。在 Rust 中触发 panic 有两种方式：被动触发和主动调用，下面依次来看看。

#### panic! 与不可恢复错误

* 被动触发

先来看一段简单又熟悉的代码:
```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

这里一看，立马就能看出这里发生了严重的错误 —— 数组访问越界，在其它编程语言中无一例外，都会报出严重的异常，甚至导致程序直接崩溃关闭。

而 Rust 也不例外，运行后将看到如下报错：
```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
上面给出了非常详细的报错信息，包含了具体的异常描述以及发生的位置，甚至你还可以加入额外的命令来看到异常发生时的堆栈信息，这个会在后面详细展开。

总之，类似的 panic 还有很多，而被动触发的 panic 是我们日常开发中最常遇到的，这也是 Rust 给我们的一种保护，毕竟错误只有抛出来，才有可能被处理，否则只会偷偷隐藏起来，寻觅时机给你致命一击。

* 主动调用

在某些特殊场景中，开发者想要主动抛出一个异常，例如开头提到的在系统启动阶段读取文件失败。

对此，Rust 为我们提供了 `panic!` 宏，当调用执行该宏时，程序会打印出一个错误信息，展开报错点往前的函数调用堆栈，最后退出程序。

这里需要注意一点，一定是不可恢复的错误，才调用 `panic!` 处理，你总不想系统仅仅因为用户随便传入一个非法参数就崩溃吧？所以，只有当你不知道该如何处理时，再去调用 panic!.

首先，来调用一下 `panic!`，这里使用了最简单的代码实现，实际上你在程序的任何地方都可以这样调用：
```rust
fn main() {
    panic!("crash and burn");
}
```
运行后输出:
```rust
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

以上信息包含了两条重要信息：
```markdown
1. main 函数所在的线程崩溃了，发生的代码位置是 src/main.rs 中的第 2 行第 5 个字符（去除该行前面的空字符）
在使用时加上一个环境变量可以获取更详细的栈展开信息：

2. Linux/macOS 等 UNIX 系统： RUST_BACKTRACE=1 cargo run
Windows 系统（PowerShell）： $env:RUST_BACKTRACE=1 ; cargo run
```

#### backtrace 栈展开

在开发的项目中,错误往往涉及到很长的调用链甚至会深入第三方库，如果没有栈展开技术，错误将难以跟踪处理，下面我们来看一个真实的崩溃例子：
```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```
上面的代码很简单，数组只有 3 个元素，我们却尝试去访问它的第 100 号元素(数组索引从 0 开始)，那自然会崩溃。

此时肯定要质疑，一个简单的数组越界访问，为何要直接让程序崩溃？是不是有些小题大作了？

如果有过 C 语言的经验，即使你越界了，问题不大，我依然尝试去访问，至于这个值是不是你想要的（100 号内存地址也有可能有值，只不过是其它变量或者程序的！），抱歉，不归我管，我只负责取，你要负责管理好自己的索引访问范围。

上面这种情况被称为缓冲区溢出，并可能会导致安全漏洞，例如攻击者可以通过索引来访问到数组后面不被允许的数据。

说实话，我宁愿程序崩溃，为什么？当你取到了一个不属于你的值，这在很多时候会导致程序上的逻辑 BUG！ 有编程经验的人都知道这种逻辑上的 BUG 是多么难被发现和修复！因此程序直接崩溃，然后告诉我们问题发生的位置，最后我们对此进行修复，这才是最合理的软件开发流程，而不是把问题藏着掖着：

```bash
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
好的，现在成功知道问题发生的位置，但是如果我们想知道该问题之前经过了哪些调用环节，该怎么办？

那就按照提示使用 `RUST_BACKTRACE=1 cargo run` 或 `$env:RUST_BACKTRACE=1 ; cargo run` 来再一次运行程序：
```bash
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/std/src/panicking.rs:517:5
   1: core::panicking::panic_fmt
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:101:14
   2: core::panicking::panic_bounds_check
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:77:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:184:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:15:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/alloc/src/vec/mod.rs:2465:9
   6: world_hello::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/ops/function.rs:227:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```
上面的代码就是一次栈展开(也称栈回溯)，它包含了函数调用的顺序，当然按照逆序排列：最近调用的函数排在列表的最上方。因为咱们的 main 函数基本是最先调用的函数了，所以排在了倒数第二位，还有一个关注点，排在最顶部最后一个调用的函数是 `rust_begin_unwind`，该函数的目的就是进行栈展开，呈现这些列表信息给我们。

要获取到栈回溯信息，你还需要开启 debug 标志，该标志在使用 `cargo run` 或者 `cargo build` 时自动开启（这两个操作默认是 Debug 运行方式）。同时，栈展开信息在不同操作系统或者 Rust 版本上也有所不同。


#### panic 时的两种终止方式

当出现 `panic!` 时，程序提供了两种方式来处理终止流程：栈展开和直接终止。

其中，默认的方式就是 栈展开，这意味着 Rust 会回溯栈上数据和函数调用，因此也意味着更多的善后工作，好处是可以给出充分的报错信息和栈调用信息，便于事后的问题复盘。直接终止，顾名思义，不清理数据就直接退出程序，善后工作交与操作系统来负责。

对于绝大多数用户，使用默认选择是最好的，但是当你关心最终编译出的二进制可执行文件大小时，那么可以尝试去使用直接终止的方式，例如下面的配置修改 `Cargo.toml` 文件，实现在 release 模式下遇到 panic 直接终止：

```bash
[profile.release]
panic = 'abort'
```

#### 线程 panic 后，程序是否会终止？

长话短说，如果是 main 线程，则程序会终止，如果是其它子线程，该线程会终止，但是不会影响 main 线程。因此，尽量不要在 main 线程中做太多任务，将这些任务交由子线程去做，就算子线程 panic 也不会导致整个程序的结束。

这里涉及到当调用 `panic!` 宏时，它会:

* 格式化 panic 信息，然后使用该信息作为参数，调用 `std::panic::panic_any()` 函数.
* `panic_any` 会检查应用是否使用了 `panic hook`，如果使用了，该 hook 函数就会被调用（hook 是一个钩子函数，是外部代码设置的，用于在 panic 触发时，执行外部代码所需的功能）.
* 当 hook 函数返回后，当前的线程就开始进行栈展开：从 panic_any 开始，如果寄存器或者栈因为某些原因信息错乱了，那很可能该展开会发生异常，最终线程会直接停止，展开也无法继续进行.
* 展开的过程是一帧一帧的去回溯整个栈，每个帧的数据都会随之被丢弃，但是在展开过程中，你可能会遇到被用户标记为 catching 的帧（通过 `std::panic::catch_unwind()` 函数标记），此时用户提供的 catch 函数会被调用，展开也随之停止：当然，如果 catch 选择在内部调用 `std::panic::resume_unwind()` 函数，则展开还会继续。

还有一种情况，在展开过程中，如果展开本身 panic 了，那展开线程会终止，展开也随之停止。

一旦线程展开被终止或者完成，最终的输出结果是取决于哪个线程 `panic：`对于 main 线程，操作系统提供的终止功能 `core::intrinsics::abort()` 会被调用，最终结束当前的 panic 进程；如果是其它子线程，那么子线程就会简单的终止，同时信息会在稍后通过 `std::thread::join()` 进行收集。


#### 何时该使用 panic!

让我们大概罗列下何时适合使用 panic，也许经过之前的学习，你已经能够对 panic 的使用有了自己的看法，但是我们还是会罗列一些常见的用法来加深自己的理解。

先看下`Result<T, E>` 这个枚举类型，它是用来表示函数的返回结果：
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
当没有错误发生时，函数返回一个用 Result 类型包裹的值 `Ok(T)`，当错误时，返回一个 `Err(E)`。对于 Result 返回我们有很多处理方法，最简单粗暴的就是 unwrap 和 expect，这两个函数非常类似，我们以 unwrap 举例：
```rust
use std::net::IpAddr;
let home: IpAddr = "127.0.0.1".parse().unwrap();
```
上面的 parse 方法试图将字符串 `"127.0.0.1"` 解析为一个 IP 地址类型 IpAddr，它返回一个 `Result<IpAddr, E>` 类型，如果解析成功，则把 `Ok(IpAddr)` 中的值赋给 home，如果失败，则不处理 Err(E)，而是直接 panic。

因此 unwrap 简而言之：成功则返回值，失败则 panic，总之不进行任何错误处理。