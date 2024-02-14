### rust 学习

Rust 之所以受欢迎是因为其内存安全性。在以往，内存安全几乎都是通过 GC 的方式实现，但是 GC 会引来性能、内存占用以及 `Stop the world `等问题，在高性能场景和系统编程上是不可接受的，因此 Rust 采用了所有权系统。

所有的程序都必须和计算机内存打交道，如何从内存中申请空间来存放程序的运行内容，如何在不需要的时候释放这些空间，成了重中之重，也是所有编程语言设计的难点之一。

在计算机语言不断演变过程中，出现了三种流派：

* 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，代表语言：Java、Go.
* 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++.
* 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查.

其中 Rust 选择了第三种，最妙的是，这种检查只发生在编译期，因此对于程序运行期，不会有任何性能上的损失。

#### 类型转换

Rust 是类型安全的语言，因此在 Rust 中做类型转换不是一件简单的事。

#### as转换

先来看一段代码：
```markdown
fn main() {
  let a: i32 = 10;
  let b: u16 = 100;

  if a < b {
    println!("Ten is less than one hundred.");
  }
}
```
这里，因为 a 和 b 拥有不同的类型，Rust 不允许两种不同的类型进行比较。

解决办法很简单，只要把 b 转换成 i32 类型即可，Rust 中内置了一些基本类型之间的转换，这里使用 as 操作符来完成： `if a < (b as i32) {...}`。那么为什么不把 a 转换成 u16 类型呢？

因为每个类型能表达的数据范围不同，如果把范围较大的类型转换成较小的类型，会造成错误，因此我们需要把范围较小的类型转换成较大的类型，来避免这些问题的发生。

这里需要注意下：
```markdown
使用类型转换需要小心，因为如果执行以下操作 300_i32 as i8，你将获得 44 这个值，而不是 300，因为 i8 类型能表达的的最大值为 2^7 - 1，使用以下代码可以查看 i8 的最大值：

let a = i8::MAX;
println!("{}",a);
```

下面列出了常用的转换形式：
```markdown
fn main() {
   let a = 3.1 as i8;
   let b = 100_i8 as i32;
   let c = 'a' as u8; // 将字符'a'转换为整数，97

   println!("{},{},{}",a,b,c)
}
```
内存地址转换为指针:
```markdown
let mut values: [i32; 2] = [1, 2];
let p1: *mut i32 = values.as_mut_ptr();
let first_address = p1 as usize; // 将p1内存地址转换为一个整数
let second_address = first_address + 4; // 4 == std::mem::size_of::<i32>()，i32类型占用4个字节，因此将内存地址 + 4
let p2 = second_address as *mut i32; // 访问该地址指向的下一个整数p2
unsafe {
    *p2 += 1;
}
assert_eq!(values[1], 3);
```
强制类型转换的边角知识:

* 转换不具有传递性 就算 `e as U1 as U2` 是合法的，也不能说明 `e as U2` 是合法的（e 不能直接转换成 U2）。

#### TryInto 转换

在一些场景中，使用 as 关键字会有比较大的限制。如果你想要在类型转换上拥有完全的控制而不依赖内置的转换，例如处理转换错误，那么可以使用 TryInto ：
```markdown
use std::convert::TryInto;

fn main() {
   let a: u8 = 10;
   let b: u16 = 1500;

   let b_: u8 = b.try_into().unwrap();

   if a < b_ {
     println!("Ten is less than one hundred.");
   }
}
```

上面代码中引入了 `std::convert::TryInto` 特征，但是却没有使用它，可能大家为此困惑，主要原因在于如果你要使用一个特征的方法，那么你需要引入该特征到当前的作用域中，我们在上面用到了 try_into 方法，因此需要引入对应的特征。

但是 Rust 又提供了一个非常便利的办法，把最常用的标准库中的特征通过`std::prelude`模块提前引入到当前作用域中，其中包括了 `std::convert::TryInto`，你可以尝试删除第一行的代码 `use ...`，看看是否会报错。

try_into 会尝试进行一次转换，并返回一个 Result，此时就可以对其进行相应的错误处理。由于我们的例子只是为了快速测试，因此使用了 unwrap 方法，该方法在发现错误时，会直接调用 panic 导致程序的崩溃退出，在实际项目中，请不要这么使用，具体见panic部分。

最主要的是 try_into 转换会捕获大类型向小类型转换时导致的溢出错误：
```markdown
fn main() {
    let b: i16 = 1500;

    let b_: u8 = match b.try_into() {
        Ok(b1) => b1,
        Err(e) => {
            println!("{:?}", e.to_string());
            0
        }
    };
}
```

运行后输出如下 `"out of range integral type conversion attempted"`，在这里我们程序捕获了错误，编译器告诉我们类型范围超出的转换是不被允许的，因为我们试图把 `1500_i16` 转换为 u8 类型，后者明显不足以承载这么大的值。

