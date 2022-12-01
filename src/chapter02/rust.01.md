### rust 学习

rust中的一些关键字
```markdown
as - 强制类型转换，消除特定包含项的 trait 的歧义，或者对 use 和 extern crate 语句中的项重命名
break - 立刻退出循环
const - 定义常量或不变裸指针（constant raw pointer）
continue - 继续进入下一次循环迭代
crate - 链接（link）一个外部 crate 或一个代表宏定义的 crate 的宏变量
dyn - 动态分发 trait 对象
else - 作为 if 和 if let 控制流结构的 fallback
enum - 定义一个枚举
extern - 链接一个外部 crate 、函数或变量
false - 布尔字面值 false
fn - 定义一个函数或 函数指针类型 (function pointer type)
for - 遍历一个迭代器或实现一个 trait 或者指定一个更高级的生命周期
if - 基于条件表达式的结果分支
impl - 实现自有或 trait 功能
in - for 循环语法的一部分
let - 绑定一个变量
loop - 无条件循环
match - 模式匹配
mod - 定义一个模块
move - 使闭包获取其所捕获项的所有权
mut - 表示引用、裸指针或模式绑定的可变性
pub - 表示结构体字段、impl 块或模块的公有可见性
ref - 通过引用绑定
return - 从函数中返回
Self - 实现 trait 的类型的类型别名
self - 表示方法本身或当前模块
static - 表示全局变量或在整个程序执行期间保持其生命周期
struct - 定义一个结构体
super - 表示当前模块的父模块
trait - 定义一个 trait，Rust语言的一个特性(性状)，它描述了它可以提供的每种类型的功能,类似于其他语言中定义的接口的特征
true - 布尔字面值 true
type - 定义一个类型别名或关联类型
unsafe - 表示不安全的代码、函数、trait 或实现
use - 引入外部空间的符号
where - 表示一个约束类型的从句
while - 基于一个表达式的结果判断是否进行循环
保留做将来使用的关键字
```

如下关键字没有任何功能，但是Rust 保留了，为了以备将来的应用。
```markdown
abstract
async
await
become
box
do
final
macro
override
priv
try
typeof
unsized
virtual
yield
```

