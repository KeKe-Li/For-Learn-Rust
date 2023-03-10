### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

####  集合类型

集合在 Rust 中是一类比较特殊的类型，因为 Rust 中大多数数据类型都只能代表一个特定的值，但是集合却可以代表一大堆值。而且与语言级别的数组、字符串类型不同，标准库里的这些家伙是分配在堆上，因此都可以进行动态的增加和减少。


#### 动态数组 Vector

动态数组类型用 `Vec<T>`,动态数组允许你存储多个值，这些值在内存中一个紧挨着另一个排列，因此访问其中某个元素的成本非常低。动态数组只能存储相同类型的元素，如果你想存储不同类型的元素，可以使用枚举类型或者特征对象。

总之，当我们想拥有一个列表，里面都是相同类型的数据时，动态数组将会非常有用。


#### 创建动态数组

在 Rust 中，有多种方式可以创建动态数组. 我们可以使用 `Vec::new` 创建动态数组是最 rusty 的方式，它调用了 Vec 中的 new 关联函数：

```rust
let v: Vec<i32> = Vec::new();
```

这里，v 被显式地声明了类型 `Vec<i32>`，这是因为 Rust 编译器无法从 `Vec::new()` 中得到任何关于类型的暗示信息，因此也无法推导出 v 的具体类型，但是当你向里面增加一个元素后，一切又不同了：
```rust
let mut v = Vec::new();
v.push(1);
```
此时，v 就无需手动声明类型，因为编译器通过 `v.push(1)`，推测出 v 中的元素类型是 i32，因此推导出 v 的类型是 `Vec<i32>`。

如果预先知道要存储的元素个数，可以使用 Vec::with_capacity(capacity) 创建动态数组，这样可以避免因为插入大量新数据导致频繁的内存分配和拷贝，提升性能。

#### 用宏创建数组

使用宏 `vec!` 来创建数组，与 `Vec::new` 有所不同，前者能在创建同时给予初始化值：

```rust
let v = vec![1, 2, 3];
```
此处的 v 也无需标注类型，编译器只需检查它内部的元素即可自动推导出 v 的类型是 `Vec<i32>` .

#### Vector

向数组尾部添加元素，可以使用 push 方法：
```rust
let mut v = Vec::new();
v.push(1);
```
与其它类型一样，必须将 v 声明为 mut 后，才能进行修改。

#### Vector 与其元素共存亡

跟结构体一样，Vector 类型在超出作用域范围后，会被自动删除：
```rust
{
    let v = vec![1, 2, 3];

    // ...
} // <- v超出作用域并在此处被删除
```
当 Vector 被删除后，它内部存储的所有内容也会随之被删除。目前来看，这种解决方案简单直白，但是当 Vector 中的元素被引用后，事情可能会没那么简单。

#### 从 Vector 中读取元素

读取指定位置的元素有两种方式可选：

* 通过下标索引访问。
* 使用 get 方法。

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("第三个元素是 {}", third);

match v.get(2) {
    Some(third) => println!("第三个元素是 {third}"),
    None => println!("去你的第三个元素，根本没有！"),
}
```
和其它语言一样，集合类型的索引下标都是从 0 开始，`&v[2]` 表示借用 v 中的第三个元素，最终会获得该元素的引用。
而 `v.get(2)` 也是访问第三个元素，但是有所不同的是，它返回了 `Option<&T>`，因此还需要额外的 match 来匹配解构出具体的值。

#### 下标索引与 .get 的区别

这两种方式都能成功的读取到指定的数组元素，既然如此为什么会存在两种方法？何况 `.get` 还会增加使用复杂度，这就涉及到数组越界的问题了，让我们通过示例说明：

```rust

let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

这里`&v[100]` 的访问方式会导致程序报错退出，因为发生了数组越界访问。 但是 `v.get` 就不会，它在内部做了处理，有值的时候返回 `Some(T)`，无值的时候返回 None，因此 `v.get` 的使用方式非常安全。

既然如此，为何不统一使用 `v.get` 的形式？因为实在是有些繁琐，Rust 语言的设计者和使用者在审美这方面还是相当统一的：简洁即正义，何况性能上也会有轻微的损耗。

既然有两个选择，肯定就有如何选择的问题，答案很简单，当你确保索引不会越界的时候，就用索引访问，否则用 `.get`。例如，访问第几个数组元素并不取决于我们，而是取决于用户的输入时，用 `.get` 会非常适合。

#### 同时借用多个数组元素

既然涉及到借用数组元素，那么很可能会遇到同时借用多个数组元素的情况。
```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);

println!("The first element is: {first}");
```
先不运行，我们来推断下结果，首先 `first = &v[0]` 进行了不可变借用，`v.push` 进行了可变借用，如果 first 在 `v.push` 之后不再使用，那么该段代码可以成功编译.

可是上面的代码中，first 这个不可变借用在可变借用 `v.push` 后被使用了，那么妥妥的，编译器就会报错：
```bash
$ cargo run
Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable 无法对v进行可变借用，因此之前已经进行了不可变借用
--> src/main.rs:6:5
|
4 |     let first = &v[0];
|                  - immutable borrow occurs here // 不可变借用发生在此处
5 |
6 |     v.push(6);
|     ^^^^^^^^^ mutable borrow occurs here // 可变借用发生在此处
7 |
8 |     println!("The first element is: {}", first);
|                                          ----- immutable borrow later used here // 不可变借用在这里被使用

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to previous error
```
这里这两个引用不应该互相影响的：一个是查询元素，一个是在数组尾部插入元素，完全不相干的操作，为何编译器要这么严格呢？

原因在于：数组的大小是可变的，当旧数组的大小不够用时，Rust 会重新分配一块更大的内存空间，然后把旧数组拷贝过来。 这种情况下，之前的引用显然会指向一块无效的内存，这非常 rusty —— 对用户进行严格的教育。

#### 迭代遍历 Vector 中的元素

如果想要依次访问数组中的元素，可以使用迭代的方式去遍历数组，这种方式比用下标的方式去遍历数组更安全也更高效（每次下标访问都会触发数组边界检查）：
```rust
let v = vec![1, 2, 3];
for i in &v {
    println!("{i}");
}
```
也可以在迭代过程中，修改 Vector 中的元素：
```rust
let mut v = vec![1, 2, 3];
for i in &mut v {
    *i += 10
}
```

#### 存储不同类型的元素

我们知道在数组中，数组的元素必须类型相同，但是有什么办法能存储不同类型的数据吗？答案是有的,那就是通过使用枚举类型和特征对象来实现不同类型元素的存储。

先来看看通过枚举如何实现：
```rust
#[derive(Debug)]
enum IpAddr {
    V4(String),
    V6(String)
}
fn main() {
    let v = vec![
        IpAddr::V4("127.0.0.1".to_string()),
        IpAddr::V6("::1".to_string())
    ];

    for ip in v {
        show_addr(ip)
    }
}

fn show_addr(ip: IpAddr) {
    println!("{:?}",ip);
}
```

数组 v 中存储了两种不同的 ip 地址，但是这两种都属于 IpAddr 枚举类型的成员，因此可以存储在数组中。

再来看看特征对象的实现：
```rust
trait IpAddr {
    fn display(&self);
}

struct V4(String);
impl IpAddr for V4 {
    fn display(&self) {
        println!("ipv4: {:?}",self.0)
    }
}
struct V6(String);
impl IpAddr for V6 {
    fn display(&self) {
        println!("ipv6: {:?}",self.0)
    }
}

fn main() {
    let v: Vec<Box<dyn IpAddr>> = vec![
        Box::new(V4("127.0.0.1".to_string())),
        Box::new(V6("::1".to_string())),
    ];

    for ip in v {
        ip.display();
    }
}
```
比枚举实现要稍微复杂一些，我们为 V4 和 V6 都实现了特征 IpAddr，然后将它俩的实例用 `Box::new` 包裹后，存在了数组 v 中，需要注意的是，这里必须手动地指定类型：`Vec<Box<dyn IpAddr>>`，表示数组 v 存储的是特征 IpAddr 的对象，这样就实现了在数组中存储不同的类型。

在实际使用场景中，特征对象数组要比枚举数组常见很多，主要原因在于特征对象非常灵活，而编译器对枚举的限制较多，且无法动态增加类型。