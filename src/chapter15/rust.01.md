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

