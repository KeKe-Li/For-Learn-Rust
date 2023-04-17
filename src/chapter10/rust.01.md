### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。


#### 认识生命周期

生命周期，简而言之就是引用的有效作用域。在大多数时候，我们无需手动的声明生命周期，因为编译器可以自动进行推导，用类型来类比下：

* 就像编译器大部分时候可以自动推导类型 `<->` 一样，编译器大多数时候也可以自动推导生命周期.
* 在多种类型存在时，编译器往往要求我们手动标明类型 `<->` 当多个生命周期存在，且编译器无法推导出某个引用的生命周期时，就需要我们手动标明生命周期.

#### 悬垂指针和生命周期

生命周期的主要作用是避免悬垂引用，它会导致程序引用了本不该引用的数据：

```rust
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```
这段代码有几点值得注意:

* `let r;` 的声明方式貌似存在使用 null 的风险，实际上，当我们不初始化它就使用时，编译器会给予报错.
* r 引用了内部花括号中的 x 变量，但是 x 会在内部花括号`}` 处被释放，因此回到外部花括号后，r 会引用一个无效的 x.

此处 r 就是一个悬垂指针，它引用了提前被释放的变量 x，可以预料到，这段代码会报错：
```bash
error[E0597]: `x` does not live long enough // `x` 活得不够久
  --> src/main.rs:7:17
   |
7  |             r = &x;
   |                 ^^ borrowed value does not live long enough // 被借用的 `x` 活得不够久
8  |         }
   |         - `x` dropped here while still borrowed // `x` 在这里被丢弃，但是它依然还在被借用
9  |
10 |         println!("r: {}", r);
   |                           - borrow later used here // 对 `x` 的借用在此处被使用
```
在这里 r 拥有更大的作用域，或者说活得更久。如果 Rust 不阻止该悬垂引用的发生，那么当 x 被释放后，r 所引用的值就不再是合法的，会导致我们程序发生异常行为，且该异常行为有时候会很难被发现。

#### 借用检查

为了保证 Rust 的所有权和借用的正确性，Rust 使用了一个借用检查器(Borrow checker)，来检查我们程序的借用正确性：
```rust
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```
这段代码和之前的一模一样，唯一的区别在于增加了对变量生命周期的注释。这里，r 变量被赋予了生命周期 'a，x 被赋予了生命周期 'b，从图示上可以明显看出生命周期 'b 比 'a 小很多。

在编译期，Rust 会比较两个变量的生命周期，结果发现 r 明明拥有生命周期 'a，但是却引用了一个小得多的生命周期 'b，在这种情况下，编译器会认为我们的程序存在风险，因此拒绝运行。

如果想要编译通过，也很简单，只要 'b 比 'a 大就好。总之，x 变量只要比 r 活得久，那么 r 就能随意引用 x 且不会存在危险：

```rust
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}   
```

根据之前的结论，我们重新实现了代码，现在 x 的生命周期 'b 大于 r 的生命周期 'a，因此 r 对 x 的引用是安全的。

接下来我们了解下生命周期.

#### 函数中的生命周期

先来看个例子 - 返回两个字符串切片中较长的那个，该函数的参数是两个字符串切片，返回值也是字符串切片：
```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

这段 longest 实现，非常标准优美，就连多余的 return 和分号都没有，可是现实总是给我们重重一击：
```bash
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter // 参数需要一个生命周期
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is
  borrowed from `x` or `y`
  = 帮助： 该函数的返回值是一个引用类型，但是函数签名无法说明，该引用是借用自 `x` 还是 `y`
help: consider introducing a named lifetime parameter // 考虑引入一个生命周期
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ^^^^    ^^^^^^^     ^^^^^^^     ^^^
```

这真是一个复杂的提示，那感觉就好像是生命周期去非诚勿扰相亲，结果在初印象环节就 23 盏灯全灭。如果你仔细阅读，就会发现，其实主要是编译器无法知道该函数的返回值到底引用 x 还是 y ，因为编译器需要知道这些，来确保函数调用后的引用生命周期分析。

不过，就这个函数而言，我们也不知道返回值到底引用哪个，因为一个分支返回 x，另一个分支返回 y...这可咋办？先来分析下。

我们在定义该函数时，首先无法知道传递给函数的具体值，因此到底是 if 还是 else 被执行，无从得知。其次，传入引用的具体生命周期也无法知道，因此也不能像之前的例子那样通过分析生命周期来确定引用是否有效。

同时，编译器的借用检查也无法推导出返回值的生命周期，因为它不知道 x 和 y 的生命周期跟返回值的生命周期之间的关系是怎样的。

因此，在存在多个引用时，编译器有时会无法自动推导生命周期，此时就需要我们手动去标注，通过为参数标注合适的生命周期来帮助编译器进行借用检查的分析。


#### 生命周期标注语法

标记的生命周期只是为了让编译器知道这个事情的最长运行时间，让编译器不要难为我们。

例如一个变量，只能活一个花括号，那么就算你给它标注一个活全局的生命周期，它还是会在前面的花括号结束处被释放掉，并不会真的全局存活。

生命周期的语法也颇为与众不同，以 ' 开头，名称往往是一个单独的小写字母，大多数人都用 'a 来作为生命周期的名称。 如果是引用类型的参数，那么生命周期会位于引用符号 & 之后，并用一个空格来将生命周期和引用参数分隔开:
```markdown
&i32        // 一个引用
&'a i32     // 具有显式生命周期的引用
&'a mut i32 // 具有显式生命周期的可变引用
```
一个生命周期标注，它自身并不具有什么意义，因为生命周期的作用就是告诉编译器多个引用之间的关系。
例如，有一个函数，它的第一个参数 first 是一个指向 i32 类型的引用，具有生命周期 `'a`，该函数还有另一个参数 second，它也是指向 i32 类型的引用，并且同样具有生命周期 `'a`。
此处生命周期标注仅仅说明，这两个参数 first 和 second 至少活得和`'a` 一样久，至于到底活多久或者哪个活得更久，抱歉我们都无法得知：

```rust
fn useless<'a>(first: &'a i32, second: &'a i32) {}
```

* 函数签名中的生命周期标注

之前的 longest 函数，从两个字符串切片中返回较长的那个：
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
需要注意的点如下：

* 和泛型一样，使用生命周期参数，需要先声明 `<'a>`.
* x、y 和返回值至少活得和 `'a` 一样久(因为返回值要么是 x，要么是 y)

该函数签名表明对于某些生命周期 `'a`，函数的两个参数都至少跟 `'a` 活得一样久，同时函数的返回引用也至少跟 `'a` 活得一样久。实际上，这意味着返回值的生命周期与参数生命周期中的较小值一致：虽然两个参数的生命周期都是标注了 `'a`，但是实际上这两个参数的真实生命周期可能是不一样的(生命周期 `'a` 不代表生命周期等于 `'a`，而是大于等于 `'a`)。

这里我们就知道了，在通过函数签名指定生命周期参数时，我们并没有改变传入引用或者返回引用的真实生命周期，而是告诉编译器当不满足此约束条件时，就拒绝编译通过。


因此 longest 函数并不知道 x 和 y 具体会活多久，只要知道它们的作用域至少能持续 'a 这么长就行。

当把具体的引用传给 longest 时，那生命周期 'a 的大小就是 x 和 y 的作用域的重合部分，换句话说，'a 的大小将等于 x 和 y 中较小的那个。由于返回值的生命周期也被标记为 'a，因此返回值的生命周期也是 x 和 y 中作用域较小的那个。

示例：
```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

这里string1 的作用域直到 main 函数的结束，而 string2 的作用域到内部花括号的结束 }，那么根据之前的理论，'a 是两者中作用域较小的那个，也就是 'a 的生命周期等于 string2 的生命周期，同理，由于函数返回的生命周期也是 'a，可以得出函数返回的生命周期也等于 string2 的生命周期。

现在来验证下上面的结论：result 的生命周期等于参数中生命周期最小的，因此要等于 string2 的生命周期，也就是说，result 要活得和 string2 一样久，观察下代码的实现，可以发现这个结论是正确的！


因此，在这种情况下，通过生命周期标注，编译器得出了和我们肉眼观察一样的结论。

再看一个例子，该例子证明了 result 的生命周期必须等于两个参数中生命周期较小的那个:
```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```
运行就有错误了：
```rust
rror[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here
```

在上述代码中，result 必须要活到 println!处，因为 result 的生命周期是 'a，因此 'a 必须持续到 println!。

在 longest 函数中，string2 的生命周期也是 `'a`，由此说明 string2 也必须活到 `println!` 处，可是 string2 在代码中实际上只能活到内部语句块的花括号处 `}`，小于它应该具备的生命周期 `'a`，因此编译出错。

作为人类，我们可以很清晰的看出 result 实际上引用了 string1，因为 string1 的长度明显要比 string2长，既然如此，编译器不该如此矫情才对，它应该能认识到 result 没有引用 string2，让我们这段代码通过。而且 Rust 编译器在调教上是非常保守的：当可能出错也可能不出错时，它会选择前者，抛出编译错误。

总之，显式的使用生命周期，可以让编译器正确的认识到多个引用之间的关系，最终帮我们提前规避可能存在的代码风险。

##### 深入思考生命周期标注

使用生命周期的方式往往取决于函数的功能，例如之前的 longest 函数，如果它永远只返回第一个参数 x，生命周期的标注该如何修改?

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```
在此例中，y 完全没有被使用，因此 y 的生命周期与 x 和返回值的生命周期没有任何关系，意味着我们也不必再为 y 标注生命周期，只需要标注 x 参数和返回值即可。

函数的返回值如果是一个引用类型，那么它的生命周期只会来源于：

* 函数参数的生命周期.
* 函数体中某个新建引用的生命周期.

若是后者情况，就是典型的悬垂引用场景：
```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```
上面的函数的返回值就和参数 x，y 没有任何关系，而是引用了函数体内创建的字符串，那么很显然，该函数会报错：
```rust
error[E0515]: cannot return value referencing local variable `result` // 返回值result引用了本地的变量
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ------^^^^^^^^^
   |     |
   |     returns a value referencing data owned by the current function
   |     `result` is borrowed here
```
主要问题就在于，result 在函数结束后就被释放，但是在函数结束后，对 result 的引用依然在继续。在这种情况下，没有办法指定合适的生命周期来让编译通过，因此我们也就在 Rust 中避免了悬垂引用。

那遇到这种情况该怎么办？最好的办法就是返回内部字符串的所有权，然后把字符串的所有权转移给调用者：

```rust
fn longest<'a>(_x: &str, _y: &str) -> String {
    String::from("really long string")
}

fn main() {
   let s = longest("not", "important");
}
```
至此，可以对生命周期进行下总结：生命周期语法用来将函数的多个引用参数和返回值的作用域关联到一起，一旦关联到一起后，Rust 就拥有充分的信息来确保我们的操作是内存安全的。

#### 结构体中的生命周期

不仅仅函数具有生命周期，结构体其实也有这个概念，只不过我们之前对结构体的使用都停留在非引用类型字段上。那么之前为什么不在结构体中使用字符串字面量或者字符串切片，而是统一使用 String 类型？原因很简单，后者在结构体初始化时，只要转移所有权即可，而前者它们是引用，它们不能为所欲为。

既然之前已经理解了生命周期，那么意味着在结构体中使用引用也变得可能：只要为结构体中的每一个引用标注上生命周期即可：
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

`ImportantExcerpt` 结构体中有一个引用类型的字段 part，因此需要为它标注上生命周期。结构体的生命周期标注语法跟泛型参数语法很像，需要对生命周期参数进行声明 <'a>。该生命周期标注说明，结构体 `ImportantExcerpt` 所引用的字符串 str 必须比该结构体活得更久。

从 main 函数实现来看，`ImportantExcerpt` 的生命周期从main开始，到 main 函数末尾结束，而该结构体引用的字符串从第一行开始，也是到 main 函数末尾结束，可以得出结论结构体引用的字符串活得比结构体久，这符合了编译器对生命周期的要求，因此编译通过。
