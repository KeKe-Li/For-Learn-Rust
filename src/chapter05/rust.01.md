### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 流程控制

Rust 程序是从上而下顺序执行的，在这个过程中，我们可以通过循环、分支等流程控制方式，更好的实现相应的功能。

常用的流程控制包括:

* 使用 if 来做分支控制

if else 表达式根据条件执行不同的代码分支：
```rust
if condition == true {
    // A...
} else {
    // B...
}
```

* 使用 else if 来处理多重条件

将 else if 与 if、else 组合在一起实现更复杂的条件分支判断:
```rust
fn main() {
    let n = 4;

    if n % 4 == 0 {
        println!("number is divisible by 4");
    } else if n % 6 == 0 {
        println!("number is divisible by 6");
    } else if n % 5== 0 {
        println!("number is divisible by 5");
    } else {
        println!("number is not divisible by 4, 6, or 5");
    }
}    
```

程序执行时，会按照自上至下的顺序执行每一个分支判断，一旦成功，则跳出 if 语句块，最终本程序会匹配执行 else if n % 4 == 0 的分支，输出 "number is divisible by 4"。

有一点要注意，就算有多个分支能匹配，也只有第一个匹配的分支会被执行！

循环控制包括:

* for 循环

* continue

* break

* while 循环

* loop 循环