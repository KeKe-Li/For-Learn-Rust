### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 复合类型

在复合类型下需要学习的就蛮多了，我们先根据一个示例来学习!

```rust
#![allow(unused_variables)]
type File = String;

fn open(f: &mut File) -> bool {
    println!("file open, {}",f);
    true
}
fn close(f: &mut File) -> bool {
    println!("file close, {}",f);
    true
}

#[allow(dead_code)]
fn read(f: &mut File, save_to: &mut Vec<u8>) -> ! {
    unimplemented!()
}

fn main() {
    let mut f1 = File::from("f.txt");
    open(&mut f1);
    //read(&mut f1, &mut vec![]);
    close(&mut f1);
}
```

我们运行:
```bash
> rustc main.rs
>./main 
file open, f.txt
file close, f.txt
```

这里就是一个复合类型的应用，有的变量在声明之后并未使用，因此在这个阶段我们需要排除一些编译器报错（Rust 在编译的时候会扫描代码，变量声明后未使用会以 `warning` 警告的形式进行提示），引入 `#![allow(unused_variables)]` 属性标记，该标记会告诉编译器忽略未使用的变量，不要抛出 `warning` 警告。


read 函数也非常有趣，它返回一个 `!` 类型，这个表明该函数是一个发散函数，不会返回任何值，包括 ()。`unimplemented!()` 告诉编译器该函数尚未实现，`unimplemented!()` 标记通常意味着我们期望快速完成主要代码，回头再通过搜索这些标记来完成次要代码，类似的标记还有 `todo!()`，当代码执行到这种未实现的地方时，程序会直接报错。


从代码设计角度来看，关于文件操作的类型和函数应该组织在一起，散落得到处都是，是难以管理和使用的。而且通过 `open(&mut f1)` 进行调用，也远没有使用 `f1.open()` 来调用好，这就体现出了只使用基本类型的局限性：无法从更高的抽象层次去简化代码。


#### 切片(slice)

切片允许你引用集合中部分连续的元素序列，而不是引用整个集合。

对于字符串而言，切片就是对 String 类型中某一部分的引用.

```rust
let s = String::from("hello great");

let hello = &s[0..5];
let great = &s[6..11];
```
hello 没有引用整个 `String s`，而是引用了 s 的一部分内容，通过 `[0..5]` 的方式来指定。

这就是创建切片的语法，使用方括号包括的一个序列：`[开始索引..终止索引]`，其中开始索引是切片中第一个元素的索引位置，而终止索引是最后一个元素后面的索引位置，也就是这是一个 右半开区间。在切片数据结构内部会保存开始的位置和切片的长度，其中长度是通过 `终止索引 - 开始索引` 的方式计算得来的。

对于 `let great = &s[6..11];` 来说，great 是一个切片，该切片的指针指向 s 的第 7 个字节(索引从 0 开始, 6 是第 7 个字节)，且该切片的长度是 5 个字节。


在使用 Rust 的 `.. range` 序列语法时，如果我们想从索引 0 开始，可以用如下的方式，这两个是等效的：
```rust

let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

如果我们的切片想要包含 String 的最后一个字节，则可以这样使用：
```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[4..len];
let slice = &s[4..];
```

如果我们想要截取完整的String 切片：
```rust

let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

注意事项: 在对字符串使用切片语法时需要格外小心，切片的索引必须落在字符之间的边界位置，也就是 UTF-8 字符的边界，例如中文在 UTF-8 中占用三个字节，下面的代码就会panic：
```rust
 let s = "中国";
 let a = &s[0..2];
 println!("{}",a);
```

因为我们只取 s 字符串的前两个字节，但是本例中每个汉字占用三个字节，因此没有落在边界处，也就是连 `中` 字都取不完整，此时程序会直接崩溃退出，如果改成 `&s[0..3]`，则可以正常通过编译。 因此，当你需要对字符串做切片索引操作时，需要格外小心这一点, 关于该如何操作 UTF-8 字符串，


#### 字符串字面量是切片

我们聊到了字符串字面量,但是没有提到它的类型：
```rust
let s = "Hello, Rust!";
```
实际上，s 的类型是 &str，因此你也可以这样声明：
```rust
let s: &str = "Hello, Rust!";
```
该切片指向了程序可执行文件中的某个点，这也是为什么字符串字面量是不可变的，因为 `&str` 是一个不可变引用。

#### 什么是字符串?

字符串是由字符组成的连续集合，Rust 中的字符是 Unicode 类型，因此每个字符占据 4 个字节内存空间，但是在字符串中不一样，字符串是 UTF-8 编码，也就是字符串中的字符所占的字节数是变化的`(1 - 4)`，这样有助于大幅降低字符串所占用的内存空间。

Rust 在语言级别，只有一种字符串类型： `str`，它通常是以引用类型出现 `&str`，也就是上文提到的字符串切片。虽然语言级别只有上述的 str 类型，但是在标准库里，还有多种不同用途的字符串类型，其中使用最广的即是 String 类型。

str 类型是硬编码进可执行文件，也无法被修改，但是 String 则是一个可增长、可改变且具有所有权的 UTF-8 编码字符串，当 Rust 用户提到字符串时，往往指的就是 String 类型和 &str 字符串切片类型，这两个类型都是 UTF-8 编码。

除了 String 类型的字符串，Rust 的标准库还提供了其他类型的字符串，例如 `OsString`， `OsStr`， `CsString` 和 `CsStr` 等，注意到这些名字都以 String 或者 Str 结尾了吗？它们分别对应的是具有所有权和被借用的变量。

#### String 与 &str 的转换

之前我们看到到很多都是如何从 &str 类型生成 String 类型的操作。

```rust
* String::from("hello,rust")
* "hello,rust".to_string()
```

那么如何将 String 类型转为 &str 类型呢？

```rust
fn main() {
    let s = String::from("hello,rust!");
    say_hello(&s);
    say_hello(&s[..]);
    say_hello(s.as_str());
}

fn say_hello(s: &str) {
    println!("{}",s);
}
```

实际上这种灵活用法是因为 `deref` 隐式强制转换.

#### 操作字符串

由于 String 是可变字符串， 那么Rust 字符串就可以使用修改，添加，删除等常用方法：

* 追加 (Push)

在字符串尾部可以使用 `push()` 方法追加字符 `char`，也可以使用 `push_str()` 方法追加字符串字面量。

这两个方法都是在原有的字符串上追加，并不会返回新的字符串。由于字符串追加操作要修改原来的字符串，则该字符串必须是可变的，即字符串变量必须由 mut 关键字修饰。

```rust
fn main() {
    let mut s = String::from("Hello ");
    s.push('r');
    println!("追加字符 push() -> {}", s);

    s.push_str("ust!");
    println!("追加字符串 push_str() -> {}", s);
}
```

运行结果：
```bash
追加字符 push() -> Hello r
追加字符串 push_str() -> Hello rust!
```

* 插入 (Insert)

可以使用 `insert()` 方法插入单个字符 `char`，也可以使用 `insert_str()` 方法插入字符串字面量，与 push() 方法不同，这俩方法需要传入两个参数，第一个参数是字符（串）插入位置的索引，第二个参数是要插入的字符（串），索引从 0 开始计数，如果越界则会发生错误。由于字符串插入操作要修改原来的字符串，则该字符串必须是可变的，即字符串变量必须由 mut 关键字修饰。

示例如下：
```rust
fn main() {
    let mut s = String::from("Hello rust!");
    s.insert(5, ',');
    println!("插入字符 insert() -> {}", s);
    s.insert_str(6, " I like");
    println!("插入字符串 insert_str() -> {}", s);
}
```

运行结果：
```bash
插入字符 insert() -> Hello, rust!
插入字符串 insert_str() -> Hello, I like rust!
```

* 替换 (Replace)

如果想要把字符串中的某个字符串替换成其它的字符串，那可以使用 replace() 方法。与替换有关的方法有三个。

1、replace

该方法可适用于 `String` 和 `&str` 类型。`replace()` 方法接收两个参数，第一个参数是要被替换的字符串，第二个参数是新的字符串。该方法会替换所有匹配到的字符串。该方法是返回一个新的字符串，而不是操作原来的字符串。

示例如下：
```rust
fn main() {
    let string_replace = String::from("I like rust. Learning rust is my favorite!");
    let new_string_replace = string_replace.replace("rust", "RUST");
    dbg!(new_string_replace);
}
```

运行结果：
```bash
new_string_replace = "I like RUST. Learning RUST is my favorite!"
```

2、replacen

该方法可适用于 `String` 和 `&str` 类型。`replacen()` 方法接收三个参数，前两个参数与 `replace()` 方法一样，第三个参数则表示替换的个数。该方法是返回一个新的字符串，而不是操作原来的字符串。

示例如下：
```rust
fn main() {
    let string_replace = "I like rust. Learning rust is great!";
    let new_string_replacen = string_replace.replacen("rust", "RUST", 1);
    dbg!(new_string_replacen);
}
```

运行结果：
```bash
new_string_replacen = "I like RUST. Learning rust is great!"
```

3、replace_range

该方法仅适用于 String 类型。`replace_range` 接收两个参数，第一个参数是要替换字符串的范围（Range），第二个参数是新的字符串。该方法是直接操作原来的字符串，不会返回新的字符串。该方法需要使用 mut 关键字修饰。

示例如下：
```rust
fn main() {
    let mut string_replace_range = String::from("I like rust!");
    string_replace_range.replace_range(7..8, "R");
    dbg!(string_replace_range);
}
```
运行结果：
```bash
string_replace_range = "I like Rust!"
```

#### 删除 (Delete)

与字符串删除相关的方法有 4 个，他们分别是 `pop()`，`remove()`，`truncate()`，`clear()`。这四个方法仅适用于 String 类型。

1、 pop —— 删除并返回字符串的最后一个字符

该方法是直接操作原来的字符串。但是存在返回值，其返回值是一个 Option 类型，如果字符串为空，则返回 None。 

示例如下：
```rust
fn main() {
    let mut string_pop = String::from("rust pop 中文!");
    let p1 = string_pop.pop();
    let p2 = string_pop.pop();
    dbg!(p1);
    dbg!(p2);
    dbg!(string_pop);
}
```

运行结果：
```bash
p1 = Some(
   '!',
)
p2 = Some(
   '文',
)
string_pop = "rust pop 中"
```

2、 remove —— 删除并返回字符串中指定位置的字符

该方法是直接操作原来的字符串。但是存在返回值，其返回值是删除位置的字符串，只接收一个参数，表示该字符起始索引位置。remove() 方法是按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。

示例如下：
```rust
fn main() {
    let mut string_remove = String::from("测试remove方法");
    println!(
        "string_remove 占 {} 个字节",
        std::mem::size_of_val(string_remove.as_str())
    );
    // 删除第一个汉字
    string_remove.remove(0);
    // 下面代码会发生错误
    // string_remove.remove(1);
    // 直接删除第二个汉字
    // string_remove.remove(3);
    dbg!(string_remove);
}

```
运行结果：
```bash
string_remove 占 18 个字节
string_remove = "试remove方法"
```

3、truncate —— 删除字符串中从指定位置开始到结尾的全部字符

该方法是直接操作原来的字符串。无返回值。该方法 truncate() 方法是按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。

示例如下：
```rust
fn main() {
    let mut string_truncate = String::from("测试truncate");
    string_truncate.truncate(3);
    dbg!(string_truncate);
}
```
运行结果：

```bash
string_truncate = "测"
```

4、clear —— 清空字符串

该方法是直接操作原来的字符串。调用后，删除字符串中的所有字符，相当于 `truncate()` 方法参数为 0 的时候。

示例如下：
```rust
fn main() {
    let mut string_clear = String::from("string clear");
    string_clear.clear();
    dbg!(string_clear);
}
```

运行结果：
```rust
string_clear = ""
```

#### 连接 (Concatenate)

1、使用 `+` 或者 `+=` 连接字符串

使用 `+` 或者 `+=` 连接字符串，要求右边的参数必须为字符串的切片引用（Slice）类型。其实当调用 `+` 的操作符时，相当于调用了 `std::string` 标准库中的 add() 方法，这里 add() 方法的第二个参数是一个引用的类型。

因此我们在使用 `+`， 必须传递切片引用类型。不能直接传递 String 类型。`+` 和 `+=` 都是返回一个新的字符串。所以变量声明可以不需要 mut 关键字修饰。

示例如下：
```rust
fn main() {
    let string_append = String::from("hello ");
    let string_rust = String::from("rust");
    // &string_rust会自动解引用为&str
    let result = string_append + &string_rust;
    let mut result = result + "!";
    result += "!!!";

    println!("连接字符串 + -> {}", result);
}
```
运行结果：
```rust
连接字符串 + -> hello rust!!!!
```
add() 方法的定义：

```rust
fn add(self, s: &str) -> String
```
我们简单分析下这个方法:
```rust
fn main() {
    let s1 = String::from("hello,");
    let s2 = String::from("rust!");
    // 在下句中，s1的所有权被转移走了，因此后面不能再使用s1
    let s3 = s1 + &s2;
    assert_eq!(s3,"hello,rust!");
    // 下面的语句如果去掉注释，就会报错
    // println!("{}",s1);
}
```

`self` 是 String 类型的字符串 `s1`，该函数说明，只能将 `&str`类型的字符串切片添加到 String 类型的 s1 上，然后返回一个新的 String 类型，所以 `let s3 = s1 + &s2`; 

这样就很好解释了，将 String 类型的 s1 与 &str 类型的 s2 进行相加，最终得到 String 类型的 s3。

由此可推，以下代码也是合法的：
```rust
let s1 = String::from("keke");
let s2 = String::from("is");
let s3 = String::from("best");

// String = String + &str + &str + &str + &str
let s = s1 + "-" + &s2 + "-" + &s3;
```

`String + &str`返回一个 `String`，然后再继续跟一个` &str` 进行 `+` 操作，返回一个 `String` 类型，不断循环，最终生成一个 `s`，也是 `String` 类型。

s1 这个变量通过调用 `add()` 方法后，所有权被转移到 add() 方法里面， add() 方法调用后就被释放了，同时 s1 也被释放了。再使用 s1 就会发生错误。这里涉及到所有权转移（Move）的相关知识。


2、使用 `format!` 连接字符串

`format!` 这种方式适用于 `String` 和 `&str` 。`format!` 的用法与 `print!` 的用法类似。

示例如下：
```rust
fn main() {
    let s1 = "hello";
    let s2 = String::from("rust");
    let s = format!("{} {}!", s1, s2);
    println!("{}", s);
}
```

运行结果：

```bash
hello rust!
```

#### 字符串转义

我们可以通过转义的方式 `\` 输出 `ASCII` 和 `Unicode` 字符。

```rust
fn main() {
    // 通过 \ + 字符的十六进制表示，转义输出一个字符
    let byte_escape = "I'm writing \x52\x75\x73\x74!";
    println!("What are you doing\x3F (\\x3F means ?) {}", byte_escape);

    // \u 可以输出一个 unicode 字符
    let unicode_codepoint = "\u{211D}";
    let character_name = "\"DOUBLE-STRUCK CAPITAL R\"";

    println!(
        "Unicode character {} (U+211D) is called {}",
        unicode_codepoint, character_name
    );

    // 换行了也会保持之前的字符串格式
    let long_string = "String literals
                        can span multiple lines.
                        The linebreak and indentation here ->\
                        <- can be escaped too!";
    println!("{}", long_string);
}
```

#### 字符串深度剖析

这里我们可以思考下为什么 String 可变，而字符串字面值 str 却不可以？

就字符串字面值来说，我们在编译时就知道其内容，最终字面值文本被直接硬编码进可执行文件中，这使得字符串字面值快速且高效，这主要得益于字符串字面值的不可变性。然而，我们不能为了获得这种性能，而把每一个在编译时大小未知的文本都放进内存中（你也做不到！），因为有的字符串是在程序运行得过程中动态生成的。

对于 String 类型，为了支持一个可变、可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存放内容，这些都是在程序运行时完成的：

* 首先向操作系统请求内存来存放 String 对象
* 在使用完成后，将内存释放，归还给操作系统

* 其中第一部分由 `String::from` 完成，它创建了一个全新的 `String`。

在有垃圾回收 GC 的语言中，GC 来负责标记并清除这些不再使用的内存对象，这个过程都是自动完成，无需开发者关心，非常简单好用；但是在无 GC 的语言中，需要开发者手动去释放这些内存对象，就像创建对象需要通过编写代码来完成一样，未能正确释放对象造成的后果简直不可估量。

对于 Rust 而言，安全和性能是写到骨子里的核心特性，如果使用 GC，那么会牺牲性能；如果使用手动管理内存，那么会牺牲安全，这该怎么办？

为此，Rust 的开发者想出了一个无比惊艳的办法：变量在离开作用域后，就自动释放其占用的内存：

```rust

{
    let s = String::from("hello"); // 从此处起，s 是有效的

    // 使用 s
}                                  // 此作用域已结束，
                                   // s 不再有效，内存被释放
```

与其它系统编程语言的 free 函数相同，Rust 也提供了一个释放内存的函数： drop，但是不同的是，其它语言要手动调用 free 来释放每一个变量占用的内存，而 Rust 则在变量离开作用域时，自动调用 drop 函数: 上面代码中，Rust 在结尾的 `}` 处自动调用 drop。