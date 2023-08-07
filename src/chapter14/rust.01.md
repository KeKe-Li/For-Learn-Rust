### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go.
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++.
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查.

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 格式化输出

提到格式化输出，可能很多人立刻就想到 "{}"，但是 Rust 能做到的远比这个多的多。

看看格式化输出的初印象：
```rust
println!("Hello");                 // => "Hello"
println!("Hello, {}!", "world");   // => "Hello, world!"
println!("The number is {}", 1);   // => "The number is 1"
println!("{:?}", (3, 4));          // => "(3, 4)"
println!("{value}", value=4);      // => "4"
println!("{} {}", 1, 2);           // => "1 2"
println!("{:04}", 42);             // => "0042" with leading zeros
```

可以看到 println! 宏接受的是可变参数，第一个参数是一个字符串常量，它表示最终输出字符串的格式，包含其中形如 {} 的符号是占位符，会被 println! 后面的参数依次替换。

#### print!，println!，format!

它们是 Rust 中用来格式化输出的三大金刚，用途如下：

* `print!` 将格式化文本输出到标准输出，不带换行符.
* `println!` 同上，但是在行的末尾添加换行符.
* `format!` 将格式化文本输出到 String 字符串.

在实际项目中，最常用的是 println! 及 format!，前者常用来调试输出，后者常用来生成格式化的字符串：
```rust
fn main() {
    let s = "hello";
    println!("{}, world", s);
    let s1 = format!("{}, world", s);
    print!("{}", s1);
    print!("{}\n", "!");
}
```

其中，s1 是通过 format! 生成的 String 字符串，最终输出如下：
```markdown
hello, world
hello, world!
```

eprint!，eprintln!

使用方式跟 print!，println! 很像，但是它们输出到标准错误输出：
```markdown
eprintln!("Error: Could not complete task")
```
它们仅应该被用于输出错误信息和进度信息，其它场景都应该使用 print! 系列。




