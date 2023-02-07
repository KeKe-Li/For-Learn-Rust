### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 特征 Trait

通常，rust中文件操作主要包含四个：open 、write、read、close，这些操作可以发生在硬盘，可以发生在内存，还可以发生在网络IO等等。总之如果你要为每一种情况都单独实现一套代码，那这种实现将过于繁杂，而且也没那个必要。

要解决上述问题，需要把这些行为抽象出来，就要使用 Rust 中的特征 trait 概念。

特征 Trait,定义了一个可以被共享的行为，只要实现了特征，你就能使用该行为。

#### 定义特征

如果不同的类型具有相同的行为，那么我们就可以定义一个特征，然后为这些类型实现该特征。定义特征是把一些方法组合在一起，目的是定义一个实现某些目标所必需的行为的集合。

定义一个特征 Trait:
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```
这里使用 trait 关键字来声明一个特征，Summary 是特征名。在大括号中定义了该特征的所有方法，在这个例子中是： `fn summarize(&self) -> String`。

特征只定义行为看起来是什么样的，而不定义行为具体是怎么样的。因此，我们只定义特征方法的签名，而不进行实现，此时方法签名结尾是 `;`，而不是一个 `{}`。

接下来，每一个实现这个特征的类型都需要具体实现该特征的相应方法，编译器也会确保任何实现 Summary 特征的类型都拥有与这个签名的定义完全一致的 summarize 方法。

#### 为类型实现特征

因为特征只定义行为看起来是什么样的，因此我们需要为类型实现具体的特征，定义行为具体是怎么样的。

首先来为 Post 和 Weibo 实现 Summary 特征：
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
}

impl Summary for Post {
    fn summarize(&self) -> String {
        format!("文章{}, 作者是{}", self.title, self.author)
    }
}

pub struct Weibo {
    pub username: String,
    pub content: String
}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```

实现特征的语法与为结构体、枚举实现方法很像：`impl Summary for Post`，读作“为 `Post` 类型实现 `Summary` 特征”，然后在 impl 的花括号中实现该特征的具体方法。


接下来就可以在这个类型上调用特征的方法：
```rust
fn main() {
    let post = Post{title: "Rust语言简介".to_string(),author: "Sunface".to_string(), content: "Rust棒极了!".to_string()};
    let weibo = Weibo{username: "sunface".to_string(),content: "好用".to_string()};

    println!("{}",post.summarize());
    println!("{}",weibo.summarize());
}
```

运行输出：
```rust
文章 Rust 语言简介, 作者是Sunface
sunface发表了微博好用
```
#### 特征定义与实现的位置(孤儿规则)

上面我们将 Summary 定义成了 pub 公开的。这样，如果他人想要使用我们的 Summary 特征，则可以引入到他们的包中，然后再进行实现。

关于特征实现与定义的位置，有一条非常重要的原则：如果你想要为类型 A 实现特征 T，那么 A 或者 T 至少有一个是在当前作用域中定义的！例如我们可以为上面的 Post 类型实现标准库中的 Display 特征，这是因为 Post 类型定义在当前的作用域中。同时，我们也可以在当前包中为 String 类型实现 Summary 特征，因为 Summary 定义在当前作用域中。

但是你无法在当前作用域中，为 String 类型实现 Display 特征，因为它们俩都定义在标准库中，其定义所在的位置都不在当前作用域。

该规则被称为孤儿规则，可以确保其它人编写的代码不会破坏你的代码，也确保了你不会莫名其妙就破坏了风马牛不相及的代码。

#### 默认实现

你可以在特征中定义具有默认实现的方法，这样其它类型无需再实现该方法，或者也可以选择重载该方法：
```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```
上面为 Summary 定义了一个默认实现，下面我们编写段代码来测试下：
```rust
impl Summary for Post {}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```
可以看到，Post 选择了默认实现，而 Weibo 重载了该方法，调用和输出如下：
```rust
println!("{}",post.summarize());
println!("{}",weibo.summarize());
```

```rust
(Read more...)
sunface发表了微博好用
```

默认实现允许调用相同特征中的其他方法，哪怕这些方法没有默认实现。如此，特征可以提供很多有用的功能而只需要实现指定的一小部分内容。例如，我们可以定义 Summary 特征，使其具有一个需要实现的 summarize_author 方法，然后定义一个 summarize 方法，此方法的默认实现调用 summarize_author 方法：

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

为了使用 Summary，只需要实现 summarize_author 方法即可：

```rust
impl Summary for Weibo {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
println!("1 new weibo: {}", weibo.summarize());
```

`weibo.summarize()` 会先调用 Summary 特征默认实现的 summarize 方法，通过该方法进而调用 Weibo 为 Summary 实现的 `summarize_author` 方法，最终输出：`1 new weibo: (Read more from keke)`。

#### 使用特征作为函数参数

特征如果仅仅是用来实现方法，那真的有些大材小用，现在我们来讲下，真正可以让特征大放光彩的地方。

现在，先定义一个函数，使用特征作为函数参数：

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

`impl Summary`，简直太贴切了，故名思义，它的意思是 实现了Summary特征 的 item 参数。

你可以使用任何实现了 Summary 特征的类型作为该函数的参数，同时在函数体内，还可以调用该特征的方法，例如 summarize 方法。具体的说，可以传递 Post 或 Weibo 的实例来作为参数，而其它类如 String 或者 i32 的类型则不能用做该函数的参数，因为它们没有实现 Summary 特征。

#### 特征约束(trait bound)

虽然 impl Trait 这种语法非常好理解，但是实际上它只是一个语法糖：

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

真正的完整书写形式如上所述，形如 `T: Summary` 被称为特征约束。

在简单的场景下 `impl Trait` 这种语法糖就足够使用，但是对于复杂的场景，特征约束可以让我们拥有更大的灵活性和语法表现能力，例如一个函数接受两个 `impl Summary` 的参数：

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
```

如果函数两个参数是不同的类型，那么上面的方法很好，只要这两个类型都实现了 Summary 特征即可。但是如果我们想要强制函数的两个参数是同一类型呢？上面的语法就无法做到这种限制，此时我们只能使特征约束来实现：

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```
泛型类型 T 说明了 item1 和 item2 必须拥有同样的类型，同时 T: Summary 说明了 T 必须实现 Summary 特征。

#### 多重约束

除了单个约束条件，我们还可以指定多个约束条件，例如除了让参数实现 Summary 特征外，还可以让参数实现 Display 特征以控制它的格式化输出：
```rust
pub fn notify(item: &(impl Summary + Display)) {}
```

除了上述的语法糖形式，还能使用特征约束的形式：

```rust
pub fn notify<T: Summary + Display>(item: &T) {}
```

通过这两个特征，就可以使用 `item.summarize` 方法，以及通过 `println!("{}", item)` 来格式化输出 item。

#### Where 约束

当特征约束变得很多时，函数的签名将变得很复杂：

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}
```

严格来说，上面的例子还是不够复杂，但是我们还是能对其做一些形式上的改进，通过 where：

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

#### 使用特征约束有条件地实现方法或特征

特征约束，可以让我们在指定类型 + 指定特征的条件下去实现方法，例如：

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```


cmp_display 方法，并不是所有的 Pair<T> 结构体对象都可以拥有，只有 T 同时实现了 Display + PartialOrd 的 Pair<T> 才可以拥有此方法。 该函数可读性会更好，因为泛型参数、参数、返回值都在一起，可以快速的阅读，同时每个泛型参数的特征也在新的代码行中通过特征约束进行了约束。

也可以有条件地实现特征, 例如，标准库为任何实现了 Display 特征的类型实现了 ToString 特征：
```rust
impl<T: Display> ToString for T {
    // --snip--
}    
```
我们可以对任何实现了 Display 特征的类型调用由 ToString 定义的 `to_string` 方法。例如，可以将整型转换为对应的 String 值，因为整型实现了 Display：
```rust
let s = 3.to_string();    
```    
    
#### 函数返回中的 impl Trait
    
可以通过 `impl Trait` 来说明一个函数返回了一个类型，该类型实现了某个特征：
```rust
 fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("sunface"),
        content: String::from(
            "m1 max太厉害了",
        )
    }
}   
```    
因为 Weibo 实现了 Summary，因此这里可以用它来作为返回值。要注意的是，虽然我们知道这里是一个 Weibo 类型，但是对于 `returns_summarizable` 的调用者而言，他只知道返回了一个实现了 Summary 特征的对象，但是并不知道返回了一个 Weibo 类型。

这种 `impl Trait` 形式的返回值，在一种场景下非常非常有用，那就是返回的真实类型非常复杂，你不知道该怎么声明时(毕竟 Rust 要求你必须标出所有的类型)，此时就可以用 `impl Trait` 的方式简单返回。例如，闭包和迭代器就是很复杂，只有编译器才知道那玩意的真实类型，好在你可以用 `impl Iterator` 来告诉调用者，返回了一个迭代器，因为所有迭代器都会实现 Iterator 特征。


但是这种返回值方式有一个很大的限制：只能有一个具体的类型，例如：
```rust

fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
            title: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Weibo {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
        }
    }
}
```
以上的代码就无法通过编译，因为它返回了两个不同的类型 Post 和 Weibo。
```rust
`if` and `else` have incompatible types
expected struct `Post`, found struct `Weibo`
```
报错提示我们 if 和 else 返回了不同的类型。
    
#### 调用方法需要引入特征
    
在一些场景中，使用 as 关键字做类型转换会有比较大的限制，因为你想要在类型转换上拥有完全的控制，例如处理转换错误，那么你将需要 TryInto：

```rust
use std::convert::TryInto;

fn main() {
  let a: i32 = 10;
  let b: u16 = 100;

  let b_ = b.try_into()
            .unwrap();

  if a < b_ {
    println!("Ten is less than one hundred.");
  }
}  
```  
代码中引入了 `std::convert::TryInto` 特征，但是却没有使用它，可能这里有些困惑，主要原因在于如果你要使用一个特征的方法，那么你需要将该特征引入当前的作用域中，我们在上面用到了 `try_into` 方法，因此需要引入对应的特征。

但是 Rust 又提供了一个非常便利的办法，即把最常用的标准库中的特征通过 `std::prelude` 模块提前引入到当前作用域中，其中包括了 `std::convert::TryInto`，你可以尝试删除第一行的代码 `use ...`，看看是否会报错。
           
           
 #### 应用示例
  
 1. 为自定义类型实现操作 
           
 在 Rust 中除了数值类型的加法，String 也可以做加法，因为 Rust 为该类型实现了 ·std::ops::Add· 特征，同理，如果我们为自定义类型实现了该特征，那就可以自己实现 Point1 + Point2 的操作:

```rust
 use std::ops::Add;

// 为Point结构体派生Debug特征，用于格式化输出
#[derive(Debug)]
struct Point<T: Add<T, Output = T>> { //限制类型T必须实现了Add特征，否则无法进行+操作。
    x: T,
    y: T,
}

impl<T: Add<T, Output = T>> Add for Point<T> {
    type Output = Point<T>;

    fn add(self, p: Point<T>) -> Point<T> {
        Point{
            x: self.x + p.x,
            y: self.y + p.y,
        }
    }
}

fn add<T: Add<T, Output=T>>(a:T, b:T) -> T {
    a + b
}

fn main() {
    let p1 = Point{x: 1.1f32, y: 1.1f32};
    let p2 = Point{x: 2.1f32, y: 2.1f32};
    println!("{:?}", add(p1, p2));

    let p3 = Point{x: 1i32, y: 1i32};
    let p4 = Point{x: 2i32, y: 2i32};
    println!("{:?}", add(p3, p4));
}          
```           

2. 自定义类型的打印输出

在开发过程中，往往只要使用 #[derive(Debug)] 对我们的自定义类型进行标注，即可实现打印输出的功能：
    
```rust
 #[derive(Debug)]
struct Point{
    x: i32,
    y: i32
}
fn main() {
    let p = Point{x:3,y:3};
    println!("{:?}",p);
}   
```    
    
但是在实际项目中，往往需要对我们的自定义类型进行自定义的格式化输出，以让用户更好的阅读理解我们的类型，此时就要为自定义类型实现 `std::fmt::Display` 特征：
    
 ```rust
 #![allow(dead_code)]

use std::fmt;
use std::fmt::{Display};

#[derive(Debug,PartialEq)]
enum FileState {
  Open,
  Closed,
}

#[derive(Debug)]
struct File {
  name: String,
  data: Vec<u8>,
  state: FileState,
}

impl Display for FileState {
   fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
     match *self {
         FileState::Open => write!(f, "OPEN"),
         FileState::Closed => write!(f, "CLOSED"),
     }
   }
}

impl Display for File {
   fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
      write!(f, "<{} ({})>",
             self.name, self.state)
   }
}

impl File {
  fn new(name: &str) -> File {
    File {
        name: String::from(name),
        data: Vec::new(),
        state: FileState::Closed,
    }
  }
}

fn main() {
  let f6 = File::new("f6.txt");
  //...
  println!("{:?}", f6);
  println!("{}", f6);
}   
 ```   
