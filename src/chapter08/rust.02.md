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





