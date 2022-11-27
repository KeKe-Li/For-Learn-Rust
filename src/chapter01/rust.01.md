### rust 学习

Rust 是一门全新的语言，它可能会带给你前所未有的体验，提升你的通用编程水平，甚至于赋予你全新的编程思想。但是难度也是有的，需要不断学习。

#### 变量绑定与解构

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