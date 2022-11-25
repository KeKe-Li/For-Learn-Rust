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