### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 流程控制

Rust 程序是从上而下顺序执行的，在这个过程中，我们可以通过循环、分支等流程控制方式，更好的实现相应的功能。

常用的流程控制包括:

* 使用 if 来做分支控制

if else 表达式根据条件执行不同的代码分支：
```rust
if condition == true {
    // A...
} else {
    // B...
}
```

* 使用 else if 来处理多重条件

将 else if 与 if、else 组合在一起实现更复杂的条件分支判断:
```rust
fn main() {
    let n = 4;

    if n % 4 == 0 {
        println!("number is divisible by 4");
    } else if n % 6 == 0 {
        println!("number is divisible by 6");
    } else if n % 5== 0 {
        println!("number is divisible by 5");
    } else {
        println!("number is not divisible by 4, 6, or 5");
    }
}    
```

程序执行时，会按照自上至下的顺序执行每一个分支判断，一旦成功，则跳出 if 语句块，最终本程序会匹配执行 else if n % 4 == 0 的分支，输出 "number is divisible by 4"。

有一点要注意，就算有多个分支能匹配，也只有第一个匹配的分支会被执行！

循环控制包括:

在 Rust 语言中有三种循环方式：for、while 和 loop。

* for 循环

for 循环是 Rust 的最有效的手段：
```rust
fn main() {
    for i in 1..=6 {
        println!("{}", i);
    }
}
```

以上代码循环输出一个从 1 到 6 的序列，核心就在于 for 和 in 的联动，语义表达如下：

```rust
for 元素 in 集合 {
  // 使用元素干一些你懂我不懂的事情
}
```

注意，使用 for 时我们往往使用集合的引用形式，除非你不想在后面的代码中继续使用该集合（比如我们这里使用了 container 的引用）。

如果不使用引用的话，所有权会被转移（move）到 for 语句块中，后面就无法再使用这个集合了)：
```rust
for item in &container {
  // ...
}
```
如果想在循环中，修改该元素，可以使用 mut 关键字：
```rust
for item in &mut collection {
  // ...
}
```

for循环的总结如下:

| 使用方法                    | 	等价使用方式                                        | 所有权     |
|----------------------------|--------------------------------------------------|-----------|
| for item in collection	  | for item in IntoIterator::into_iter(collection)	 | 转移所有权  |
| for item in &collection	  | for item in collection.iter()	                 | 不可变借用  |                  
| for item in &mut collection | 	for item in collection.iter_mut()            | 	可变借用  |            

如果想在循环中获取元素的索引：
```rust
fn main() {
    let a = [5, 6, 2, 1];
    // `.iter()` 方法把 `a` 数组变成一个迭代器
    for (i, v) in a.iter().enumerate() {
        println!("第{}个元素是{}", i + 1, v);
    }
}
```

如果我们想用 for 循环控制某个过程执行 10 次，但是又不想单独声明一个变量来控制这个流程，这个可以用 _ 来替代 i 用于 for 循环中，在 Rust 中 _ 的含义是忽略该值或者类型的意思，如果不使用 _，那么编译器会给你一个 变量未使用的 的警告。
```rust
for _ in 0..10 {
  // ...
}
```
通常在循环数组数据中，我们推荐使用:
```rust
let collection = [1, 2, 3, 4, 5, 6];
for item in collection {
}
```

* continue

使用 continue 可以跳过当前当次的循环，开始下次的循环：
```rust

 for i in 1..5 {
     if i == 2 {
         continue;
     }
     println!("{}", i);
 }
```
上面代码对 1 到 4 的序列进行迭代，且跳过值为 2 时的循环，输出如下：
```rust
1
3
4
```
* break

使用 break 可以直接跳出当前整个循环：

```rust

 for i in 1..5 {
     if i == 2 {
         break;
     }
     println!("{}", i);
}
```
上面代码对 1 到 4 的序列进行迭代，在遇到值为 2 时的跳出整个循环，后面的循环不再执行，输出如下：
```rust
1
```

* while 循环

如果你需要一个条件来循环，当该条件为 true 时，继续循环，条件为 false，跳出循环，那么 while 就非常适用：
```rust
fn main() {
    let mut n = 0;

    while n <= 6  {
        println!("{}!", n);

        n = n + 1;
    }

    println!("完成任务了！");
}
```

该 while 循环，只有当 n 小于等于 6时，才执行，否则就立刻跳出循环，因此在上述代码中，它会先从 0 开始，满足条件，进行循环，然后是 1，满足条件，进行循环，最终到 7 的时候，大于 6，不满足条件，跳出 while 循环，执行 我出来了 的打印，然后程序结束：
```rust
0!
1!
2!
3!
4!
5!
6!
完成任务了！
```

当然，你也可以用其它方式组合实现，例如 loop（无条件循环） + if + break：
```rust
fn main() {
    let mut n = 0;

    loop {
        if n > 5 {
            break
        }
        println!("{}", n);
        n+=1;
    }

    println!("完成任务了！");
}
```

* loop 循环

loop是适用面最高的，它可以适用于所有循环场景（虽然能用，但是在很多场景下， for 和 while 才是最优选择），因为 loop 就是一个简单的无限循环，可以在内部实现逻辑通过 break 关键字来控制循环何时结束。

当使用 loop 时，必不可少的伙伴是 break 关键字，它能让循环在满足某个条件时跳出:
```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

以上代码当 counter 递增到 10 时，就会通过 break 返回一个 `counter * 2` 的值，最后赋给 result 并打印出来。

这里有几点值得注意：

* break 可以单独使用，也可以带一个返回值，有些类似 return.
* loop 是一个表达式，因此可以返回一个值.