# String 与 str

## 1. 这一章是干什么的

这是 Rust 初学者最容易反复困惑的一章之一。

很多人一开始都会问：

- `String` 和 `&str` 到底有什么区别？
- 为什么有时候函数参数写 `String`，有时候写 `&str`？
- 为什么字符串字面量不是 `String`？
- 为什么 Rust 不能像很多语言那样随手按下标取一个字符？

这一章就是把这些问题一次讲清楚。

## 2. 学这一章前，你需要知道什么

你最好已经理解：

- 所有权和 move
- 引用与借用
- `&T` 和 `&mut T` 的基本含义

你还需要先带着一个非常重要的前提往下看：

`String` 和 `&str` 不是“一个新写法、一个旧写法”，也不是“两个完全平级的字符串类”。

它们代表的是两种不同层次的东西。

## 3. 先给出最短答案

如果先用一句话粗略概括：

- `String`：拥有字符串数据，可增长，可修改
- `&str`：字符串切片，是对一段字符串数据的借用视图，通常只读

这句话现在先记住，后面我们把它拆开。

## 4. 什么是 `&str`

字符串字面量就是 `&str`：

```rust
fn main() {
    let s = "hello";
    println!("{}", s);
}
```

这里的 `s` 不是 `String`，而是 `&str`。

你可以先把 `&str` 理解成：

- 对字符串内容的一段只读视图
- 它本身不负责拥有和管理那块字符串数据

因为它是借用视图，所以名字里会有 `&`。

## 5. 什么是 `String`

`String` 是标准库提供的拥有型字符串：

```rust
fn main() {
    let s = String::from("hello");
    println!("{}", s);
}
```

`String` 的特点是：

- 拥有数据
- 数据通常在堆上
- 可以增长
- 可以修改

例如：

```rust
fn main() {
    let mut s = String::from("hello");
    s.push_str(" world");

    println!("{}", s);
}
```

这也是为什么上一章讲所有权时，`String` 比整数更容易体现 move。

## 6. 为什么会同时存在 `String` 和 `&str`

因为 Rust 想把两件事分开：

1. 谁拥有数据
2. 谁只是借用数据来读

在很多语言里，这两层感觉经常混在一起，所以你平时不太会明确区分。

Rust 则很认真地区分：

- 拥有者
- 借用者

`String` 更偏“拥有者”，`&str` 更偏“借用视图”。

## 7. `String` 和 `&str` 的关系

先看这段代码：

```rust
fn print_text(text: &str) {
    println!("{}", text);
}

fn main() {
    let s1 = "hello";
    let s2 = String::from("world");

    print_text(s1);
    print_text(&s2);
}
```

这段代码能工作。

为什么？

因为：

- `s1` 本来就是 `&str`
- `s2` 是 `String`
- `&s2` 可以被当成字符串切片来使用

这也是为什么很多函数参数更常写成 `&str`，因为它更通用、更灵活。

## 8. 为什么函数参数常写 `&str`

看这两个函数：

```rust
fn print_a(text: String) {
    println!("{}", text);
}

fn print_b(text: &str) {
    println!("{}", text);
}
```

`print_a` 的问题是：

- 它会拿走所有权
- 调用者必须传 `String`
- 不能直接传字符串字面量

而 `print_b` 更灵活：

- 不拿走所有权
- 字面量能传
- `String` 借用后也能传

所以在“只是读取文本”的场景里，参数经常优先写成 `&str`。

## 9. `String` 什么时候更合适

虽然 `&str` 很常见，但 `String` 也不是“应该避免用”。

`String` 更适合：

- 你需要拥有一份字符串数据
- 你要修改它
- 你要把它存起来长期使用
- 你要从函数里返回拥有所有权的字符串

例如：

```rust
fn make_name() -> String {
    String::from("Rust")
}
```

这里返回 `String` 就很自然，因为函数把一份拥有权明确交出来了。

## 10. 字符串切片

`&str` 里的 `str` 可以是整个字符串，也可以是其中一段切片。

```rust
fn main() {
    let s = String::from("hello world");
    let hello = &s[0..5];
    let world = &s[6..11];

    println!("{} {}", hello, world);
}
```

这里：

- `hello` 是 `&str`
- `world` 也是 `&str`

它们只是对原字符串某一段内容的借用视图。

## 11. 为什么不能像数组那样直接索引字符串

很多初学者会自然地写：

```rust
fn main() {
    let s = String::from("hello");
    let c = s[0];
}
```

Rust 不允许这样做。

最核心的原因是：Rust 字符串使用 UTF-8 编码，一个“字符”不一定只占 1 个字节。

这就导致“第 0 个字节”不一定等于“第 0 个字符”。

所以 Rust 不愿意给你一个看似简单、实际上很容易误导人的字符串随机索引接口。

你现在先记住结论：

- 数组按下标访问很自然
- 字符串不行
- 因为字符串和字节、字符、Unicode 边界有关

## 12. `String` 和修改操作

`String` 可以追加内容：

```rust
fn main() {
    let mut s = String::from("Rust");
    s.push('!');
    s.push_str(" language");

    println!("{}", s);
}
```

常见两个方法：

- `push`：追加单个 `char`
- `push_str`：追加字符串切片 `&str`

这里你也能看出一个细节：

`push_str` 接收的是 `&str`，不是 `String`。这再次说明字符串 API 很常围绕“借用视图”来设计。

## 13. `String` 和 `&str` 的常见转换直觉

你现在先只记最常见的两个方向：

### 从 `&str` 变成 `String`

```rust
fn main() {
    let s1 = "hello".to_string();
    let s2 = String::from("world");

    println!("{} {}", s1, s2);
}
```

### 从 `String` 借成 `&str`

```rust
fn main() {
    let s = String::from("hello");
    let view: &str = &s;

    println!("{}", view);
}
```

这里重点不是死背转换 API，而是理解：

- 拥有型数据可以借成视图
- 视图如果想变成拥有型数据，通常要分配/复制

## 14. 和 JavaScript、Python、Java 的类比

### 和 JavaScript 类比

JavaScript 里的字符串通常让你感觉“就是字符串”，不会强迫你区分“拥有型字符串”和“字符串视图”。

Rust 把这件事拆开，是因为它更重视内存和所有权边界。

### 和 Python 类比

Python 字符串是不可变对象，你日常不会被迫区分“这是一份拥有的数据，还是某种只读切片视图”。Rust 则要求你更清楚这层差别。

### 和 Java 类比

Java 的 `String` 初学时也容易让人形成“字符串就是一种对象”的整体印象。Rust 则进一步分开：

- `String`：拥有型、可增长
- `&str`：借用型视图

从工程角度看，这种拆分更细，也更利于性能和边界控制。

## 15. 常见错误与如何调试

### 错误 1：把字符串字面量当成 `String`

```rust
let s: String = "hello";
```

这不行，因为 `"hello"` 本身是 `&str`，不是 `String`。

你需要：

```rust
let s = String::from("hello");
```

或者：

```rust
let s = "hello".to_string();
```

### 错误 2：读取函数参数写成 `String`

如果函数只是打印或读取内容，优先想一想能不能写成 `&str`。

### 错误 3：想按下标取字符串字符

```rust
let c = s[0];
```

这不是数组，不要用数组直觉去理解字符串。

### 错误 4：混淆 `char`、`String`、`&str`

这三者不是一回事：

- `char`：单个字符
- `&str`：字符串切片
- `String`：拥有型字符串

## 16. 小练习

1. 定义一个字符串字面量，并确认它是 `&str`。
2. 用 `String::from` 创建一个 `String`，追加内容后打印。
3. 写一个函数，参数类型是 `&str`，然后分别传入字面量和 `String` 借用。
4. 试着把一个 `&str` 转成 `String`，再把 `String` 借成 `&str`。

## 17. 挑战题

做一个“自我介绍字符串实验”，要求：

1. 用 `String` 保存你的可变介绍文本
2. 用 `push_str` 再补充一句学习目标
3. 写一个函数，参数是 `&str`，负责打印介绍
4. 从 `String` 中切出一段 `&str`，单独打印其中的某一部分

额外要求：

1. 故意写一次把 `"hello"` 直接赋值给 `String` 类型变量的错误代码
2. 故意写一次字符串下标访问错误代码
3. 读一遍编译器反馈，并写下你自己的理解

## 18. 小结

这一章最关键的理解是：

1. `String` 是拥有型字符串
2. `&str` 是字符串切片，是借用视图
3. 只读函数参数经常优先写 `&str`
4. `String` 适合拥有、修改、返回字符串数据
5. Rust 不让你像数组那样直接索引字符串

下一篇建议进入：

- [结构体、枚举与模式匹配进阶](./09-结构体、枚举与模式匹配进阶.md)
