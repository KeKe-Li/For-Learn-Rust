
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

