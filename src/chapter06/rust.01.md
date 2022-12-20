### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 模式匹配

模式匹配经常出现在函数式编程里，用于为复杂的类型系统提供一个轻松的解构能力。


#### match 和 if let

在 Rust 中，模式匹配最常用的就是 match 和 if let。

```rust
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        _ => println!("West"),
    };
}
```
在 match 中用三个匹配分支来完全覆盖枚举变量 Direction 的所有成员类型，有以下几点值得注意：

* match 的匹配必须要穷举出所有可能，因此这里用 _ 来代表未列出的所有可能性
* match 的每一个分支都必须是一个表达式，且所有分支的表达式最终返回值的类型必须相同
* `X | Y`，类似逻辑运算符 或，代表该分支可以匹配 X 也可以匹配 Y，只要满足一个即可

其实 match 跟其他语言中的 switch 非常像，`_` 类似于 switch 中的 default。

#### match 匹配

match 匹配的通用形式：
```rust
match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3
}
```

模式匹配：将模式与 target 进行匹配，即为模式匹配，而模式匹配不仅仅局限于 match，后面我们会详细阐述。

match 允许我们将一个值与一系列的模式相比较，并根据相匹配的模式执行对应的代码.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny =>  {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```
一个分支有两个部分：一个模式和针对该模式的处理代码。第一个分支的模式是 `Coin::Penny`，其后的 `=>` 运算符将模式和将要运行的代码分开。这里的代码就仅仅是表达式 1，不同分支之间使用逗号分隔。

当 match 表达式执行时，它将目标值 coin 按顺序依次与每一个分支的模式相比较，如果模式匹配了这个值，那么模式之后的代码将被执行。

如果模式并不匹配这个值，将继续执行下一个分支。

每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 match 表达式的返回值。如果分支有多行代码，那么需要用 `{}` 包裹，同时最后一行代码需要是一个表达式。


* 使用 match 表达式赋值

match 本身也是一个表达式，因此可以用它来赋值：

```rust
enum IpAddr {
   Ipv4,
   Ipv6
}

fn main() {
    let ip1 = IpAddr::Ipv6;
    let ip_str = match ip1 {
        IpAddr::Ipv4 => "127.0.0.1",
        _ => "::1",
    };

    println!("{}", ip_str);
}
```
因为这里匹配到 `_` 分支，所以将 `"::1"` 赋值给了 `ip_str`。

* 模式绑定

模式匹配的另外一个重要功能是从模式中取出绑定的值，例如：
```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState), // 25美分硬币
}
```

其中 `Coin::Quarter` 成员还存放了一个值：美国的某个州（因为在 1999 年到 2008 年间，美国在 25 美分(Quarter)硬币的背后为 50 个州印刷了不同的标记，其它硬币都没有这样的设计）。

我们希望在模式匹配中，获取到 25 美分硬币上刻印的州的名称：
```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

在匹配 `Coin::Quarter(state)` 模式时，我们把它内部存储的值绑定到了 state 变量上，因此 state 变量就是对应的 UsState 枚举类型。

* 穷尽匹配

这里需要注意的是，match 的匹配必须穷尽所有情况：
```rust
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
    };
}
```
这里没有处理 `Direction::West` 的情况，因此会报错：
```rust
error[E0004]: non-exhaustive patterns: `West` not covered // 非穷尽匹配，`West` 没有被覆盖
  --> src/main.rs:10:11
   |
1  | / enum Direction {
2  | |     East,
3  | |     West,
   | |     ---- not covered
4  | |     North,
5  | |     South,
6  | | }
   | |_- `Direction` defined here
...
10 |       match dire {
   |             ^^^^ pattern `West` not covered // 模式 `West` 没有被覆盖
   |
   = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
   = note: the matched value is of type `Direction`
```
Rust 编译器清晰地知道 match 中有哪些分支没有被覆盖, 这种行为能强制我们处理所有的可能性,这里不得不说Rust编译器的强大了。

* `_` 通配符

当我们不想在匹配时列出所有值的时候，可以使用 Rust 提供的一个特殊模式，例如，u8 可以拥有 0 到 255 的有效的值，但是我们只关心 1、3、5 和 7 这几个值，不想列出其它的 0、2、4、6、8、9 一直到 255 的值。

那么, 我们不必一个一个列出所有值, 因为可以使用特殊的模式`_`替代：

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```
通过将 `_` 其放置于其他分支后，`_` 将会匹配所有遗漏的值。`()` 表示返回单元类型与所有分支返回值的类型相同，所以当匹配到 `_` 后，就什么也不会发生。

#### if let 匹配

有时会遇到只有一个模式的值需要被处理，其它值直接忽略的场景，这里就需要if let 来匹配来。

```rust
if let Some(5) = v {
    println!("five");
}
```

注意：当你只要匹配一个条件，且忽略其他条件时就用 `if let` ，否则都用 match。

#### matches!宏

这里我们先介绍下宏，在 Rust 中，宏是一种特殊的语法，允许在编译时生成代码。宏与函数非常相似，但是在编译时会执行，而不是在运行时执行。这意味着，宏在编译时生成的代码将直接替换宏调用的位置。

宏分为两种类型：

* 宏定义（macro definition）：定义了宏的行为。
* 宏调用（macro invocation）：调用宏并生成代码。

宏有一些限制，例如无法访问运行时数据和无法执行任何动态调度。相反，宏主要用于生成静态代码，例如创建结构体字段、实现模式匹配和生成错误处理代码。

使用宏的一个常见方式是使用宏生成类型安全的包装器，例如用于封装环境变量的 `env!` 宏。

我们先定义一个宏：
```rust
macro_rules! create_function {
    // This macro takes an argument of designator `ident` and
    // creates a function named `$func_name`.
    // The `ident` designator is used for variable/function names.
    ($func_name:ident) => {
        fn $func_name() {
            // The `stringify!` macro converts an `ident` into a string.
            println!("You called {:?}()", stringify!($func_name))
        }
    };
}

// Create functions named `foo` and `bar`.
create_function!(foo);
create_function!(bar);

fn main() {
    foo();
    bar();
}
```
运行:
```rust
You called "foo"()
You called "bar"()
```

接下来我们看看，Rust 标准库中提供了一个非常实用的宏：`matches!`，它可以将一个表达式跟模式进行匹配，然后返回匹配的结果 `true or false`。

有一个动态数组，里面存有以下枚举：
```rust
enum MyEnum {
    Foo,
    Bar
}

fn main() {
    let v = vec![MyEnum::Foo,MyEnum::Bar,MyEnum::Foo];
}
```
现在如果想对 v 进行过滤，只保留类型是 `MyEnum::Foo` 的元素：

```rust
v.iter().filter(|x| x == MyEnum::Foo);
```
但是，实际上这行代码会报错，因为你无法将 x 直接跟一个枚举成员进行比较。所以这里最好需要使用 `matches!` 宏来进行匹配。

```rust
v.iter().filter(|x| matches!(x, MyEnum::Foo));
```

#### 变量覆盖

无论是 match 还是 if let，他们都可以在模式匹配时覆盖掉老的值，绑定新的值:
```rust
fn main() {
   let age = Some(50);
   println!("在匹配前，age是{:?}",age);
   if let Some(age) = age {
       println!("匹配出来的age是{}",age);
   }

   println!("在匹配后，age是{:?}",age);
}
```
运行:
```rust
在匹配前，age是Some(50)
匹配出来的age是50
在匹配后，age是Some(50)
```

可以看出在 `if let` 中，`=` 右边 `Some(i32)` 类型的 age 被左边 i32 类型的新 age 覆盖了，该覆盖一直持续到 `if let` 语句块的结束。

因此第三个 `println!` 输出的 age 依然是 `Some(i32)` 类型。

对于 match 类型也是如此:
```rust
fn main() {
    let age = Some(50);
    println!("在匹配前，age是{:?}",age);
    match age {
        Some(age) =>  println!("匹配出来的age是{}",age),
        _ => ()
    }
    println!("在匹配后，age是{:?}",age);
}
```

需要注意的是，match 中的变量覆盖其实不是那么的容易看出,所以这里需要注意下.


