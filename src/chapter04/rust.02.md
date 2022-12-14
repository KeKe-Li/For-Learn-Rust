### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 复合类型

#### 元组

元组是由多种类型组合到一起形成的，因此它是复合类型，元组的长度是固定的，元组中元素的顺序也是固定的。

可以通过以下语法创建一个元组：
```rust
fn main() {
    let tup: (i32, f64, u8) = (100, 6.8, 2);
}
```
变量 tup 被绑定了一个元组值 `(500, 6.4, 1)`，该元组的类型是 `(i32, f64, u8)`，这里可以看到元组是用括号将多个类型组合到一起. 可以使用模式匹配或者 . 操作符来获取元组中的值。

* 用模式匹配解构元组
```rust
fn main() {
    let tup = (100, 6.8, 2);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

这里创建一个元组，然后将其绑定到 tup 上，接着使用 `let (x, y, z) = tup;` 来完成一次模式匹配，因为元组是 `(n1, n2, n3) `形式的，因此我们用一模一样的 `(x, y, z)` 形式来进行匹配，元组中对应的值会绑定到变量 x， y， z上。

这里用到到就是解构：用同样的形式把一个复杂对象中的值匹配出来。

* 用 `.` 来访问元组

模式匹配可以让我们一次性把元组中的值全部或者部分获取出来，如果只想要访问某个特定元素，那模式匹配就略显繁琐，对此，Rust 提供了 `.` 的访问方式：

```rust
fn main() {
    let x: (i32, f64, u8) = (100, 6.8, 2);

    let  m = x.0;

    let  n = x.1;

    let  q = x.2;
}
```
和其它语言的数组、字符串一样，元组的索引从 0 开始。

示例:
```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length)
}
```

这里可以看到`calculate_length` 函数接收 `s1` 字符串的所有权，然后计算字符串的长度，接着把字符串所有权和字符串长度再返回给 `s2` 和 len 变量。


#### 结构体

结构体跟元组有些相像：都是由多种类型组合而成。但是与元组不同的是，结构体可以为内部的每个字段起一个富有含义的名称。因此结构体更加灵活更加强大，无需依赖这些字段的顺序来访问和解析它们。

* 定义结构体

一个结构体由几部分组成：

* 通过关键字 struct 定义
* 一个清晰明确的结构体 名称
* 几个有名字的结构体 字段

我们定义一个结构体:

```rust
struct User {
    username: String,
    email: String,
    age: u8,
    sign_in_count: u64,
}
```
这里的，`username` 代表了用户名，是一个可变的 String 类型。

*创建结构体实例

```rust
let user1 = User {
        username: String::from("keke"),
        email: String::from("keke@gmail.com"),
        age: 18,
        sign_in_count: 1,
 };
```
这里需要注意的是:

有几点值得注意:

1. 初始化实例时，每个字段都需要进行初始化.
2. 初始化时的字段顺序不需要和结构体定义时的顺序一致.


* 访问结构体字段

创建了结构体我们就来访问下结构体:

通过 `.` 操作符即可访问结构体实例内部的字段值，也可以修改它们：

```rust
let mut user1 = User {
    username: String::from("keke"),
    email: String::from("keke@gmail.com"),
    age: 18,
    sign_in_count: 1,
 };
 user1.email = String::from("keke@gmail.com");
```

需要注意的是，必须要将结构体实例声明为可变的，才能修改其中的字段，Rust 不支持将某个结构体某个字段标记为可变。

* 简化结构体创建

```rust
fn build_user(email: String, username: String) -> User {
    User {
        username: username,
        email: email,
        age: 18,
        sign_in_count: 1,
    }
}
```

* 结构体更新语法

在实际场景中，有一种情况很常见：根据已有的结构体实例，创建新的结构体实例，例如根据已有的 user1 实例来构建 user2：
```rust
let user2 = User {
    username: user1.username,
    email: String::from("keke@qq.com"),
    age: user1.age,
    sign_in_count: user1.sign_in_count,
};
```
如果其他的字段保持不变，只更新变化的字段：

```rust
let user2 = User {
    email: String::from("keke@qq.com"),
    ..user1
};
```

因为 user2 仅仅在 email 上与 user1 不同，因此我们只需要对 email 进行赋值，剩下的通过结构体更新语法 `..user1` 即可完成。

`..` 语法表明凡是我们没有显式声明的字段，全部从 user1 中自动获取。需要注意的是 `..user1` 必须在结构体的尾部使用。

结构体更新语法跟赋值语句 = 非常相像，因此在user1 的部分字段所有权被转移到 user2 中：username 字段发生了所有权转移，作为结果，user1 无法再被使用。

看到这里我们看到，明明有三个字段进行了自动赋值，为何只有 username 发生了所有权转移？

这是因为 Copy 特征：实现了 Copy 特征的类型无需所有权转移，可以直接在赋值时进行 数据拷贝，其中 bool 和 u64 类型就实现了 Copy 特征，因此 age 和 sign_in_count 字段在赋值给 user2 时，仅仅发生了拷贝，而不是所有权转移。

这里需要注意的是：username 所有权被转移给了 user2，导致了 user1 无法再被使用，但是并不代表 user1 内部的其它字段不能被继续使用，例如：
```rust
let user1 = User {
    username: String::from("keke"),
    email: String::from("keke@gmail.com"),
    age: 18,
    sign_in_count: 1,
};
let user2 = User {
    username: user1.username,
    email: String::from("keke@qq.com"),
    age: user1.age,
    sign_in_count: user1.sign_in_count,
};
println!("{}", user1.age);
// 下面这行会报错
println!("{:?}", user1);
```

#### 结构体的内存排列

```rust
#[derive(Debug)]
struct File {
    name: String,
    data: Vec<u8>,
}

fn main() {
    let f1 = File {
     name: String::from("f1.txt"),
     data: Vec::new(),
    };
    
    let f1_name = &f1.name;
    let f1_length = &f1.data.len();
    
    println!("{:?}", f1);
    println!("{} is {} bytes long", f1_name, f1_length);
}
```
定义的 File 结构体在内存中的排列如下图所示：

<p align="center">
    <img width="100%" align="center" src="src/images/4.png" />
</p>

这里可以清晰的看出 File 结构体两个字段 name 和 data 分别拥有底层两个 `[u8]` 数组的所有权(String 类型的底层也是 `[u8]` 数组)，通过 `ptr` 指针指向底层数组的内存地址，这里你可以把 `ptr` 指针理解为 Rust 中的引用类型。

这里也侧面印证了：把结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段。

#### 结构体数据的所有权

在之前的 User 结构体的定义中，有一处细节：我们使用了自身拥有所有权的 String 类型而不是基于引用的 &str 字符串切片类型。这是一个有意而为之的选择：因为我们想要这个结构体拥有它所有的数据，而不是从其它地方借用数据。

你也可以让 User 结构体从其它对象借用数据，不过这么做，就需要引入生命周期(lifetimes)这个新概念（也是一个复杂的概念），简而言之，生命周期能确保结构体的作用范围要比它所借用的数据的作用范围要小。

总之，如果你想在结构体中使用一个引用，就必须加上生命周期，否则就会报错：
```rust
struct User {
    username: &str,
    email: &str,
    age: u8,
    sign_in_count: u64,
}


fn main() {
    let user1 = User {
        username: String::from("keke"),
        email: String::from("keke@gmail.com"),
        age: 18,
        sign_in_count: 1,
    };
}
```
运行:
```rust
error[E0106]: missing lifetime specifier
 --> main2.rs:2:15
  |
2 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 ~     username: &'a str,
  |

error[E0106]: missing lifetime specifier
 --> main.rs:3:12
  |
3 |     email: &str,
  |            ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     username: &str,
3 ~     email: &'a str,
  |

error: aborting due to 2 previous errors

For more information about this error, try `rustc --explain E0106`.

```
编译器就会报错,它需要生命周期标识符.

之所以会这样，就是因为，`&str` 是不可变的量，为了修复这个问题，你需要使用可变的引用，或者使用字符串类型（String）而不是字符串字面量。 在 Rust 中，字符串字面量（string literals）通常被当作不可变的引用来使用。

```rust
use std::fmt;

struct User {
    username: String,
    email: String,
    age: u8,
    sign_in_count: u64,
}

impl fmt::Debug for User {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "User {{ username: {:?},email: {:?},  age: {:?},sign_in_count:{:?} }}", self.username, self.email,self.age,self.sign_in_count)
    }
}
fn main() {
    let user1 = User {
        username: String::from("keke"),
        email: String::from("keke@gmail.com"),
        age: 18,
        sign_in_count: 1,
    };

    println!("user1 = {:?}", user1);
    // dbg!(user1);
}
```
运行结果：
```bash
[main2.rs:32] user1 = User { username: "keke",email: "keke@gmail.com",  age: 18,sign_in_count:1 }
```

* 使用 `#[derive(Debug)]` 来打印结构体的信息

在代码中我们使用 `#[derive(Debug)]` 对结构体进行了标记，这样才能使用 `println!("{:?}", s);` 的方式对其进行打印输出.

如果我们没有使用 `#[derive(Debug)]`，在下面示例中：
```rust 
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let result = Rectangle {
        width: 30,
        height: 50,
    };

    println!("result is {}", result);
}
```
运行结果:
```bash
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
  --> main.rs:12:29
   |
12 |     println!("result is {}", result);
   |                             ^^^^^ `Rectangle` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
```
可以看到报错了，首先可以观察到，上面使用了 {} 而不是之前的 {:?}，运行后报错。

提示我们结构体 Rectangle 没有实现 Display 特征，这是因为如果我们使用 `{}` 来格式化输出，那对应的类型就必须实现 Display 特征。

以前学习的基本类型，都默认实现了该特征:
```rust
fn main() {
    let v = 10;
    let b = true;

    println!("{}, {}", v, b);
}
```

上面代码不会报错，那么结构体为什么不默认实现 Display 特征呢？

原因在于结构体较为复杂，类似的还有很多，由于这种复杂性，Rust 不希望猜测我们想要的是什么，而是把选择权交给我们自己来实现：如果要用 {} 的方式打印结构体，那就自己实现 Display 特征。

那么为了解决这个结构体打印问题，我们可以选择什么方式打印解决呢？
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let result = Rectangle {
        width: 30,
        height: 50,
    };

    println!("result is {:?}", result);
}
```
运行:
```rust
error[E0277]: `Rectangle` doesn't implement `Debug`
  --> main.rs:12:32
   |
12 |     println!("result is {:?}", result);
   |                                ^^^^^^ `Rectangle` cannot be formatted using `{:?}`
   |
   = help: the trait `Debug` is not implemented for `Rectangle`
   = note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
   = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider annotating `Rectangle` with `#[derive(Debug)]`
   |
1  | #[derive(Debug)]
   |

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
````


上面的运行错误中有提示我们，让我们实现 Debug 特征，因为我们就是不想实现 Display 特征，才用的 `{:?}`，怎么又要实现 Debug，但是仔细看，提示中有一行： `add #[derive(Debug)]` to Rectangle，那我们就使用下。

首先，Rust 默认不会为我们实现 Debug，为了实现，有两种方式可以选择：

* 手动实现
* 使用 derive 派生实现

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let result = Rectangle {
        width: 30,
        height: 50,
    };

    println!("result is {:?}", result);
}
```
运行:
```bash
result is Rectangle { width: 30, height: 50 }
```

这个输出格式看上去也可以，虽然未必是最好的。这种格式是 Rust 自动为我们提供的实现，看上基本就跟结构体的定义形式一样。

当结构体较大时，我们可能希望能够有更好的输出表现，此时可以使用 `{:#?}` 来替代 `{:?}`，输出如下:
```bash
result is Rectangle {
    width: 30,
    height: 50,
}
```
除此之外，还有一个简单的输出 debug 信息的方法，那就是使用 `dbg!` 宏，它会拿走表达式的所有权，然后打印出相应的文件名、行号等 debug 信息，当然还有我们需要的表达式的求值结果。除此之外，它最终还会把表达式值的所有权返回！

注意：`dbg!` 输出到标准错误输出 `stderr`，而 `println!` 输出到标准输出 `stdout`.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let result = Rectangle {
        width: 30,
        height: 50,
    };

    dbg!(&result);
}
```
运行:
```rust
[main3.rs:21] &result = Rectangle {
    width: 30,
    height: 50,
}
```
至此，可以看到，我们想要的 debug 信息几乎都有了：代码所在的文件名、行号、表达式以及表达式的值！



