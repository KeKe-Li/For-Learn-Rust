### rust 学习

Rust 是一门全新的语言，它可能会带给你前所未有的体验，提升你的通用编程水平，甚至于赋予你全新的编程思想。但是难度也是有的，需要不断学习。

#### 变量绑定与解构

在大多数语言中，要么只支持声明可变的变量，要么只支持声明不可变的变量( 例如函数式语言 )，前者为编程提供了灵活性，后者为编程提供了安全性，而
Rust 比较开放的一点，选择了两者我都要，既要灵活性又要安全性。

下面看看，Rust命名规范：

基本的 Rust 命名规范在 `RFC 430` 中有描述。

通常，对于 type-level 的构造 Rust 倾向于使用驼峰命名法，而对于 value-level 的构造使用蛇形命名法。详情如下：

| 条目 | 惯例 |
|------|-----|
| 包 Crates | unclear |
| 模块 Modules | snake_case |
| 类型 Types | UpperCamelCase |
| 特征 Traits | UpperCamelCase |                                            
| 枚举 Enumerations | UpperCamelCase |         
| 结构体 Structs | UpperCamelCase |
| 函数 Functions | snake_case |
| 方法 Methods | snake_case |
| 通用构造器 General constructors | new or with_more_details |
| 转换构造器 Conversion constructors | from_some_other_type |
| 宏 Macros | snake_case!                               |
| 局部变量 Local variables | snake_case |
| 静态类型 Statics | SCREAMING_SNAKE_CASE |
| 常量 Constants | SCREAMING_SNAKE_CASE |
| 类型参数 Type parameters | UpperCamelCase，通常使用一个大写字母: T |
| 生命周期 Lifetimes | 通常使用小写字母: 'a，'de，'src |
| Features | unclear but see C-FEATURE |

对于驼峰命名法，复合词的缩略形式我们认为是一个单独的词语，所以只对首字母进行大写：使用 Uuid 而不是 UUID，Usize 而不是
USize，Stdin 而不是 StdIn。

对于蛇形命名法，缩略词用全小写：`is_start`。

对于蛇形命名法（包括全大写的 SCREAMING_SNAKE_CASE），除了最后一部分，其它部分的词语都不能由单个字母组成： `btree_map`
而不是 `b_tree_map`.

包名不应该使用 `-rs` 或者 `-rust` 作为后缀，因为每一个包都是 Rust 写的，因此这种多余的注释其实没有任何意义。

除此之外，类型转换要遵守 `as_`，`to_`，`into_` 命名惯例(C-CONV).

类型转换应该通过方法调用的方式实现，其中的前缀规则如下：

方法前缀 性能开销 所有权改变:

|as_    | Free | borrowed -> borrowed |
|-------|-----------|----------------------|
|to_    | Expensive | borrowed -> borrowed borrowed -> owned (non-Copy types) owned -> owned (Copy types)|
|into_    | Variable | owned -> owned (non-Copy types)|

例如：

`str::as_bytes()` 把 str 变成 UTF-8 字节数组，性能开销是 0。输入是一个借用的 &str，输出也是一个借用的 &str
`Path::to_str` 会执行一次昂贵的 UTF-8 字节数组检查，输入和输出都是借用的。对于这种情况，如果把方法命名为 `as_str`
是不正确的，因为这个方法的开销还挺大.
`str::to_lowercase()` 在调用过程中会遍历字符串的字符，且可能会分配新的内存对象。输入是一个借用的 str，输出是一个有独立所有权的
String.
`String::into_bytes()` 返回 String 底层的 `Vec<u8>` 数组，转换本身是零消耗的。该方法获取 String
的所有权，然后返回一个新的有独立所有权的 `Vec<u8>`
当一个单独的值被某个类型所包装时，访问该类型的内部值应通过 `into_inner()` 方法来访问。例如将一个缓冲区值包装为 BufReader
类型，还有 `GzDecoder`、`AtomicBool` 等，都是这种类型。


如果 mut 限定符在返回类型中出现，那么在命名上也应该体现出来。例如，`Vec::as_mut_slice` 就说明它返回了一个 mut 切片，在这种情况下 `as_mut_slice` 比 `as_slice_mut` 更适合。


#### 变量和常量之间的差异

变量的值不能更改可能让你想起其他另一个很多语言都有的编程概念：常量(constant)
。与不可变变量一样，常量也是绑定到一个常量名且不允许更改的值，但是常量和变量之间存在一些差异：

* 常量不允许使用 mut。常量不仅仅默认不可变，而且自始至终不可变，因为常量在编译完成后，已经确定它的值。
* 常量使用 const 关键字而不是 let 关键字来声明，并且值的类型必须标注。

下面是一个常量声明的例子，其常量名为 MAX_POINTS，值设置为 100,000。（Rust
常量的命名约定是全部字母都使用大写，并使用下划线分隔单词，另外对数字字面量可插入下划线以提高可读性）：

```rust
const MAX_POINTS: u32 = 100_000;
```

常量可以在任意作用域内声明，包括全局作用域，在声明的作用域内，常量在程序运行的整个过程中都有效。对于需要在多处代码共享一个不可变的值时非常有用。
<<<<<<< HEAD

#### 整数类型

整数是没有小数部分的数字。之前使用过的 i32 类型，表示有符号的 32 位整数（ i 是英文单词 integer 的首字母，与之相反的是
u，代表无符号 unsigned 类型）。下表显示了 Rust 中的内置的整数类型：

| 长度       | 有符号类型     | 无符号类型  |
|----------|--------------|--------|
| 8 位      | i8           | u8     |
| 16 位     | i16          | u16    |
| 32 位     | i32          | u32    |
| 64 位     | i64          | u64    |
| 128 位    | i128         | u128   |
| 视架构而定  | isize        | usize  |

类型定义的形式统一为：有无符号 + 类型大小(位数)。无符号数表示数字只能取正数，而有符号则表示数字既可以取正数又可以取负数。就像在纸上写数字一样：当要强调符号时，数字前面可以带上正号或负号；然而，当很明显确定数字为正数时，就不需要加上正号了。有符号数字以补码形式存储。

每个有符号类型规定的数字范围是 -(2n - 1) ~ 2n - 1 - 1，其中 n 是该定义形式的位长度。因此 i8 可存储数字范围是 -(27) ~ 27 - 1，即 -128 ~ 127。无符号类型可以存储的数字范围是 0 ~ 2n - 1，所以 u8 能够存储的数字为 0 ~ 28 - 1，即 0 ~ 255。

此外，isize 和 usize 类型取决于程序运行的计算机 CPU 类型： 若 CPU 是 32 位的，则这两个类型是 32 位的，同理，若 CPU 是 64 位，那么它们则是 64 位。

整形字面量可以用下表的形式书写：

|数字字面量       | 示例 |
|---------------|------|
|十进制	        |98_222|
|十六进制	        |0xff  |
|八进制	        |0o77  |
|二进制	        | 0b1111_0000 |
|字节 (仅限于 u8) | 	 b'A' |

这么多类型，有没有一个简单的使用准则？答案是肯定的， Rust 整型默认使用 i32，例如 let i = 1，那 i 就是 i32 类型，因此你可以首选它，同时该类型也往往是性能最好的。isize 和 usize 的主要应用场景是用作集合的索引。


#### 整型溢出


这里需要注意下的是有关整型溢出问题。

假设有一个 u8 ，它可以存放从 0 到 255 的值。那么当你将其修改为范围之外的值，比如 256，则会发生整型溢出。关于这一行为 Rust 有一些有趣的规则：当在 debug 模式编译时，Rust 会检查整型溢出，若存在这些问题，则使程序在编译时 panic(崩溃,Rust 使用这个术语来表明程序因错误而退出)。

在当使用 --release 参数进行 release 模式构建时，Rust 不检测溢出。相反，当检测到整型溢出时，Rust 会按照补码循环溢出（two’s complement wrapping）的规则处理。

简而言之，大于该类型最大值的数值会被补码转换成该类型能够支持的对应数字的最小值。比如在 u8 的情况下，256 变成 0，257 变成 1，依此类推。程序不会 panic，但是该变量的值可能不是你期望的值。依赖这种默认行为的代码都应该被认为是错误的代码。

要显式处理可能的溢出，可以使用标准库针对原始数字类型提供的这些方法：

使用 `wrapping_*` 方法在所有模式下都按照补码循环溢出规则处理，例如 `wrapping_add`.
如果使用 `checked_*` 方法时发生溢出，则返回 None 值
使用 `overflowing_*` 方法返回该值和一个指示是否存在溢出的布尔值
使用 `saturating_*` 方法使值达到最小值或最大值

示例：
```rust
fn main() {
    let a : u8 = 255;
    let b = a.wrapping_add(20);
    println!("{}", b);  // 19
}
```

#### 浮点类型

浮点类型数字 是带有小数点的数字，在 Rust 中浮点类型数字也有两种基本类型： f32 和 f64，分别为 32 位和 64 位大小。默认浮点类型是 f64，在现代的 CPU 中它的速度与 f32 几乎相同，但精度更高。

示例：
```rust
fn main() {
let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

浮点数根据 IEEE-754 标准实现。f32 类型是单精度浮点型，f64 为双精度。

浮点数陷阱
浮点数由于底层格式的特殊性，导致了如果在使用浮点数时不够谨慎，就可能造成危险，有两个原因：

浮点数往往是你想要数字的近似表达 浮点数类型是基于二进制实现的，但是我们想要计算的数字往往是基于十进制，例如 0.1 在二进制上并不存在精确的表达形式，但是在十进制上就存在。这种不匹配性导致一定的歧义性，更多的，虽然浮点数能代表真实的数值，但是由于底层格式问题，它往往受限于定长的浮点数精度，如果你想要表达完全精准的真实数字，只有使用无限精度的浮点数才行

浮点数在某些特性上是反直觉的 例如大家都会觉得浮点数可以进行比较，对吧？是的，它们确实可以使用 >，>= 等进行比较，但是在某些场景下，这种直觉上的比较特性反而会害了你。因为 f32 ， f64 上的比较运算实现的是 `std::cmp::PartialEq` 特征(类似其他语言的接口)，但是并没有实现 `std::cmp::Eq` 特征，但是后者在其它数值类型上都有定义.

Rust 的 HashMap 数据结构，是一个 KV 类型的 `Hash Map` 实现，它对于 K 没有特定类型的限制，但是要求能用作 K 的类型必须实现了 std::cmp::Eq 特征，因此这意味着你无法使用浮点数作为 HashMap 的 Key，来存储键值对，但是作为对比，Rust 的整数类型、字符串类型、布尔类型都实现了该特征，因此可以作为 HashMap 的 Key。

为了避免上面说的两个陷阱，你需要遵守以下准则：

* 避免在浮点数上测试相等性.

* 当结果在数学上可能存在未定义时，需要格外的小心.

#### 位运算

Rust的运算基本上和其他语言一样

| 运算符	 |  说明                             |
|--------|----------------------------------|
| & 位与	 |  相同位置均为1时则为1，否则为0         |
| | 位或	 |  相同位置只要有1时则为1，否则为0       |
| ^ 异或	 |  相同位置不相同则为1，相同则为0        |
| ! 位非	 |  把位中的0和1相互取反，即0置为1，1置为0 |
| << 左移 |	所有位向左移动指定位数，右位补0        |
| >> 右移 |	所有位向右移动指定位数，带符号移动（正数补0，负数补1） |

示例：
```rust
fn main() {
    // 二进制为00000010
    let a:i32 = 2;
    // 二进制为00000011
    let b:i32 = 3;

    println!("(a & b) value is {}", a & b);

    println!("(a | b) value is {}", a | b);

    println!("(a ^ b) value is {}", a ^ b);

    println!("(!b) value is {} ", !b);

    println!("(a << b) value is {}", a << b);

    println!("(a >> b) value is {}", a >> b);

    let mut a = a;
    // 注意这些计算符除了!之外都可以加上=进行赋值 (因为!=要用来判断不等于)
    a <<= b;
    println!("(a << b) value is {}", a);
}
```

#### 序列(Range)

Rust 提供了一个非常简洁的方式，用来生成连续的数值，例如 `1..5`，生成从 1 到 4 的连续数字，不包含 5；`1..=5`，生成从 1 到 5 的连续数字，包含 5，它的用途很简单，常常用于循环中：
```rust 
for i in 1..=5 {
println!("{}",i);
}
```

最终程序输出:
```bash
1
2
3
4
5
```
序列只允许用于数字或字符类型，原因是：它们可以连续，同时编译器在编译期可以检查该序列是否为空，字符和数字值是 Rust 中仅有的可以用于判断是否为空的类型。如下是一个使用字符类型序列的例子：
```rust
for i in 'a'..='z' {
    println!("{}",i);
}
```

#### 有理数和复数

Rust 的标准库相比其它语言，准入门槛较高，因此有理数和复数并未包含在标准库中：

* 有理数和复数
* 任意大小的整数和任意精度的浮点数
* 固定精度的十进制小数，常用于货币相关的场景

好在社区已经开发出高质量的 Rust 数值库：[num](https://crates.io/crates/num)。

按照以下步骤来引入 num 库：

* 创建新工程 `cargo new complex-num && cd complex-num`
* 在 `Cargo.toml` 中的 [dependencies] 下添加一行 `num = "0.4.0"`
* 将 `src/main.rs` 文件中的 main 函数替换为下面的代码
* 运行 `cargo run`

```rust
use num::complex::Complex;

fn main() {
    let a = Complex { re: 2.1, im: -1.2 };
    let b = Complex::new(11.1, 22.2);
    let result = a + b;
    println!("{} + {}i", result.re, result.im)
}
```
Rust 的数值类型和运算跟其他语言较为相似，但是实际上，除了语法上的不同之外，还是存在一些差异点：

* Rust 拥有相当多的数值类型. 因此你需要熟悉这些类型所占用的字节数，这样就知道该类型允许的大小范围以及你选择的类型是否能表达负数
* 类型转换必须是显式的. Rust 永远也不会偷偷把你的 `16bit` 整数转换成 `32bit` 整数
* Rust 的数值上可以使用方法. 例如你可以用以下方法来将 `13.14` 取整：`13.14_f32.round()`，在这里我们使用了类型后缀，因为编译器需要知道 13.14 的具体类型

在大多数语言中，要么只支持声明可变的变量，要么只支持声明不可变的变量( 例如函数式语言 )，前者为编程提供了灵活性，后者为编程提供了安全性，而 Rust 比较开放的一点，选择了两者我都要，既要灵活性又要安全性。

下面看看，Rust命名规范：

基本的 Rust 命名规范在 `RFC 430` 中有描述。

通常，对于 type-level 的构造 Rust 倾向于使用驼峰命名法，而对于 value-level 的构造使用蛇形命名法。详情如下：
|           条目                           |            惯例                             |
|-----------------------------------------|--------------------------------------------|
|       包 Crates	                      |  unclear                                   |
|       模块 Modules	                      |  snake_case                                |
|       类型 Types	                      |  UpperCamelCase                            |
|       特征 Traits	                      |  UpperCamelCase                            |                                            
|       枚举 Enumerations	              |  UpperCamelCase                            |         
|       结构体 Structs	                  |  UpperCamelCase                            |
|       函数 Functions	                  |  snake_case                                |
|       方法 Methods	                      |  snake_case                                |
|       通用构造器 General constructors	  |  new or with_more_details                  |
|       转换构造器 Conversion constructors  |	 from_some_other_type                      |
|       宏 Macros	                      |  snake_case!                               |
|       局部变量 Local variables	          |  snake_case                                |
|       静态类型 Statics	                  |  SCREAMING_SNAKE_CASE                      |
|       常量 Constants	                  |  SCREAMING_SNAKE_CASE                      |
|       类型参数 Type parameters	          |  UpperCamelCase，通常使用一个大写字母: T       |
|       生命周期 Lifetimes	              |  通常使用小写字母: 'a，'de，'src               |
|       Features	                      |  unclear but see C-FEATURE                 |

对于驼峰命名法，复合词的缩略形式我们认为是一个单独的词语，所以只对首字母进行大写：使用 Uuid 而不是 UUID，Usize 而不是 USize，Stdin 而不是 StdIn。

对于蛇形命名法，缩略词用全小写：`is_start`。

对于蛇形命名法（包括全大写的 SCREAMING_SNAKE_CASE），除了最后一部分，其它部分的词语都不能由单个字母组成： `btree_map` 而不是 `b_tree_map`.

包名不应该使用 `-rs` 或者 `-rust` 作为后缀，因为每一个包都是 Rust 写的，因此这种多余的注释其实没有任何意义。

除此之外，类型转换要遵守 `as_`，`to_`，`into_` 命名惯例(C-CONV).

类型转换应该通过方法调用的方式实现，其中的前缀规则如下：

方法前缀	性能开销	所有权改变
|as_	| Free	    | borrowed -> borrowed |
|-------|-----------|----------------------|
|to_	| Expensive | borrowed -> borrowed borrowed -> owned (non-Copy types) owned -> owned (Copy types)|
|into_	| Variable	| owned -> owned (non-Copy types)|

例如：

`str::as_bytes()` 把 str 变成 UTF-8 字节数组，性能开销是 0。输入是一个借用的 &str，输出也是一个借用的 &str
`Path::to_str` 会执行一次昂贵的 UTF-8 字节数组检查，输入和输出都是借用的。对于这种情况，如果把方法命名为 `as_str` 是不正确的，因为这个方法的开销还挺大.
`str::to_lowercase()` 在调用过程中会遍历字符串的字符，且可能会分配新的内存对象。输入是一个借用的 str，输出是一个有独立所有权的 String.
`String::into_bytes()` 返回 String 底层的 `Vec<u8>` 数组，转换本身是零消耗的。该方法获取 String 的所有权，然后返回一个新的有独立所有权的 `Vec<u8>`
当一个单独的值被某个类型所包装时，访问该类型的内部值应通过 `into_inner()` 方法来访问。例如将一个缓冲区值包装为 BufReader 类型，还有 `GzDecoder`、`AtomicBool` 等，都是这种类型。

如果 mut 限定符在返回类型中出现，那么在命名上也应该体现出来。例如，`Vec::as_mut_slice` 就说明它返回了一个 mut 切片，在这种情况下 `as_mut_slice` 比 `as_slice_mut` 更适合。


#### 语句和表达式

* 语句:

在函数返回中会经常出现表达式，这里需要弄明白一些。

Rust 的函数体是由一系列语句组成，最后由一个表达式来返回值，例如：
```rust
fn add_with_extra(x: i32, y: i32) -> i32 {
    let x = x + 1; // 语句
    let y = y + 5; // 语句
    x + y // 表达式
}
```

语句会执行一些操作但是不会返回一个值，而表达式会在求值后返回一个值，因此在上述函数体的三行代码中，前两行是语句，最后一行是表达式。

对于 Rust 语言而言，这种基于语句（statement）和表达式（expression）的方式是非常重要的，需要能明确的区分这两个概念, 但是对于很多其它语言而言，这两个往往无需区分。基于表达式是函数式语言的重要特征，表达式总要返回值。

其实，在此之前，我们已经多次使用过语句和表达式。

* 表达式:

表达式会进行求值，然后返回一个值。例如 5 + 6，在求值后，返回值 11，因此它就是一条表达式。

表达式可以成为语句的一部分，例如 `let y = 6` 中，6 就是一个表达式，它在求值后返回一个值 6（有些反直觉，但是确实是表达式）。

调用一个函数是表达式，因为会返回一个值，调用宏也是表达式，用花括号包裹最终返回一个值的语句块也是表达式，总之，能返回值，它就是表达式:

```rust 
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

#### 函数

声明函数的关键字 fn ,函数名 add()，参数 i 和 j，参数类型和返回值类型都是 i32.

```rust
 fn add(i: i32, j: i32) -> i32 {
   i + j
 }
 ```

<p align="center">
<img width="100%" align="center" src="src/images/1.png" />
</p>

函数需要注意的:
* 函数名和变量名使用蛇形命名法(snake case)，例如 `fn add_two() -> {}`
* 函数的位置可以随便放，Rust 不关心我们在哪里定义了函数，只要有定义即可
* 每个函数参数都需要标注类型


#### 函数返回

函数的返回值就是函数体最后一条表达式的返回值，当然我们也可以使用 return 提前返回，下面的函数使用最后一条表达式来返回一个值：

```rust
fn add_two(i: i32, j:i32) ->i32{
    return i+j
}
```

#### Rust中的特殊返回类型

无返回值()

例如单元类型 ()，是一个零长度的元组。它没啥作用，但是可以用来表达一个函数没有返回值：

* 函数没有返回值，那么返回一个 ()
* 通过 `;` 结尾的表达式返回一个 ()

例如下面的 report 函数会隐式返回一个 ()：
```rust 
use std::fmt::Debug;

fn report<T: Debug>(item: T) {
  println!("{:?}", item);

}
```

与上面的函数返回值相同，但是下面的函数显式的返回了 ()：

```rust 
fn clean(text: &mut String) -> () {
  *text = String::from("");
}
```

在实际编程中，你会经常在错误提示中看到该 () 的身影出没，假如你的函数需要返回一个 u32 值，但是如果你不幸的以 `x+y;` 的方式作为函数的最后一行代码，就会报错：


```rust
fn add(x:i32,y:i32) -> i32 {
    x + y; // 语句不具备返回值条件
}
```
运行之后:
```bash 
error[E0308]: mismatched types // 类型不匹配
   --> src/main.rs:13:25
    |
113 | fn add(i: i32, j:i32) ->i32{
    |    ---                  ^^^ expected `i32`, found `()`
    |    |
    |    implicitly returns `()` as its body has no tail or `return` expression
114 |     i+j;
    |        - help: remove this semicolon

For more information about this error, try `rustc --explain E0308`.
```
只有表达式能返回值，而 `;` 结尾的是语句，在 Rust 中，一定要严格区分表达式和语句的区别，这个在其它语言中往往是被忽视的点。

最后提下最特殊的函数，永不返回的`发散函数 !`

当用 `!` 作函数返回类型的时候，表示该函数永不返回( diverge function )，特别的，这种语法往往用做会导致程序崩溃的函数：
```rust 
fn its_end() -> ! {
  panic!("这样就崩溃了！");
}
```
