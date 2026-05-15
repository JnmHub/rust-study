# Option 与 Result 入门

## 1. 这一章是干什么的

这一章是入门阶段非常重要的一次收束。

前面你已经学过：

- `enum`
- `match`
- 所有权
- 借用

现在这些东西要第一次真正汇合起来，用来解决两个现实问题：

1. 一个值可能存在，也可能不存在，怎么办？
2. 一个操作可能成功，也可能失败，怎么办？

Rust 对这两个问题的回答分别是：

- `Option<T>`
- `Result<T, E>`

## 2. 学这一章前，你需要知道什么

你最好已经理解：

- `enum` 和 `match`
- 函数返回值
- `String`、`&str`

另外先带着一个总体印象往下看：

Rust 不喜欢用“模糊空值”或“默认把错误往运行时扔”来糊过去，它更倾向于让这些情况直接出现在类型里。

## 3. 为什么需要 `Option`

很多语言里，“没有值”经常用：

- `null`
- `None`
- `undefined`

这很方便，但也很容易让你忘记处理“值不存在”的情况。

Rust 的思路是：

如果一个值可能不存在，那就把这件事明确写进类型。

于是就有了：

```rust
enum Option<T> {
    Some(T),
    None,
}
```

你现在不用背它的完整定义，只要先理解：

- `Some(value)`：有值
- `None`：没值

## 4. `Option` 的最小例子

```rust
fn main() {
    let name: Option<String> = Some(String::from("Rust"));
    let empty_name: Option<String> = None;
}
```

这里的含义很直白：

- `name` 可能有字符串
- `empty_name` 当前没有字符串

你不能假装它们和普通 `String` 一样，因为类型已经明确告诉你：可能缺值。

## 5. 用 `match` 处理 `Option`

```rust
fn main() {
    let name = Some(String::from("Rust"));

    match name {
        Some(value) => println!("name = {}", value),
        None => println!("no name"),
    }
}
```

这就是 Rust 很典型的风格：

- 状态用 `enum` 表达
- 分支用 `match` 显式处理

你必须承认“没有值”这件事是可能发生的。

## 6. `if let` 处理 `Option`

如果你只关心“有值”这一种情况，可以简化：

```rust
fn main() {
    let name = Some(String::from("Rust"));

    if let Some(value) = name {
        println!("name = {}", value);
    }
}
```

这适合：

- 你只关心某一种分支
- 不想写完整 `match`

## 7. 为什么不能直接把 `Option<String>` 当 `String` 用

例如：

```rust
fn main() {
    let name = Some(String::from("Rust"));
    println!("{}", name);
}
```

这不是你真正想要的“打印内部字符串”。

因为 `name` 的类型是 `Option<String>`，不是 `String`。

Rust 在提醒你：

- 这个值可能不存在
- 你得先拆开它，再决定怎么处理

## 8. `unwrap()` 和 `expect()`

Rust 提供了快捷方式：

```rust
fn main() {
    let name = Some(String::from("Rust"));
    let value = name.unwrap();

    println!("{}", value);
}
```

如果是 `Some`，它会取出内部值。

但如果是 `None`，程序会 panic。

所以初学时你要建立一个健康习惯：

- 学习例子里可以用 `unwrap()`
- 但要知道它不是“严肃处理缺值”的最终方案

`expect()` 类似，只是能加自定义报错信息：

```rust
let value = name.expect("name should exist");
```

## 9. 为什么需要 `Result`

`Option` 解决的是“可能没有值”。

但有时候你不只是想知道“失败了”，你还想知道：

- 为什么失败
- 错误信息是什么

这时 Rust 用：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

你现在先只记住：

- `Ok(value)`：成功
- `Err(error)`：失败

## 10. `Result` 的最小例子

看一个很常见的解析例子：

```rust
fn main() {
    let good = "42".parse::<i32>();
    let bad = "abc".parse::<i32>();

    println!("{:?}", good);
    println!("{:?}", bad);
}
```

这里：

- `"42"` 能成功解析，结果类似 `Ok(42)`
- `"abc"` 不能成功解析，结果类似 `Err(...)`

这就是 `Result` 在表达：

这个操作可能成功，也可能失败。

## 11. 用 `match` 处理 `Result`

```rust
fn main() {
    let input = "42";
    let parsed = input.parse::<i32>();

    match parsed {
        Ok(number) => println!("number = {}", number),
        Err(error) => println!("parse failed: {}", error),
    }
}
```

这时你不只是知道“失败了”，还可以拿到失败原因。

## 12. `Result` 和 `expect()`

如果你现在只想快速写例子，可以这么做：

```rust
fn main() {
    let input = "42";
    let number = input.parse::<i32>().expect("input should be a valid integer");

    println!("{}", number);
}
```

这表示：

- 如果成功，就拿到值
- 如果失败，就直接 panic，并打印你写的提示

这在学习和小实验里很常见，但后面真实项目里会逐渐走向更稳妥的错误处理。

## 13. `Option` 和 `Result` 的差别

你可以先这样记：

### `Option<T>`

适合：

- 可能有值，也可能没值
- 但“没值”本身不一定是错误

例如：

- 查找一个不存在的昵称
- 从列表里拿第一个元素，但列表可能为空

### `Result<T, E>`

适合：

- 操作可能失败
- 而且你想知道为什么失败

例如：

- 字符串转数字
- 文件读取
- 网络请求

## 14. 为什么这是 Rust 的核心风格

因为 Rust 不想让你“假设一切都成功”。

它更喜欢：

- 缺值要明确
- 错误要明确
- 调用者必须面对这些分支

这会让你在一开始觉得代码更啰嗦，但它带来的好处是：

- 边界更清楚
- 漏处理更少
- 运行时惊喜更少

## 15. 和 JavaScript、Python、Java 的类比

### 和 JavaScript 类比

JavaScript 里常见的是：

- `undefined`
- `null`
- `throw`

这些机制很灵活，但也容易让“没值”和“失败”在代码里比较散。Rust 则把它们结构化成类型。

### 和 Python 类比

Python 常见是：

- `None`
- `try/except`

Rust 不走完全一样的路线，它希望“可能缺值、可能失败”在函数签名和类型里直接可见。

### 和 Java 类比

Java 里你可能会接触：

- `null`
- 异常
- `Optional`

Rust 的 `Option` 和 `Result` 在语言日常使用里地位更核心，不是“偶尔才用的高级工具”。

## 16. 常见错误与如何调试

### 错误 1：把 `Option<T>` 当成 `T`

你需要先匹配或解包，而不是直接当内部值使用。

### 错误 2：一上来就无脑 `unwrap()`

`unwrap()` 很方便，但它不是认真处理边界情况的终点。

### 错误 3：分不清“没值”和“失败”

如果只是可能不存在，用 `Option` 更自然。

如果需要错误原因，用 `Result` 更自然。

### 错误 4：看见 `Err` 就慌

先别慌。`Err` 不是“程序坏了”，它只是类型在老老实实告诉你：这步操作可能失败。

## 17. 小练习

1. 写一个 `Option<String>`，分别测试 `Some` 和 `None` 的匹配。
2. 写一个把字符串解析成整数的例子，并用 `match` 处理 `Ok` / `Err`。
3. 试试 `unwrap()` 和 `expect()` 的差别。
4. 自己写一句话总结：`Option` 和 `Result` 分别在解决什么问题。

## 18. 挑战题

做一个“小型输入分析器”，要求：

1. 准备三个字符串输入，例如 `"42"`、`"abc"`、`""`
2. 先写一个函数，判断输入是否为空，如果为空返回 `None`，否则返回 `Some(&str)` 或 `Some(String)`
3. 再写一个函数，把非空输入尝试解析成整数，返回 `Result<i32, _>`
4. 在 `main` 中把两层结果都打印清楚

额外要求：

1. 至少用一次 `match`
2. 至少用一次 `if let`
3. 至少用一次 `expect()`，并理解它为什么不适合乱用

## 19. 小结

这一章最关键的理解是：

1. `Option<T>` 用来表示“可能有值，也可能没值”
2. `Result<T, E>` 用来表示“可能成功，也可能失败”
3. Rust 喜欢把边界情况写进类型
4. `match` 和 `if let` 是处理这类类型的基础工具
5. `unwrap()` 很方便，但不能成为长期默认方案

到这里，入门阶段最核心的第一圈已经基本闭合了。下一批最自然的内容会是：

- 集合类型：`Vec<T>` 与 `HashMap<K, V>`
- 模块与文件组织
- 方法与 `impl`
- 测试基础
