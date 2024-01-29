
### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go.
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++.
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查.

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 迭代器 Iterator

迭代器允许我们迭代一个连续的集合，例如数组、动态数组 Vec、HashMap 等，在此过程中，只需关心集合中的元素如何处理，而无需关心如何开始、如何结束、按照什么样的索引去访问等问题。

#### For 循环与迭代器

从用途来看，迭代器跟 for 循环颇为相似，都是去遍历一个集合，但是实际上它们存在不小的差别，其中最主要的差别就是：是否通过索引来访问集合。

例如以下的 JS 代码就是一个循环：
```markdown
let arr = [1, 2, 3];
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
```

在上面代码中，我们设置索引的开始点和结束点，然后再通过索引去访问元素 `arr[i]`，这就是典型的循环，来对比下 Rust 中的 for：
```markdown
let arr = [1, 2, 3];
for v in arr {
    println!("{}",v);
}
```
首先，不得不说这两语法还挺像！与 JS 循环不同，Rust中没有使用索引，它把 arr 数组当成一个迭代器，直接去遍历其中的元素，从哪里开始，从哪里结束，都无需操心。
因此严格来说，Rust 中的 for 循环是编译器提供的语法糖，最终还是对迭代器中的元素进行遍历。

简而言之就是数组实现了 IntoIterator 特征，Rust 通过 for 语法糖，自动把实现了该特征的数组类型转换为迭代器（你也可以为自己的集合类型实现此特征），最终让我们可以直接对一个数组进行迭代，类似的还有：
```markdown
for i in 1..10 {
    println!("{}", i);
}
```
直接对数值序列进行迭代，也是很常见的使用方式。

IntoIterator 特征拥有一个 into_iter 方法，因此我们还可以显式的把数组转换成迭代器：
```markdown
let arr = [1, 2, 3];
for v in arr.into_iter() {
    println!("{}", v);
}
```

#### 惰性初始化

在 Rust 中，迭代器是惰性的，意味着如果你不使用它，那么它将不会发生任何事：
```markdown
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("{}", val);
}
```

在 for 循环之前，我们只是简单的创建了一个迭代器 v1_iter，此时不会发生任何迭代行为，只有在 for 循环开始后，迭代器才会开始迭代其中的元素，最后打印出来。

这种惰性初始化的方式确保了创建迭代器不会有任何额外的性能损耗，其中的元素也不会被消耗，只有使用到该迭代器的时候，一切才开始。

#### next 方法

对于 for 如何遍历迭代器，还有一个问题，它如何取出迭代器中的元素？
先来看一个特征：
```markdown
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 省略其余有默认实现的方法
}
```
迭代器之所以成为迭代器，就是因为实现了 Iterator 特征，要实现该特征，最主要的就是实现其中的 next 方法，该方法控制如何从集合中取值，最终返回值的类型是关联类型 Item。

for 循环通过不停调用迭代器上的 next 方法，来获取迭代器中的元素。

既然 for 可以调用 next 方法,我们来看看一个示例:
```markdown
fn main() {
    let arr = [1, 2, 3];
    let mut arr_iter = arr.into_iter();

    assert_eq!(arr_iter.next(), Some(1));
    assert_eq!(arr_iter.next(), Some(2));
    assert_eq!(arr_iter.next(), Some(3));
    assert_eq!(arr_iter.next(), None);
}
```
将 arr 转换成迭代器后，通过调用其上的 next 方法，我们获取了 arr 中的元素，有两点需要注意：

* next 方法返回的是 Option 类型，当有值时返回 Some(i32)，无值时返回 None.

* 遍历是按照迭代器中元素的排列顺序依次进行的，因此我们严格按照数组中元素的顺序取出了 Some(1)，Some(2)，Some(3).

* 手动迭代必须将迭代器声明为 mut 可变，因为调用 next 会改变迭代器其中的状态数据（当前遍历的位置等），而 for 循环去迭代则无需标注 mut，因为它会帮我们自动完成.

总之，next 方法对迭代器的遍历是消耗性的，每次消耗它一个元素，最终迭代器中将没有任何元素，只能返回 None。

示例：模拟实现 for 循环

因为 for 循环是迭代器的语法糖，因此我们完全可以通过迭代器来模拟实现它：
```markdown
let values = vec![1, 2, 3];

{
    let result = match IntoIterator::into_iter(values) {
        mut iter => loop {
            match iter.next() {
                Some(x) => { println!("{}", x); },
                None => break,
            }
        },
    };
    result
}
```
IntoIterator::into_iter 是使用完全限定的方式去调用 into_iter 方法，这种调用方式跟 values.into_iter() 是等价的。

同时我们使用了 loop 循环配合 next 方法来遍历迭代器中的元素，当迭代器返回 None 时，跳出循环。

#### IntoIterator 特征

由于 Vec 动态数组实现了 IntoIterator 特征，因此可以通过 into_iter 将其转换为迭代器，那如果本身就是一个迭代器，该怎么办？

实际上，迭代器自身也实现了 IntoIterator，标准库早就帮我们考虑好了：
```markdown
impl<I: Iterator> IntoIterator for I {
    type Item = I::Item;
    type IntoIter = I;

    #[inline]
    fn into_iter(self) -> I {
        self
    }
}
```
最终你可以写出这样的代码：
```markdown
fn main() {
    let values = vec![1, 2, 3];

    for v in values.into_iter().into_iter().into_iter() {
        println!("{}",v)
    }
}
```
```markdown
into_iter, iter, iter_mut
```

在之前的代码中，我们统一使用了 into_iter 的方式将数组转化为迭代器，除此之外，还有 iter 和 iter_mut，这里应该大概能猜到这三者的区别：

* into_iter 会夺走所有权.
* iter 是借用.
* iter_mut 是可变借用.

其实如果见多了，你会发现这种问题一眼就能看穿，into_ 之类的，都是拿走所有权，_mut 之类的都是可变借用，剩下的就是不可变借用。

使用一段代码来解释下：
```markdown
fn main() {
    let values = vec![1, 2, 3];

    for v in values.into_iter() {
        println!("{}", v)
    }

    // 下面的代码将报错，因为 values 的所有权在上面 `for` 循环中已经被转移走
    // println!("{:?}",values);

    let values = vec![1, 2, 3];
    let _values_iter = values.iter();

    // 不会报错，因为 values_iter 只是借用了 values 中的元素
    println!("{:?}", values);

    let mut values = vec![1, 2, 3];
    // 对 values 中的元素进行可变借用
    let mut values_iter_mut = values.iter_mut();

    // 取出第一个元素，并修改为0
    if let Some(v) = values_iter_mut.next() {
        *v = 0;
    }

    // 输出[0, 2, 3]
    println!("{:?}", values);
}
```

具体解释在代码注释中，就不再赘述，不过有两点需要注意的是：

* `.iter()` 方法实现的迭代器，调用 next 方法返回的类型是 `Some(&T)`
* `.iter_mut()` 方法实现的迭代器，调用 next 方法返回的类型是 `Some(&mut T)`，因此在 `if let Some(v) = values_iter_mut.next()` 中，v 的类型是 `&mut i32`，最终我们可以通过 `*v = 0` 的方式修改其值
Iterator 和 IntoIterator 的区别
这两个其实还蛮容易搞混的，但我们只需要记住，Iterator 就是迭代器特征，只有实现了它才能称为迭代器，才能调用 next。

而 IntoIterator 强调的是某一个类型如果实现了该特征，它可以通过 `into_iter`，iter 等方法变成一个迭代器。

#### 消费者与适配器

消费者是迭代器上的方法，它会消费掉迭代器中的元素，然后返回其类型的值，这些消费者都有一个共同的特点：在它们的定义中，都依赖 next 方法来消费元素，因此这也是为什么迭代器要实现 Iterator 特征，而该特征必须要实现 next 方法的原因。

#### 消费者适配器

只要迭代器上的某个方法 A 在其内部调用了 next 方法，那么 A 就被称为消费性适配器：因为 next 方法会消耗掉迭代器上的元素，所以方法 A 的调用也会消耗掉迭代器上的元素。

其中一个例子是 sum 方法，它会拿走迭代器的所有权，然后通过不断调用 next 方法对里面的元素进行求和：
```markdown
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);

    // v1_iter 是借用了 v1，因此 v1 可以照常使用
    println!("{:?}",v1);

    // 以下代码会报错，因为 `sum` 拿到了迭代器 `v1_iter` 的所有权
    // println!("{:?}",v1_iter);
}
```

如代码注释中所说明的：在使用 sum 方法后，我们将无法再使用 `v1_iter`，因为 sum 拿走了该迭代器的所有权：
```markdown
fn sum<S>(self) -> S
    where
        Self: Sized,
        S: Sum<Self::Item>,
    {
        Sum::sum(self)
    }
```
从 sum 源码中也可以清晰看出，self 类型的方法参数拿走了所有权。

#### 迭代器适配器

既然消费者适配器是消费掉迭代器，然后返回一个值。那么迭代器适配器，顾名思义，会返回一个新的迭代器，这是实现链式方法调用的关键：`v.iter().map().filter()...`。

与消费者适配器不同，迭代器适配器是惰性的，意味着你需要一个消费者适配器来收尾，最终将迭代器转换成一个具体的值：
```markdown
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```
运行后输出:
```markdown
warning: unused `Map` that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: iterators are lazy and do nothing unless consumed // 迭代器 map 是惰性的，这里不产生任何效果
```
如上述中文注释所说，这里的 map 方法是一个迭代者适配器，它是惰性的，不产生任何行为，因此我们还需要一个消费者适配器进行收尾：
```markdown
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```




