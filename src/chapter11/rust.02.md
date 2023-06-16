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

#### 对返回的错误进行处理

直接 panic 还是过于粗暴，因为实际上 IO 的错误有很多种，我们需要对部分错误进行特殊处理，而不是所有错误都直接崩溃：
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```
上面代码在匹配出 error 后，又对 error 进行了详细的匹配解析，最终结果：

如果是文件不存在错误 `ErrorKind::NotFound`，就创建文件，这里创建文件`File::create` 也是返回 Result，因此继续用 match 对其结果进行处理：创建成功，将新的文件句柄赋值给 f，如果失败，则 panic.
剩下的错误，一律 panic.

#### 失败就 panic: unwrap 和 expect

在不需要处理错误的场景，例如写原型、示例时，我们不想使用 match 去匹配 `Result<T, E>` 以获取其中的 T 值，因为 match 的穷尽匹配特性，你总要去处理下 Err 分支。那么有没有办法简化这个过程？有，答案就是 unwrap 和 expect。

它们的作用就是，如果返回成功，就将 Ok(T) 中的值取出来，如果失败，就直接 panic。

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```
如果调用这段代码时 `hello.txt` 文件不存在，那么 unwrap 就将直接 panic：
```rust
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:4:37
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
expect 跟 unwrap 很像，也是遇到错误直接 panic, 但是会带上自定义的错误提示信息，相当于重载了错误打印的函数：
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```
报错如下：
```rust
thread 'main' panicked at 'Failed to open hello.txt: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:4:37
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

可以看出，expect 相比 unwrap 能提供更精确的错误信息，在有些场景也会更加实用。

#### 传播错误

们的程序几乎不太可能只有 A->B 形式的函数调用，一个设计良好的程序，一个功能涉及十几层的函数调用都有可能。而错误处理也往往不是哪里调用出错，就在哪里处理，实际应用中，大概率会把错误层层上传然后交给调用链的上游函数进行处理，错误传播将极为常见。

例如以下函数从文件中读取用户名，然后将结果进行返回：
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    // 打开文件，f是`Result<文件句柄,io::Error>`
    let f = File::open("hello.txt");

    let mut f = match f {
        // 打开文件成功，将file句柄赋值给f
        Ok(file) => file,
        // 打开文件失败，将错误返回(向上传播)
        Err(e) => return Err(e),
    };
    // 创建动态字符串s
    let mut s = String::new();
    // 从f文件句柄读取数据并写入s中
    match f.read_to_string(&mut s) {
        // 读取成功，返回Ok封装的字符串
        Ok(_) => Ok(s),
        // 将错误向上传播
        Err(e) => Err(e),
    }
}
```
有几点值得注意：

* 该函数返回一个 `Result<String, io::Error>` 类型，当读取用户名成功时，返回 `Ok(String)`，失败时，返回 `Err(io:Error)`.
* `File::open` 和 `f.read_to_string` 返回的 `Result<T, E>` 中的 E 就是 `io::Error`
由此可见，该函数将 `io::Error` 的错误往上进行传播，该函数的调用者最终会对 `Result<String,io::Error>` 进行再处理，至于怎么处理就是调用者的事，如果是错误，它可以选择继续向上传播错误，也可以直接 panic，亦或将具体的错误原因包装后写入 socket 中呈现给终端用户。

但是上面的代码也有自己的问题，那就是太长了，我自认为也有那么一点点优秀，因此见不得这么啰嗦的代码，下面咱们来讲讲如何简化它。

* 传播界的大明星: ?

我们看看 ? 的排面：
```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```
看到没，这就是排面，相比前面的 match 处理错误的函数，代码直接减少了一半不止，但是，一山更比一山难，看不懂啊！

其实 ? 就是一个宏，它的作用跟上面的 match 几乎一模一样：
```rust
let mut f = match f {
    // 打开文件成功，将file句柄赋值给f
    Ok(file) => file,
    // 打开文件失败，将错误返回(向上传播)
    Err(e) => return Err(e),
};
```
如果结果是 Ok(T)，则把 T 赋值给 f，如果结果是 Err(E)，则返回该错误，所以 `?` 特别适合用来传播错误。

虽然 `?` 和 match 功能一致，但是事实上 `?` 会更胜一筹。何解？

想象一下，一个设计良好的系统中，肯定有自定义的错误特征，错误之间很可能会存在上下级关系，例如标准库中的 `std::io::Error` 和 `std::error::Error`，前者是 IO 相关的错误结构体，后者是一个最最通用的标准错误特征，同时前者实现了后者，因此 `std::io::Error` 可以转换为 `std:error::Error`。


