# 方法与 impl

## 1. 这一章是干什么的

前面你已经学会了：

- 用 `struct` 表示数据
- 用函数写逻辑

但真实代码里，很多行为并不是随便散在外面的函数，而是会和某种数据类型绑定在一起。

例如：

- 一个用户可以“打招呼”
- 一个计数器可以“加一”
- 一个学习计划可以“追加任务”

这一章要解决的就是：

- Rust 怎么把“数据”和“行为”放在一起
- 什么是方法
- 什么是关联函数
- `impl` 到底在干什么
- `&self`、`&mut self`、`self` 分别代表什么

## 2. 学这一章前，你需要知道什么

你最好已经理解：

- 结构体 `struct`
- 函数参数和返回值
- 借用与可变借用

这里先带着一句话往下看：

`impl` 不是“类”的完全等价物，但它确实承担了“给类型添加行为”的职责。

## 3. 为什么需要方法

先看一个普通函数写法：

```rust
struct User {
    name: String,
}

fn greet(user: &User) {
    println!("Hello, {}!", user.name);
}

fn main() {
    let user = User {
        name: String::from("Alice"),
    };

    greet(&user);
}
```

这当然可以。

但如果某个行为天然属于 `User`，很多时候写成方法会更自然：

```rust
struct User {
    name: String,
}

impl User {
    fn greet(&self) {
        println!("Hello, {}!", self.name);
    }
}

fn main() {
    let user = User {
        name: String::from("Alice"),
    };

    user.greet();
}
```

这读起来更像：

- “这个用户自己会打招呼”

而不是：

- “外面有个函数拿用户来处理”

## 4. `impl` 是什么

`impl` 可以先简单理解成：

- 给某个类型补充方法或关联函数的代码块

例如：

```rust
struct Counter {
    value: i32,
}

impl Counter {
    fn show(&self) {
        println!("{}", self.value);
    }
}
```

这里的意思就是：

- 为 `Counter` 这个类型定义行为

## 5. `self`、`&self`、`&mut self`

这是这一章最关键的部分。

### 5.1 `&self`

表示：

- 借用当前对象
- 只读访问
- 不拿走所有权

```rust
struct User {
    name: String,
}

impl User {
    fn greet(&self) {
        println!("Hello, {}!", self.name);
    }
}
```

这里 `self` 可以理解成“当前这个实例自己”。

### 5.2 `&mut self`

表示：

- 可变借用当前对象
- 允许修改内部状态

```rust
struct Counter {
    value: i32,
}

impl Counter {
    fn increment(&mut self) {
        self.value += 1;
    }
}

fn main() {
    let mut counter = Counter { value: 0 };
    counter.increment();
    println!("{}", counter.value);
}
```

这里要注意：

- 方法签名里是 `&mut self`
- 调用它的实例绑定也要是 `mut`

### 5.3 `self`

表示：

- 直接拿走当前实例所有权

```rust
struct User {
    name: String,
}

impl User {
    fn into_name(self) -> String {
        self.name
    }
}
```

这表示：

- 这个方法调用后，原实例会被消费掉

这和前面你学的 move 是一致的。

## 6. 关联函数

不是所有在 `impl` 里的函数都必须接收 `self`。

例如：

```rust
struct User {
    name: String,
    age: u32,
}

impl User {
    fn new(name: String, age: u32) -> Self {
        Self { name, age }
    }
}

fn main() {
    let user = User::new(String::from("Alice"), 20);
    println!("{}", user.name);
}
```

这里的 `new` 没有 `self` 参数，所以它不是实例方法，而是关联函数。

调用方式是：

- `User::new(...)`

这种写法很常见，尤其是构造函数风格。

## 7. `Self` 是什么

在 `impl User` 里，`Self` 指的就是当前类型本身，也就是 `User`。

所以：

```rust
fn new(name: String, age: u32) -> Self
```

可以理解成：

```rust
fn new(name: String, age: u32) -> User
```

`Self` 的好处是：

- 更简洁
- 类型名改了时更不容易到处改漏

## 8. 一个稍微完整一点的例子

```rust
struct StudyPlan {
    title: String,
    hours: i32,
}

impl StudyPlan {
    fn new(title: String) -> Self {
        Self { title, hours: 0 }
    }

    fn add_hours(&mut self, h: i32) {
        self.hours += h;
    }

    fn summary(&self) {
        println!("{}: {} hours", self.title, self.hours);
    }
}

fn main() {
    let mut plan = StudyPlan::new(String::from("Rust Basics"));
    plan.add_hours(3);
    plan.summary();
}
```

这个例子里三种常见行为都出现了：

- `new`：关联函数
- `add_hours`：可变方法
- `summary`：只读方法

## 9. 方法语法为什么重要

因为随着代码变大，你会越来越需要：

- 让结构更清晰
- 让数据和行为靠近
- 让代码读起来像“对象自己在做什么”

这会比所有逻辑都散成独立函数更容易维护。

## 10. 和 JavaScript、Python、Java 的类比

### 和 JavaScript 类比

如果你写过对象方法或类方法，会很快接受“实例可以调用方法”这件事。

但 Rust 和 JavaScript 的差别在于：

- Rust 会非常明确地区分只读借用、可变借用、拿走所有权

### 和 Python 类比

Python 方法里经常写 `self`，这一点会让你很容易产生熟悉感。

但 Rust 的 `self` 系列更细：

- `&self`
- `&mut self`
- `self`

这背后直接对应所有权和借用规则。

### 和 Java 类比

Java 初学者对“字段 + 方法”会很熟悉，但 Rust 没有传统 OOP 里那套完全相同的类体系。`impl` 更像“给类型组织行为”，而不是把一整套面向对象机制照搬过来。

## 11. 常见错误与如何调试

### 错误 1：想修改字段，却把方法写成 `&self`

```rust
fn increment(&self) {
    self.value += 1;
}
```

如果你要修改，就应该考虑是不是该改成 `&mut self`。

### 错误 2：方法是 `&mut self`，但实例没写 `mut`

```rust
let counter = Counter { value: 0 };
counter.increment();
```

这会报错，因为实例本身不是可变绑定。

### 错误 3：把关联函数当实例方法调

```rust
let user = User::new(...);
user.new(...);
```

`new` 这类不接收 `self` 的函数，通常应该写成：

- `User::new(...)`

### 错误 4：`self` 方法调用后还想继续用旧值

如果方法签名拿走了 `self`，那实例就被消费了。你需要回头确认自己到底是想借用、修改，还是直接消耗掉这个对象。

## 12. 小练习

1. 定义一个 `Counter` 结构体，并给它写 `increment` 方法。
2. 给一个 `User` 结构体写 `greet(&self)` 方法。
3. 给一个 `Book` 结构体写 `new(...) -> Self` 关联函数。
4. 写一个拿走 `self` 的方法，返回其中某个字段。

## 13. 挑战题

做一个“学习计划对象”，要求：

1. 定义 `StudyPlan` 结构体
2. 字段至少包含标题、累计学习小时数、是否完成
3. 用 `impl` 为它写：
   `new`
   `add_hours`
   `mark_done`
   `show`
4. 在 `main` 中创建实例并连续调用这些方法

额外要求：

1. 至少写一个 `&self` 方法
2. 至少写一个 `&mut self` 方法
3. 至少写一个关联函数

## 14. 小结

这一章最关键的理解是：

1. `impl` 用来给类型组织行为
2. `&self` 表示只读借用当前实例
3. `&mut self` 表示可变借用当前实例
4. `self` 表示直接拿走当前实例
5. 关联函数常用来做构造和辅助逻辑

下一篇建议进入：

- [模块、包与文件组织](./13-模块、包与文件组织.md)
