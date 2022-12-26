### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 方法 Method

在 Rust 中，方法的概念也大差不差，往往和对象成对出现：
```rust
object.method()
```
通常在使用read方法时候，如果用函数的写法 read(f, buffer)，用方法的写法 f.read(buffer)。不过与其它语言 class 跟方法的联动使用不同（这里可能要修改下），Rust 的方法往往跟结构体、枚举、特征(Trait)一起使用。

#### 定义方法

Rust 使用 `impl` 来定义方法。 定义方法示例：
```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    // new是Circle的关联函数，因为它的第一个参数不是self，且new并不是关键字
    // 这种方法往往用于初始化当前结构体的实例
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }

    // Circle的方法，&self表示借用当前的Circle结构体
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

下面的图片将 Rust 方法定义与其它语言的方法定义做了对比：

<p align="center">
<img width="100%" align="center" src="src/images/5.png" />
</p>

通过这个对比可以看出，其它语言中所有定义都在 class 中，但是 Rust 的对象定义和方法定义是分离的，这种数据和使用分离的方式，会给予使用者极高的灵活度。

我们在看一个新的应用示例:
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

`·`#[derive(Debug)]` 是 Rust 中的一种语法，它允许 Rust 编译器为类型生成一个名为 Debug 的 trait 的实现。

`Debug trait` 为类型提供了一种调试用的方法，用于将类型转换为人类可读的字符串。这在调试代码或者打印调试信息时非常有用。

该例子定义了一个 Rectangle 结构体，并且在其上定义了一个 area 方法，用于计算该矩形的面积。

`impl Rectangle {}` 表示为 Rectangle 实现方法(impl 是实现 implementation 的缩写)，这样的写法表明 impl 语句块中的一切都是跟 Rectangle 相关联的。

#### self、&self 和 &mut self

在上面 area 的签名中，我们使用 `&self` 替代 `rectangle: &Rectangle`，`&self` 其实是 `self: &Self` 的简写（注意大小写）。

在一个 impl 块内，Self 指代被实现方法的结构体类型，self 指代此类型的实例，换句话说，self 指代的是 Rectangle 结构体实例，这样的写法会让我们的代码简洁很多，而且非常便于理解：我们为哪个结构体实现方法，那么 self 就是指代哪个结构体的实例。

需要注意的是，self 依然有所有权的概念：

* self 表示 Rectangle 的所有权转移到该方法中，这种形式用的较少.
* `&self` 表示该方法对 Rectangle 的不可变借用.
* `&mut self` 表示可变借用.

总之，self 的使用就跟函数参数一样，要严格遵守 Rust 的所有权规则。

回到上面示例，选择 `&self` 的理由跟在函数中使用 `&Rectangle` 是相同的：我们并不想获取所有权，也无需去改变它，只是希望能够读取结构体中的数据。

如果想要在方法中去改变当前的结构体，需要将第一个参数改为 `&mut self`。仅仅通过使用 self 作为第一个参数来使方法获取实例的所有权是很少见的，这种使用方式往往用于把当前的对象转成另外一个对象时使用，转换完后，就不再关注之前的对象，且可以防止对之前对象的误调用。

简单总结下，使用方法代替函数有以下好处：

* 不用在函数签名中重复书写 self 对应的类型

* 代码的组织性和内聚性更强，对于代码维护和阅读来说，好处巨大

#### 方法名跟结构体字段名相同

在 Rust 中，允许方法名跟结构体的字段名相同：

```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
```
当我们使用 `rect1.width()` 时，Rust 知道我们调用的是它的方法，如果使用 `rect1.width`，则是访问它的字段

一般来说，方法跟字段同名，往往适用于实现 getter 访问器，例如:

```rust
pub struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    pub fn new(width: u32, height: u32) -> Self {
        Rectangle { width, height }
    }
    pub fn width(&self) -> u32 {
        return self.width;
    }
}

fn main() {
    let rect1 = Rectangle::new(30, 50);

    println!("{}", rect1.width());
}
```
用这种方式，我们可以把 Rectangle 的字段设置为私有属性，只需把它的 new 和 width 方法设置为公开可见，那么用户就可以创建一个矩形，同时通过访问器 `rect1.width()` 方法来获取矩形的宽度，因为 width 字段是私有的，当用户访问 `rect1.width` 字段时，就会报错。注意在此例中，Self 指代的就是被实现方法的结构体 Rectangle。

#### 关联函数

如何为一个结构体定义一个构造器方法？也就是接受几个参数，然后构造并返回该结构体的实例。其实这个很简单，参数中不包含 `self` 即可。

这种定义在 impl 中且没有 self 的函数被称之为关联函数： 因为它没有 self，不能用 `f.read()` 的形式调用，因此它是一个函数而不是方法，它又在 impl 中，与结构体紧密关联，因此称为关联函数。

通常，我们已经多次使用过关联函数，例如 `String::from`，用于创建一个动态字符串。

```rust
impl Rectangle {
    fn new(w: u32, h: u32) -> Rectangle {
        Rectangle { width: w, height: h }
    }
}
```

Rust 中有一个约定俗成的规则，使用 new 来作为构造器的名称，出于设计上的考虑，Rust 特地没有用 new 作为关键字.

因为是函数，所以不能用 `.` 的方式来调用，我们需要用 `::` 来调用，例如 `let sq = Rectangle::new(3, 3);`。这个方法位于结构体的命名空间中：`::` 语法用于关联函数和模块创建的命名空间。

