### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go.
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++.
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查.

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。


#### 生命周期

在 Rust 语言学习中，一个很重要的部分就是阅读一些你可能不经常遇到，但是一旦遇到就难以理解的代码，这些代码往往最令人头疼的就是生命周期，这里我们就来看看一些本以为可以编译，但是却因为生命周期系统不够聪明导致编译失败的代码。

示例1：
```markdown
#[derive(Debug)]
struct Foo;

impl Foo {
    fn mutate_and_share(&mut self) -> &Self {
        &*self
    }
    fn share(&self) {}
}

fn main() {
    let mut foo = Foo;
    let loan = foo.mutate_and_share();
    foo.share();
    println!("{:?}", loan);
}
```

上面的代码中，`foo.mutate_and_share()` 虽然借用了 `&mut self`，但是它最终返回的是一个 `&self`，然后赋值给 loan，因此理论上来说它最终是进行了不可变借用，同时 `foo.share` 也进行了不可变借用，那么根据 Rust 的借用规则：多个不可变借用可以同时存在，因此该代码应该编译通过。

事实上，运行代码后，你将看到一个错误：
```markdown
error[E0502]: cannot borrow `foo` as immutable because it is also borrowed as mutable
  --> src/main.rs:12:5
   |
11 |     let loan = foo.mutate_and_share();
   |                ---------------------- mutable borrow occurs here
12 |     foo.share();
   |     ^^^^^^^^^^^ immutable borrow occurs here
13 |     println!("{:?}", loan);
   |                      ---- mutable borrow later used here
```

编译器的提示在这里其实有些难以理解，因为可变借用仅在 `mutate_and_share` 方法内部有效，出了该方法后，就只有返回的不可变借用，因此，按理来说可变借用不应该在 main 的作用范围内存在。

对于这个反直觉的事情，让我们用生命周期来解释下，可能你就很好理解了：
```markdown
struct Foo;

impl Foo {
    fn mutate_and_share<'a>(&'a mut self) -> &'a Self {
        &'a *self
    }
    fn share<'a>(&'a self) {}
}

fn main() {
    'b: {
        let mut foo: Foo = Foo;
        'c: {
            let loan: &'c Foo = Foo::mutate_and_share::<'c>(&'c mut foo);
            'd: {
                Foo::share::<'d>(&'d foo);
            }
            println!("{:?}", loan);
        }
    }
}
```

以上是模拟了编译器的生命周期标注后的代码，可以注意到 `&mut foo` 和 `loan` 的生命周期都是 `'c`。

还记得生命周期消除规则中的第三条吗？因为该规则，导致了 `mutate_and_share` 方法中，参数 `&mut self` 和返回值 `&self` 的生命周期是相同的，因此，若返回值的生命周期在 main 函数有效，那 `&mut self` 的借用也是在 main 函数有效。

这就解释了可变借用为啥会在 main 函数作用域内有效，最终导致`foo.share()` 无法再进行不可变借用。

总结下：`&mut self` 借用的生命周期和 loan 的生命周期相同，将持续到 println 结束。而在此期间 `foo.share()` 又进行了一次不可变 `&foo` 借用，违背了可变借用与不可变借用不能同时存在的规则，最终导致了编译错误。

上述代码实际上完全是正确的，但是因为生命周期系统的“粗糙实现”，导致了编译错误，目前来说，遇到这种生命周期系统不够聪明导致的编译错误，我们也没有太好的办法，只能修改代码去满足它的需求，并期待以后它会更聪明。

示例2：
```markdown
#![allow(unused)]
fn main() {
    use std::collections::HashMap;
    use std::hash::Hash;
    fn get_default<'m, K, V>(map: &'m mut HashMap<K, V>, key: K) -> &'m mut V
    where
        K: Clone + Eq + Hash,
        V: Default,
    {
        match map.get_mut(&key) {
            Some(value) => value,
            None => {
                map.insert(key.clone(), V::default());
                map.get_mut(&key).unwrap()
            }
        }
    }
}
```
这段代码不能通过编译的原因是编译器未能精确地判断出某个可变借用不再需要，反而谨慎的给该借用安排了一个很大的作用域，结果导致后续的借用失败：
```markdown
error[E0499]: cannot borrow `*map` as mutable more than once at a time
  --> src/main.rs:13:17
   |
5  |       fn get_default<'m, K, V>(map: &'m mut HashMap<K, V>, key: K) -> &'m mut V
   |                      -- lifetime `'m` defined here
...
10 |           match map.get_mut(&key) {
   |           -     ----------------- first mutable borrow occurs here
   |  _________|
   | |
11 | |             Some(value) => value,
12 | |             None => {
13 | |                 map.insert(key.clone(), V::default());
   | |                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ second mutable borrow occurs here
14 | |                 map.get_mut(&key).unwrap()
15 | |             }
16 | |         }
   | |_________- returning this value requires that `*map` is borrowed for `'m`
```

分析代码可知在 `match map.get_mut(&key)` 方法调用完成后，对 map 的可变借用就可以结束了。但从报错看来，编译器不太聪明，它认为该借用会持续到整个 match 语句块的结束(第 16 行处)，这便造成了后续借用的失败。

#### 无界生命周期

不安全代码(unsafe)经常会凭空产生引用或生命周期，这些生命周期被称为是 无界(unbound) 的。

无界生命周期往往是在解引用一个裸指针(裸指针 raw pointer)时产生的，换句话说，它是凭空产生的，因为输入参数根本就没有这个生命周期：
```markdown
fn f<'a, T>(x: *const T) -> &'a T {
    unsafe {
        &*x
    }
}
```

上述代码中，参数 x 是一个裸指针，它并没有任何生命周期，然后通过 unsafe 操作后，它被进行了解引用，变成了一个 Rust 的标准引用类型，该类型必须要有生命周期，也就是 'a。

可以看出 'a 是凭空产生的，因此它是无界生命周期。这种生命周期由于没有受到任何约束，因此它想要多大就多大，这实际上比 'static 要强大。例如 &'static &'a T 是无效类型，但是无界生命周期 &'unbounded &'a T 会被视为 &'a &'a T 从而通过编译检查，因为它可大可小，就像孙猴子的金箍棒一般。

我们在实际应用中，要尽量避免这种无界生命周期。最简单的避免无界生命周期的方式就是在函数声明中运用生命周期消除规则。若一个输出生命周期被消除了，那么必定因为有一个输入生命周期与之对应。


#### 生命周期约束 HRTB

生命周期约束跟特征约束类似，都是通过形如 'a: 'b 的语法，来说明两个生命周期的长短关系。

* 'a: 'b

假设有两个引用 `&'a i32` 和 `&'b i32`，它们的生命周期分别是 `'a` 和 `'b`，若 `'a >= 'b`，则可以定义 `'a:'b`，表示 `'a` 至少要活得跟 `'b` 一样久。

```markdown
struct DoubleRef<'a,'b:'a, T> {
    r: &'a T,
    s: &'b T
}
```
例如上述代码定义一个结构体，它拥有两个引用字段，类型都是泛型 T，每个引用都拥有自己的生命周期，由于我们使用了生命周期约束 `'b: 'a`，因此 `'b` 必须活得比 `'a` 久，也就是结构体中的 s 字段引用的值必须要比 r 字段引用的值活得要久。

* T: 'a

表示类型 T 必须比 'a 活得要久：

```markdown
struct Ref<'a, T: 'a> {
    r: &'a T
}
```
因为结构体字段 r 引用了 T，因此 r 的生命周期 'a 必须要比 T 的生命周期更短(被引用者的生命周期必须要比引用长)。

在 Rust 1.30 版本之前，该写法是必须的，但是从 1.31 版本开始，编译器可以自动推导 `T: 'a` 类型的约束，因此我们只需这样写即可：
```markdown
struct Ref<'a, T> {
    r: &'a T
}
```
来看一个使用了生命周期约束的综合例子：

```markdown
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a: 'b, 'b> ImportantExcerpt<'a> {
    fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
上面的例子中必须添加约束 `'a: 'b` 后，才能成功编译，因为 `self.part` 的生命周期与 self的生命周期一致，将 `&'a` 类型的生命周期强行转换为 `&'b` 类型，会报错，只有在 `'a >= 'b` 的情况下，`'a` 才能转换成 `'b`。