+++
title = "陪老 K 学 Rust (九)"
author = ["Evilee"]
date = 2020-01-09
lastmod = 2020-01-10T13:59:50+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1003
+++

枚举。
<!--more-->


## <span class="section-num">1</span> 基本形式 {#基本形式}

经典形式

```rust
enum Direction {
    North,
    East,
    Sourth,
    West,
}
```

元组形式关联值：

```rust
enum NonamedShape {
    Square(u32),
    Rectangle(u32, u32),
    Circle(u32),
}
```

记录形式关联值：

```rust
enum NamedShape {
    Square {
        width: u32,
    },
    Rectangel {
        width: u32,
        height: u32,
    },
    Circle {
        radio: u32,
    },
}
```

经典形式可以当作是元组形式的特殊形式，毕竟 `struct NoFieldTuple;` 是`struct NoField ();` 的简写。

混合形式，不同形式的类型结构可以混合起来组成单一的枚举类型。

```rust
enum HybridShape {
    Dot,
    Square(u32),
    Rectangle {
        width: u32,
        height: u32
    },
    Circle(u32),
}
```


## <span class="section-num">2</span> 解构 {#解构}

```rust
enum NamedShape {
    Square {
        width: u32,
    },
}

fn main() {
    let NamedShape::Square{width} = NamedShape::Square{width: 14};
    println!("width: {}", width);
}
```

运行输出:

```text
width: 14
```

因为 `NamedShape` 只有一种情况： `Square`, 所以可以正常解构到 `width` 变量上。但是如果 `NamedShape` 有多种情况，会发生什么呢？

```rust
enum NamedShape {
    Square {
        width: u32,
    },
    Rectangle {
        width: u32,
        height: u32
    },
}

fn main() {
    let NamedShape::Square{width} = NamedShape::Square{width: 14};
    println!("width: {}", width);
}
```

编译报错：

```text
error[E0005]: refutable pattern in local binding: `Rectangle { .. }` not covered
  --> r43.rs:12:9
   |
1  | / enum NamedShape {
2  | |     Square {
3  | |         width: u32,
4  | |     },
5  | |     Rectangle {
   | |     --------- not covered
...  |
8  | |     },
9  | | }
   | |_- `NamedShape` defined here
...
12 |       let NamedShape::Square{width} = NamedShape::Square{width: 14};
   |           ^^^^^^^^^^^^^^^^^^^^^^^^^ pattern `Rectangle { .. }` not covered
   |
   = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
   = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
help: you might want to use `if let` to ignore the variant that isn't matched
   |
12 |     if let NamedShape::Square{width} = NamedShape::Square{width: 14} { /* */ }
   |

error[E0381]: borrow of possibly-uninitialized variable: `width`
  --> r43.rs:13:27
   |
13 |     println!("width: {}", width);
   |                           ^^^^^ use of possibly-uninitialized `width`

error: aborting due to 2 previous errors

Some errors have detailed explanations: E0005, E0381.
For more information about an error, try `rustc --explain E0005`.
```

所以对枚举进行解构不能象对元组或者结构一样进行，必须先进行模式匹配，确定数据的类型结构，再进行解构或者绑定。


## <span class="section-num">3</span> 模式匹配与解构 {#模式匹配与解构}


## <span class="section-num">4</span> 模式匹配与绑定 {#模式匹配与绑定}


## <span class="section-num">5</span> Option 枚举 {#option-枚举}


## <span class="section-num">6</span> Result 枚举 {#result-枚举}


## <span class="section-num">7</span> 波动拳和 `guard` {#波动拳和-guard}
