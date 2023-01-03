### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 泛型 Generics

在开始了解 Rust 的泛型之前，我们先来看看什么是多态？

多态是同一个行为具有多个不同表现形式或形态的能力。

多态就是同一个接口，使用不同的实例而执行不同操作，举个例子。

在编程的时候，我们经常利用多态。通俗的讲，多态就是好比坦克的炮管，既可以发射普通弹药，也可以发射制导炮弹（导弹），也可以发射贫铀穿甲弹，甚至发射子母弹，没有必要为每一种炮弹都在坦克上分别安装一个专用炮管，即使生产商愿意，炮手也不愿意。所以在编程开发中，我们也需要这样“通用的炮管”，这个“通用的炮管”就是多态。

实际上，泛型就是一种多态。泛型主要目的是为程序员提供编程的便利，减少代码的臃肿，同时可以极大地丰富语言本身的表达能力，为程序员提供了一个合适的炮管。

```rust
fn add<T>(a:T, b:T) -> T {
    a + b
}

fn main() {
    println!("add i8: {}", add(2i8, 3i8));
    println!("add i32: {}", add(20, 30));
    println!("add f64: {}", add(1.23, 1.23));
}
```

这个就是Rust 泛型。

#### 泛型详解
代码中的 `T` 就是泛型参数，实际上在 Rust 中，泛型参数的名称你可以任意起，但是出于惯例，我们都用 T ( T 是 type 的首字母)来作为首选，这个名称越短越好，除非需要表达含义，否则一个字母是最完美的。

使用泛型参数，有一个先决条件，必需在使用前对其进行声明：

```rust
fn largest<T>(list: &[T]) -> T {}
```

该泛型函数的作用是从列表中找出最大的值，其中列表中的元素类型为 T。首先 `largest<T>` 对泛型参数 T 进行了声明，然后才在函数参数中进行使用该泛型参数 `list: &[T]` （还记得 `&[T]` 类型吧？这是数组切片）。

总之，我们可以这样理解这个函数定义：函数 largest 有泛型类型 T，它有个参数 list，其类型是元素为 T 的数组切片，最后，该函数返回值的类型也是 T。

实现一个泛型函数实现如下：
```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```
运行后报错：
```rust
error[E0369]: binary operation `>` cannot be applied to type `T` // `>`操作符不能用于类型`T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
help: consider restricting type parameter `T` // 考虑对T进行类型上的限制 :
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ++++++++++++++++++++++
```

因为 T 可以是任何类型，但不是所有的类型都能进行比较，因此上面的错误中，编译器建议我们给 T 添加一个类型限制：使用 `std::cmp::PartialOrd` 特征（Trait）对 T 进行限制，该特征的目的就是让类型实现可比较的功能。

同样的函数，我们在运行`add` 泛型函数.
```rust
fn add<T>(a:T, b:T) -> T {
    a + b
}

fn main() {
    println!("add i8: {}", add(2i8, 3i8));
}
```
运行：
```rust
error[E0369]: cannot add `T` to `T`
   --> main1.rs:131:7
    |
131 |     a + b
    |     - ^ - T
    |     |
    |     T
    |
help: consider restricting type parameter `T`
    |
130 | fn add<T: std::ops::Add>(a:T, b:T) -> T {
    |         +++++++++++++++

error: aborting due to previous error

For more information about this error, try `rustc --explain E0369`.
```

不是所有 T 类型都能进行相加操作，因此我们需要用 `std::ops::Add<Output = T>` 对 `T` 进行限制：
```rust
fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
    a + b
}
```
进行如上修改后，就可以正常运行。

### 结构体中使用泛型

结构体中的字段类型也可以用泛型来定义，我们在下面定义了一个坐标点 Point，它可以存放任何类型的坐标值：
```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

这里有两点需要特别的注意：

* 提前声明，跟泛型函数定义类似，首先我们在使用泛型参数之前必需要进行声明 `Point<T>`，接着就可以在结构体的字段类型中使用 T 来替代具体的类型
* `x` 和 y 是相同的类型

第二点非常重要，如果使用不同的类型，那么它会导致下面代码的报错：

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let p = Point{x: 1, y :1.1};
}
```

错误如下：
```rust
error[E0308]: mismatched types //类型不匹配
 --> src/main.rs:7:28
  |
7 |     let p = Point{x: 1, y :1.1};
  |                            ^^^ expected integer, found floating-point number //期望y是整数，但是却是浮点数

```

当把 1 赋值给 x 时，变量 p 的 T 类型就被确定为整数类型，因此 y 也必须是整数类型，但是我们却给它赋予了浮点数，因此导致报错。

如果想让 x 和 y 既能类型相同，又能类型不同，就需要使用不同的泛型参数：

```rust
struct Point<T,U> {
    x: T,
    y: U,
}
fn main() {
    let p = Point{x: 1, y :1.1};
}
```
因此这里需要注意的是，所有的泛型参数都要提前声明：Point<T,U> ! 