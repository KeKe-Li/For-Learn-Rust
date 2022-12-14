### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 引用与借用

Rust 通过 借用(Borrowing) 这个概念来达成上述的目的，获取变量的引用，称之为借用(borrowing)。正如现实生活中，如果一个人拥有某样东西，你可以从他那里借来，当使用完毕后，也必须要物归原主。

#### 引用与解引用

常规引用是一个指针类型，指向了对象存储的内存地址。

例如，创建一个 i32 值的引用y，然后使用解引用运算符来解出 y 所使用的值:
```rust
fn main() {
    let x = 10;
    let y = &x;

    assert_eq!(10, x);
    assert_eq!(10, *y);
}
```
变量 x 存放了一个 i32 值 10。y 是 x 的一个引用。可以断言 x 等于 10。然而，如果希望对 y 的值做出断言，必须使用 *y 来解出引用所指向的值（也就是解引用）。一旦解引用了 y，就可以访问 y 所指向的整型值并可以与 10 做比较。

#### 不可变引用

我们可以通过一段代码来看，用 s1 的引用作为参数传递给 calculate_length 函数，而不是把 s1 的所有权转移给该函数：
```rust
fn main() {
    let s1 = String::from("great");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

能注意到两点：

* 先通过函数参数传入所有权，然后再通过函数返回来传出所有权，代码更加简洁.
* `calculate_length` 的参数 s 类型从 String 变为 `&String`

这里，`&` 符号即是引用，它们允许你使用值，但是不获取所有权，如图所示：

<p align="center">
    <img width="100%" align="center" src="src/images/3.png" />
</p>

通过 `&s1` 语法，我们创建了一个指向 s1 的引用，但是并不拥有它。因为并不拥有这个值，当引用离开作用域后，其指向的值也不会被丢弃。

同理，函数 `calculate_length` 使用 `&` 来表明参数 s 的类型是一个引用：

```rust
fn calculate_length(s: &String) -> usize { // s 是对 String 的引用
    s.len()
} // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权，
  // 所以什么也不会发生
```

#### 可变引用

有了不可变引用就会有可变引用:

```rust
fn main() {
    let mut s = String::from("great");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", rust");
}
```

首先，声明 s 是可变类型，其次创建一个可变的引用 `&mut s` 和接受可变引用参数 `some_string: &mut String` 的函数。


#### NLL

对于这种编译器优化行为，Rust 专门起了一个名字 —— Non-Lexical Lifetimes(NLL)，专门用于找到某个引用在作用域(})结束前就不再被使用的代码位置。


#### 悬垂引用(Dangling References)

悬垂引用也叫做悬垂指针，意思为指针指向某个值后，这个值被释放掉了，而指针仍然存在，其指向的内存可能不存在任何值或已被其它变量重新使用。

在 Rust 中编译器可以确保引用永远也不会变成悬垂状态：当你获取数据的引用后，编译器可以确保数据不会在引用结束前被释放，要想释放数据，必须先停止其引用的使用。

让我们尝试创建一个悬垂引用，Rust 会抛出一个编译时错误：
```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("great");

    &s
}
```

运行后错误:
```rust
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                ~~~~~~~~

```

错误信息引用了一个功能：生命周期(lifetimes)。不过，即使现在还不理解生命周期，也可以通过错误信息知道这段代码错误的关键信息：

```rust
this function's return type contains a borrowed value, but there is no value for it to be borrowed from.
该函数返回了一个借用的值，但是已经找不到它所借用值的来源
```

仔细看看 dangle 代码的每一步到底发生了什么：
```rust

fn dangle() -> &String { // dangle 返回一个字符串的引用

    let s = String::from("hello"); // s 是一个新字符串

    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。
  // 危险！
```

因为 s 是在 dangle 函数内创建的，当 dangle 的代码执行完毕后，s 将被释放, 但是此时我们又尝试去返回它的引用。这意味着这个引用会指向一个无效的 String，这是不正确的。

其中一个很好的解决方法是直接返回 String：
```rust
fn no_dangle() -> String {
    let s = String::from("great");

    s
}
```

这样就没有任何错误了，最终 String 的所有权被转移给外面的调用者。


总的来说，借用规则如下：

* 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用.
* 引用必须总是有效的.

