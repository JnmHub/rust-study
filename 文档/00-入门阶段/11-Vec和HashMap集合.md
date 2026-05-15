# Vec 和 HashMap 集合

## 1. 这一章是干什么的

前面你已经学过数组、元组、结构体、枚举。

但真正写程序时，你会很快遇到这类需求：

- 一组数据数量不固定，怎么办？
- 想保存很多同类型元素，怎么办？
- 想通过“键”快速找到对应值，怎么办？

这一章要解决的就是这些现实问题。

Rust 里最常见的两个基础集合是：

- `Vec<T>`
- `HashMap<K, V>`

如果你把前面学的内容理解成“一个值怎么表示”，这一章就是“很多值怎么组织”。

## 2. 学这一章前，你需要知道什么

你最好已经理解：

- 数组和元组的区别
- 所有权和借用的第一轮概念
- `Option` 和 `Result` 的基础用法

这里先带着一个问题往下看：

为什么前面学过数组了，Rust 还要单独强调 `Vec<T>`？

因为数组长度固定，而现实程序里的数据量经常是动态变化的。

## 3. 为什么需要 `Vec<T>`

数组适合长度固定的数据：

```rust
fn main() {
    let numbers = [1, 2, 3, 4];
    println!("{}", numbers[0]);
}
```

但如果你想：

- 后面继续追加元素
- 运行时才知道有多少项
- 数据量会变化

数组就不够灵活了。

这时更适合用 `Vec<T>`，也就是动态数组。

## 4. `Vec<T>` 的基本写法

### 4.1 创建向量

```rust
fn main() {
    let numbers = vec![1, 2, 3];
    println!("{:?}", numbers);
}
```

这里的 `vec!` 是一个宏，用来方便地创建向量。

也可以创建空向量：

```rust
fn main() {
    let mut numbers: Vec<i32> = Vec::new();
    numbers.push(1);
    numbers.push(2);

    println!("{:?}", numbers);
}
```

这里需要注意：

- 空向量初始时看不出元素类型
- 所以通常要显式写 `Vec<i32>`

### 4.2 追加元素

```rust
fn main() {
    let mut items = vec![10, 20];
    items.push(30);

    println!("{:?}", items);
}
```

因为向量内容会变，所以变量绑定通常要是 `mut`。

## 5. 读取 `Vec<T>` 中的元素

### 5.1 用下标访问

```rust
fn main() {
    let items = vec![10, 20, 30];
    println!("{}", items[1]);
}
```

这很直接，但有风险：

- 如果下标越界，程序会 panic

### 5.2 用 `get()` 更稳妥

```rust
fn main() {
    let items = vec![10, 20, 30];

    match items.get(1) {
        Some(value) => println!("{}", value),
        None => println!("not found"),
    }
}
```

这里 `get()` 返回的是 `Option<&T>`。

这就和你上一章学过的 `Option` 连起来了：

- 找到了：`Some`
- 没找到：`None`

这也是 Rust 很典型的风格：边界情况写进类型，不靠侥幸。

## 6. 遍历 `Vec<T>`

### 6.1 只读遍历

```rust
fn main() {
    let items = vec![10, 20, 30];

    for item in &items {
        println!("{}", item);
    }
}
```

这里用的是 `&items`，表示借用遍历，不拿走原向量所有权。

### 6.2 可变遍历

```rust
fn main() {
    let mut items = vec![10, 20, 30];

    for item in &mut items {
        *item += 1;
    }

    println!("{:?}", items);
}
```

这里第一次会让很多新手看到 `*item` 有点别扭。

你现在先不用把它想太深，只先记住：

- `item` 是对元素的可变引用
- 你想改里面的值，就要通过引用去改

## 7. 为什么需要 `HashMap<K, V>`

如果你想按“键”找值，`Vec<T>` 就不合适了。

例如：

- 用户名对应分数
- 单词对应出现次数
- 配置项名对应配置值

这时更适合：

- `HashMap<K, V>`

它可以理解成“键值对集合”。

## 8. `HashMap<K, V>` 的基本写法

### 8.1 创建 `HashMap`

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Alice"), 95);
    scores.insert(String::from("Bob"), 88);

    println!("{:?}", scores);
}
```

这里要注意：

- `HashMap` 不在 prelude 里
- 所以通常要先 `use std::collections::HashMap;`

### 8.2 读取值

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Alice"), 95);

    let name = String::from("Alice");

    match scores.get(&name) {
        Some(score) => println!("{}", score),
        None => println!("not found"),
    }
}
```

这里 `get()` 返回的也是 `Option<&V>`。

所以你会发现：

- 集合里“可能找不到”的情况
- Rust 通常都要求你显式处理

### 8.3 遍历 `HashMap`

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Alice"), 95);
    scores.insert(String::from("Bob"), 88);

    for (name, score) in &scores {
        println!("{} -> {}", name, score);
    }
}
```

这里和元组解构有点像，因为每一项本质上是键和值的组合。

## 9. `Vec<T>` 和 `HashMap<K, V>` 什么时候用

### 更适合 `Vec<T>` 的场景

- 顺序很重要
- 主要按位置遍历
- 数据类型一致
- 数据数量会变化

例如：

- 待办事项列表
- 学习任务列表
- 一批分数

### 更适合 `HashMap<K, V>` 的场景

- 你想按键查值
- 数据天然是“名称 -> 内容”
- 重点不是位置，而是映射关系

例如：

- 单词频次
- 配置项
- 用户名到积分

## 10. 和 JavaScript、Python、Java 的类比

### 和 JavaScript 类比

你可以粗略理解为：

- `Vec<T>` 有点像数组
- `HashMap<K, V>` 有点像 `Map`

但 Rust 比 JavaScript 更强调元素类型一致、借用规则和缺值处理。

### 和 Python 类比

你可以粗略理解为：

- `Vec<T>` 类似 `list`
- `HashMap<K, V>` 类似 `dict`

但 Rust 不会像 Python 那样默认放任你在运行时再慢慢发现类型和边界问题。

### 和 Java 类比

你可以粗略理解为：

- `Vec<T>` 有点像 `ArrayList<T>`
- `HashMap<K, V>` 和 Java 的 `HashMap<K, V>` 很接近

不过 Rust 在读取时常返回 `Option`，这比很多语言“直接给空值或抛错”的风格更显式。

## 11. 常见错误与如何调试

### 错误 1：把 `Vec<T>` 当固定数组理解

它和数组都能装一串同类型元素，但它们不是一回事：

- 数组长度固定
- `Vec<T>` 长度可变

### 错误 2：越界访问

```rust
let items = vec![1, 2, 3];
println!("{}", items[10]);
```

这会 panic。

如果你不确定下标是否存在，优先考虑 `get()`。

### 错误 3：忘了 `HashMap` 需要 `use`

```rust
let map = HashMap::new();
```

如果没有导入，通常会直接报找不到类型。

### 错误 4：拿到 `Option` 却不处理

无论是 `Vec::get()` 还是 `HashMap::get()`，都要记得：

- 结果可能是 `Some`
- 也可能是 `None`

### 错误 5：遍历时不小心 move 集合

如果你写：

```rust
for item in items {
    println!("{}", item);
}
```

有时这意味着你把 `items` 本身消耗掉了。初学时如果只是想看，不妨优先用 `&items` 遍历。

## 12. 小练习

1. 创建一个 `Vec<i32>`，追加 3 个数字并打印。
2. 用 `get()` 读取一个存在下标和一个不存在下标，观察 `Some` / `None`。
3. 创建一个 `HashMap<String, i32>`，存两个人的分数。
4. 用 `match` 或 `if let` 读取某个键对应的值。

## 13. 挑战题

做一个“学习记录面板”，要求：

1. 用 `Vec<String>` 保存本周学习主题
2. 用 `HashMap<String, i32>` 保存每个主题对应的学习小时数
3. 打印所有主题
4. 查询一个指定主题的学习时长
5. 如果查不到，要明确打印提示

额外要求：

1. 至少用一次 `get()`
2. 至少用一次 `match`
3. 至少用一次借用遍历

## 14. 小结

这一章最关键的理解是：

1. `Vec<T>` 适合动态长度、同类型、有顺序的数据
2. `HashMap<K, V>` 适合按键查值
3. `get()` 返回 `Option`，需要显式处理
4. 集合遍历时也要留意所有权和借用
5. Rust 的集合设计和前面学的所有权、借用、`Option` 是连在一起的

下一篇建议进入：

- [方法与 impl](./12-方法与impl.md)
