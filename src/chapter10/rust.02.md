### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go.
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++.
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查.

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 可恢复的错误 Result

如果读取过程中，正常返回和遇到可以恢复的错误时该如何处理。

假设，我们有一台消息服务器，每个用户都通过 websocket 连接到该服务器来接收和发送消息，该过程就涉及到 socket 文件的读写，那么此时，如果一个用户的读写发生了错误，显然不能直接 panic，否则服务器会直接崩溃，所有用户都会断开连接，因此我们需要一种更温和的错误处理方式：Result<T, E>。

之前章节有提到过，`Result<T, E>` 是一个枚举类型，定义如下：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
泛型参数 T 代表成功时存入的正确值的类型，存放方式是 Ok(T)，E 代表错误时存入的错误值，存放方式是 Err(E)，枯燥的讲解永远不及代码生动准确，因此先来看下打开文件的例子：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```
以上 File::open 返回一个 Result 类型，那么问题来了：

```markdown
如何获知变量类型或者函数的返回类型
有几种常用的方式，此处更推荐第二种方法：

第一种是查询标准库或者三方库文档，搜索 File，然后找到它的 open 方法
在 Rust IDE 章节，我们推荐了 VSCode IDE 和 rust-analyzer 插件，如果你成功安装的话，那么就可以在 VSCode 中很方便的通过代码跳转的方式查看代码，同时 rust-analyzer 插件还会对代码中的类型进行标注，非常方便好用！
你还可以尝试故意标记一个错误的类型，然后让编译器告诉你：
```

```rust
let f: u32 = File::open("hello.txt");
```
错误提示如下：

```bash
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
             found type `std::result::Result<std::fs::File, std::io::Error>`
```

上面代码，故意将 f 类型标记成整形，编译器立刻不乐意了，你是在忽悠我吗？打开文件操作返回一个整形？来，大哥来告诉你返回什么：`std::result::Result<std::fs::File`, `std::io::Error>`，但是，怎么这么长的类型！

别慌，其实很简单，首先 Result 本身是定义在 `std::result` 中的，但是因为 Result 很常用，所以就被包含在了 prelude 中（将常用的东东提前引入到当前作用域内），因此无需手动引入 `std::result::Result`，那么返回类型可以简化为 `Result<std::fs::File,std::io::Error>`，你看看是不是很像标准的 `Result<T, E>` 枚举定义？

只不过 T 被替换成了具体的类型 `std::fs::File`，是一个文件句柄类型，E 被替换成 `std::io::Error`，是一个 IO 错误类型。

这个返回值类型说明 `File::open` 调用如果成功则返回一个可以进行读写的文件句柄，如果失败，则返回一个 IO 错误：文件不存在或者没有访问文件的权限等。总之 `File::open` 需要一个方式告知调用者是成功还是失败，并同时返回具体的文件句柄(成功)或错误信息(失败)，万幸的是，这些信息可以通过 Result 枚举提供：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```
代码很清晰，对打开文件后的 `Result<T, E>` 类型进行匹配取值，如果是成功，则将 Ok(file) 中存放的的文件句柄 file 赋值给 f，如果失败，则将 Err(error) 中存放的错误信息 error 使用 panic 抛出来，进而结束程序，这非常符合上文提到过的 panic 使用场景。

好吧，也没有那么合理 `:)`.
