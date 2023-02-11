### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 特征对象

有一段代码无法通过编译:
```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
           // ...
        }
    } else {
        Weibo {
            // ...
        }
    }
}
```

其中 Post 和 Weibo 都实现了 Summary 特征，因此上面的函数试图通过返回 impl Summary 来返回这两个类型，但是编译器却无情地报错了，原因是 impl Trait 的返回值类型并不支持多种不同的类型返回，那如果我们想返回多种类型，该怎么办？

那么现在考虑一个问题：做一款游戏，需要将多个对象渲染在屏幕上，这些对象属于不同的类型，存储在列表中，渲染的时候，需要循环该列表并顺序渲染每个对象，在 Rust 中该怎么实现？

可能我们最常想到是利用枚举：
```rust
#[derive(Debug)]
enum UiObject {
    Button,
    SelectBox,
}

fn main() {
    let objects = [
        UiObject::Button,
        UiObject::SelectBox
    ];

    for o in objects {
        draw(o)
    }
}

fn draw(o: UiObject) {
    println!("{:?}",o);
}
```
这个确实是一个办法，但是问题来了，如果你的对象集合并不能事先明确地知道呢？或者别人想要实现一个 UI 组件呢？此时枚举中的类型是有些缺少的，是不是还要修改你的代码增加一个枚举成员？

总之，在编写这个 UI 库时，我们无法知道所有的 UI 对象类型，只知道的是：

* UI 对象的类型不同
* 需要一个统一的类型来处理这些对象，无论是作为函数参数还是作为列表中的一员
* 需要对每一个对象调用 draw 方法
在拥有继承的语言中，可以定义一个名为 Component 的类，该类上有一个 draw 方法。其他的类比如 Button、Image 和 SelectBox 会从 Component 派生并因此继承 draw 方法。它们各自都可以覆盖 draw 方法来定义自己的行为，但是框架会把所有这些类型当作是 Component 的实例，并在其上调用 draw。

不过 Rust 并没有继承，我们可能需要想下其他办法。那就是特征对象定义。

#### 特征对象定义

为了解决上面的所有问题，Rust 引入了一个概念 —— 特征对象。

在介绍特征对象之前，先来为之前的 UI 组件定义一个特征：
```rust
pub trait Draw {
    fn draw(&self);
}
```

只要组件实现了 Draw 特征，就可以调用 draw 方法来进行渲染。假设有一个 Button 和 SelectBox 组件实现了 Draw 特征：
```rust

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // 绘制按钮的代码
    }
}

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // 绘制SelectBox的代码
    }
}
```
此时，还需要一个动态数组来存储这些 UI 对象：

```rust
pub struct Screen {
    pub components: Vec<?>,
}
```

注意到上面代码中的 `?` 吗？它的意思是：我们应该填入什么类型，可以说就之前学过的内容里，你找不到哪个类型可以填入这里，但是因为 Button 和 SelectBox 都实现了 Draw 特征，那我们是不是可以把 Draw 特征的对象作为类型，填入到数组中呢？答案是肯定的。

特征对象指向实现了 Draw 特征的类型的实例，也就是指向了 Button 或者 SelectBox 的实例，这种映射关系是存储在一张表中，可以在运行时通过特征对象找到具体调用的类型方法。

可以通过 `&`引用或者 `Box<T>` 智能指针的方式来创建特征对象。

```rust
trait Draw {
    fn draw(&self) -> String;
}

impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self)
    }
}

impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self)
    }
}

// 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
fn draw1(x: Box<dyn Draw>) {
    // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法
    x.draw();
}

fn draw2(x: &dyn Draw) {
    x.draw();
}

fn main() {
    let x = 1.1f64;
    // do_something(&x);
    let y = 8u8;

    // x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
    // 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上
    draw1(Box::new(x));
    // 基于 y 的值创建一个 Box<u8> 类型的智能指针
    draw1(Box::new(y));
    draw2(&x);
    draw2(&y);
}
```
上面代码，有几个非常重要的点：

* draw1 函数的参数是 `Box<dyn Draw>` 形式的特征对象，该特征对象是通过 `Box::new(x)` 的方式创建的.
* draw2 函数的参数是 `&dyn Draw` 形式的特征对象，该特征对象是通过 `&x` 的方式创建的.
* dyn 关键字只用在特征对象的类型声明上，在创建时无需使用 dyn.

因此，可以使用特征对象来代表泛型或具体的类型。

