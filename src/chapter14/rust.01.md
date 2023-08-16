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
它们仅应该被用于输出错误信息和进度信息，其它场景都应该使用 `print!` 系列。


#### {} 与

与其它语言常用的 `%d`，`%s` 不同，Rust 特立独行地选择了 `{}` 作为格式化占位符，事实证明，这种选择非常正确，它帮助用户减少了很多使用成本，你无需再为特定的类型选择特定的占位符，统一用 {} 来替代即可，剩下的类型推导等细节只要交给 Rust 去做。

与 `{}` 类似，`{:?}` 也是占位符：

* `{}` 适用于实现了 `std::fmt::Display` 特征的类型，用来以更优雅、更友好的方式格式化文本，例如展示给用户.
* `{:?}` 适用于实现了 `std::fmt::Debug` 特征的类型，用于调试场景.

其实两者的选择很简单，当你在写代码需要调试时，使用 `{:?}`，剩下的场景，选择 `{}`。


1. Debug 特征

事实上，为了方便我们调试，大多数 Rust 类型都实现了 Debug 特征或者支持派生该特征：

```markdown
#[derive(Debug)]
struct Person {
    name: String,
    age: u8
}

fn main() {
    let i = 3.1415926;
    let s = String::from("hello");
    let v = vec![1, 2, 3];
    let p = Person{name: "sunface".to_string(), age: 18};
    println!("{:?}, {:?}, {:?}, {:?}", i, s, v, p);
}
```
对于数值、字符串、数组，可以直接使用 `{:?}` 进行输出，但是对于结构体，需要派生Debug特征后，才能进行输出，总之很简单。

2. Display 特征

与大部分类型实现了 Debug 不同，实现了 Display 特征的 Rust 类型并没有那么多，往往需要我们自定义想要的格式化方式：
```markdown
let i = 3.1415926;
let s = String::from("hello");
let v = vec![1, 2, 3];
let p = Person {
    name: "sunface".to_string(),
    age: 18,
};
println!("{}, {}, {}, {}", i, s, v, p);
```
运行后可以看到 v 和 p 都无法通过编译，因为没有实现 Display 特征，但是你又不能像派生 Debug 一般派生 Display，只能另寻他法：

* 使用 `{:?}` 或 `{:#?}` .
* 为自定义类型实现 Display 特征.
* 使用 newtype 为外部类型实现 Display 特征.


`{:#?}` 与 `{:?}` 几乎一样，唯一的区别在于它能更优美地输出内容：
```markdown
// {:?}
[1, 2, 3], Person { name: "sunface", age: 18 }

// {:#?}
[
    1,
    2,
    3,
], Person {
    name: "sunface",
}
```
因此对于 Display 不支持的类型，可以考虑使用 `{:#?}` 进行格式化，虽然理论上它更适合进行调试输出。


* 为自定义类型实现 Display 特征

如果你的类型是定义在当前作用域中的，那么可以为其实现 Display 特征，即可用于格式化输出：

```markdown
struct Person {
    name: String,
    age: u8,
}

use std::fmt;
impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(
            f,
            "真棒哈哈!",
            self.name, self.age
        )
    }
}
fn main() {
    let p = Person {
        name: "sunface".to_string(),
        age: 18,
    };
    println!("{}", p);
}
```

如上所示，只要实现 Display 特征中的 fmt 方法，即可为自定义结构体 Person 添加自定义输出：
```markdown
真棒哈哈!
```

* 为外部类型实现 Display 特征

在 Rust 中，无法直接为外部类型实现外部特征，但是可以使用newtype解决此问题：
```markdown
struct Array(Vec<i32>);

use std::fmt;
impl fmt::Display for Array {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "数组是：{:?}", self.0)
    }
}
fn main() {
    let arr = Array(vec![1, 2, 3]);
    println!("{}", arr);
}
```

Array 就是我们的 newtype，它将想要格式化输出的 Vec 包裹在内，最后只要为 Array 实现 Display 特征，即可进行格式化输出：
```markdown
数组是：[1, 2, 3]
```

#### 位置参数

除了按照依次顺序使用值去替换占位符之外，还能让指定位置的参数去替换某个占位符，例如 {1}，表示用第二个参数替换该占位符(索引从 0 开始)：
```markdown
fn main() {
    println!("{}{}", 1, 2); // =>"12"
    println!("{1}{0}", 1, 2); // =>"21"
    // => Alice, this is Bob. Bob, this is Alice
    println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");
    println!("{1}{}{0}{}", 1, 2); // => 2112
}
```

#### 具名参数

除了像上面那样指定位置外，我们还可以为参数指定名称：

```markdown
fn main() {
    println!("{argument}", argument = "test"); // => "test"
    println!("{name} {}", 1, name = 2); // => "2 1"
    println!("{a} {c} {b}", a = "a", b = 'b', c = 3); // => "a 3 b"
}
```
需要注意的是：带名称的参数必须放在不带名称参数的后面，例如下面代码将报错：
```markdown
println!("{abc} {1}", abc = "def", 2);
```

```markdown
error: positional arguments cannot follow named arguments
--> src/main.rs:4:36
|
4 | println!("{abc} {1}", abc = "def", 2);
|                             -----  ^ positional arguments must be before named arguments
|                             |
|                             named argument
```

#### 格式化参数

格式化输出，意味着对输出格式会有更多的要求，例如只输出浮点数的小数点后两位：

```markdown
fn main() {
    let v = 3.1415926;
    // Display => 3.14
    println!("{:.2}", v);
    // Debug => 3.14
    println!("{:.2?}", v);
}
```

上面代码只输出小数点后两位。同时我们还展示了 `{}` 和 `{:?}` 的用法，后面如无特殊区别，就只针对 {} 提供格式化参数说明。

接下来，让我们一起来看看 Rust 中有哪些格式化参数。

#### 宽度

宽度用来指示输出目标的长度，如果长度不够，则进行填充和对齐：


* 字符串填充

字符串格式化默认使用空格进行填充，并且进行左对齐。
```markdown
fn main() {
    //-----------------------------------
    // 以下全部输出 "Hello x    !"
    // 为"x"后面填充空格，补齐宽度5
    println!("Hello {:5}!", "x");
    // 使用参数5来指定宽度
    println!("Hello {:1$}!", "x", 5);
    // 使用x作为占位符输出内容，同时使用5作为宽度
    println!("Hello {1:0$}!", 5, "x");
    // 使用有名称的参数作为宽度
    println!("Hello {:width$}!", "x", width = 5);
    //-----------------------------------

    // 使用参数5为参数x指定宽度，同时在结尾输出参数5 => Hello x    !5
    println!("Hello {:1$}!{}", "x", 5);
}
```

* 数字填充:符号和 0

数字格式化默认也是使用空格进行填充，但与字符串左对齐不同的是，数字是右对齐。
```markdown
fn main() {
    // 宽度是5 => Hello     5!
    println!("Hello {:5}!", 5);
    // 显式的输出正号 => Hello +5!
    println!("Hello {:+}!", 5);
    // 宽度5，使用0进行填充 => Hello 00005!
    println!("Hello {:05}!", 5);
    // 负号也要占用一位宽度 => Hello -0005!
    println!("Hello {:05}!", -5);
}
```
