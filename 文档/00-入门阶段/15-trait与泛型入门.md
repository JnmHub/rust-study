# trait 与泛型入门

## 1. 这一章是干什么的

到这里，你已经能写不少具体代码了。

但很快你会遇到两个更高一级的问题：

- 如果几个函数逻辑一样，只是类型不同，难道要重复写很多遍吗？
- 如果几个类型都支持某种共同能力，怎么把这种“能力”抽象出来？

这一章要解决的就是：

- 什么是泛型
- 什么是 trait
- 它们分别解决什么问题
- 为什么 Rust 会经常把“类型”与“能力”拆开表达

如果你把前面学的内容理解成“怎么写代码”，这一章就是开始进入“怎么减少重复、怎么抽象能力”。

## 2. 学这一章前，你需要知道什么

你最好已经理解：

- 函数和返回值
- 结构体、枚举、`impl`
- `Option` / `Result`

另外先带着一个总印象往下看：

Rust 不喜欢“先全都写死，后面再想办法兼容”。它更喜欢一开始就明确：

- 这段逻辑对哪些类型通用
- 这些类型必须具备什么能力

## 3. 为什么需要泛型

先看一个很朴素的问题：

如果你要写“返回两个数里较大的那个”，可能先写：

```rust
fn max_i32(a: i32, b: i32) -> i32 {
    if a > b { a } else { b }
}
```

如果接着你又想支持 `i64`：

```rust
fn max_i64(a: i64, b: i64) -> i64 {
    if a > b { a } else { b }
}
```

逻辑几乎一样，只是类型不同。

这时就很自然会问：

- 能不能把“类型不同但逻辑相同”抽象掉？

这就是泛型要解决的问题。

## 4. 泛型函数的最小例子

```rust
fn identity<T>(value: T) -> T {
    value
}

fn main() {
    let a = identity(10);
    let b = identity("Rust");

    println!("{}", a);
    println!("{}", b);
}
```

这里先抓住最核心一点：

- `T` 代表“某个类型”

你现在可以先把它理解成：

- 这个函数不绑死具体类型
- 它对很多类型都适用

## 5. 泛型在结构体中的写法

不只是函数可以泛型，结构体也可以：

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let p1 = Point { x: 3, y: 7 };
    let p2 = Point { x: 1.5, y: 2.5 };

    println!("{}", p1.x);
    println!("{}", p2.x);
}
```

这里的意思是：

- `Point<T>` 表示这个点的坐标类型是某个统一的类型 `T`

也就是说：

- 可以是整数点
- 也可以是浮点点

但同一个 `Point<T>` 里，`x` 和 `y` 都必须是同一个类型。

如果你想让两个字段类型不同，也可以写：

```rust
struct Pair<T, U> {
    first: T,
    second: U,
}
```

## 6. 泛型 `impl`

前面你已经学过 `impl`，泛型类型当然也可以配合 `impl`：

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

这里先不要被 `impl<T>` 吓到。

它的意思很直接：

- 我在给一个泛型类型 `Point<T>` 写方法

## 7. 为什么单有泛型还不够

再回到前面的“取最大值”问题。

你也许会自然写成：

```rust
fn max_value<T>(a: T, b: T) -> T {
    if a > b { a } else { b }
}
```

但这通常会报错。

为什么？

因为 Rust 会问你：

- 你凭什么假设所有 `T` 都支持 `>` 比较？

这时就需要一种方式告诉编译器：

- 这个泛型类型虽然不固定
- 但它必须具备某种能力

这就是 trait 要解决的问题。

## 8. trait 是什么

你可以先把 trait 理解成：

- 一组能力要求
- 一份行为契约

例如：

- 某个类型是否能打印
- 是否能比较
- 是否能复制
- 是否能转成字符串

这些都可以理解成“能力”。

## 9. 最小 trait 例子

```rust
trait Summary {
    fn summary(&self) -> String;
}
```

这可以先理解成：

- 如果一个类型想实现 `Summary`
- 那它就得提供 `summary(&self) -> String` 这个方法

例如：

```rust
struct Article {
    title: String,
}

impl Summary for Article {
    fn summary(&self) -> String {
        format!("Article: {}", self.title)
    }
}
```

这表示：

- `Article` 这个类型拥有了 `Summary` 这项能力

## 10. trait 约束

现在泛型和 trait 就可以配合起来了：

```rust
trait Summary {
    fn summary(&self) -> String;
}

fn print_summary<T: Summary>(item: &T) {
    println!("{}", item.summary());
}
```

这里最关键的部分是：

```rust
T: Summary
```

它的意思是：

- `T` 可以是很多类型
- 但它必须实现 `Summary`

这就是 trait 约束。

## 11. 为什么这套设计很重要

因为 Rust 想把两件事拆开：

1. 这是什么类型
2. 它具备什么能力

这会比很多语言里“类型和能力混在一起”的感觉更明确。

## 12. 一个稍微真实一点的例子

```rust
trait Named {
    fn name(&self) -> &str;
}

struct Learner {
    name: String,
}

impl Named for Learner {
    fn name(&self) -> &str {
        &self.name
    }
}

fn greet<T: Named>(item: &T) {
    println!("Hello, {}!", item.name());
}

fn main() {
    let learner = Learner {
        name: String::from("Alice"),
    };

    greet(&learner);
}
```

这里的重点不是语法花样，而是思路：

- `greet` 不关心你到底是不是 `Learner`
- 它只关心你是否具备 `Named` 这项能力

## 13. 泛型和 trait 的分工

你可以先这样记：

### 泛型解决的是

- 代码别写死在一个具体类型上

### trait 解决的是

- 这些类型必须具备哪些能力

它们经常一起出现，因为：

- 泛型提供“通用性”
- trait 提供“约束”

## 14. 和 JavaScript、Python、Java 的类比

### 和 JavaScript 类比

JavaScript 很多时候是“对象上有这个方法就用”，偏鸭子类型直觉。Rust 不走这条默认路线，它要你更明确地说：

- 这个泛型类型需要实现什么 trait

### 和 Python 类比

Python 也常有“你能调这个方法就行”的感觉。Rust 更偏编译期约束，会更早告诉你：

- 这个类型不满足要求

### 和 Java 类比

如果你学过 Java 泛型和接口，会更容易理解：

- 泛型像“类型参数”
- trait 有点像“接口能力要求”

但 Rust trait 的使用地位更核心，也更广。

## 15. 常见错误与如何调试

### 错误 1：以为泛型就是“万能类型”

不是。泛型不代表它自动拥有一切能力。

### 错误 2：用了比较、打印、克隆等操作，却没给 trait 约束

如果编译器说某个操作对 `T` 不可用，先检查：

- 这个 `T` 是不是缺少必要 trait 约束

### 错误 3：把 trait 理解成“类”

trait 不是类，也不是字段容器。它更像“能力接口”。

### 错误 4：泛型结构体方法里忘了 `impl<T>`

如果类型本身带泛型，方法实现也通常要把泛型参数带上。

## 16. 小练习

1. 写一个泛型函数 `echo<T>(value: T) -> T`。
2. 写一个泛型结构体 `Boxed<T>`，里面只存一个值。
3. 自己定义一个 trait，例如 `Named` 或 `Describable`。
4. 给一个结构体实现这个 trait，再写一个带 trait 约束的函数使用它。

## 17. 挑战题

做一个“学习对象抽象实验”，要求：

1. 定义一个 trait，表示“可以展示学习摘要”
2. 定义两个不同结构体，例如 `Learner` 和 `Course`
3. 为它们都实现这个 trait
4. 写一个泛型函数，接收任何实现了这个 trait 的类型并打印摘要

额外要求：

1. 至少写一个泛型结构体或泛型函数
2. 至少写一个 trait
3. 至少写一个 `T: SomeTrait` 风格的约束

## 18. 小结

这一章最关键的理解是：

1. 泛型让代码不绑死在具体类型上
2. trait 用来表达“能力”或“约束”
3. 泛型和 trait 经常配合使用
4. Rust 会要求你明确说明泛型类型需要具备什么能力
5. 这套机制是后面进入更强抽象能力的起点

下一篇建议进入：

- [生命周期的第一轮必要直觉](./16-生命周期的第一轮必要直觉.md)
