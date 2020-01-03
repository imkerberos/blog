+++
title = "陪老 K 学 Rust (七)"
author = ["Evilee"]
lastmod = 2020-01-03T00:13:15+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
+++

结构体和枚举。
<!--more-->


## <span class="section-num">1</span> 结构体的形式 {#结构体的形式}

元组形式

```rust
struct NoField;
struct OneField(i32);
struct TowField(u32, i32);
```

使用方式: 索引方式解构方式: 可变解构，引用解构，可变引用解构. 多余的解构，不足的解构.
匹配方式: 可变匹配，引用匹配，可变引用匹配。

记录形式

```rust
struct NoField {}
struct OneField {
    age: i32,
}
struct TowField {
    age: i32,
    score: i32,
}
```


## <span class="section-num">2</span> 枚举的形式 {#枚举的形式}

经典形式

```rust
enum Direction {
    North,
    East,
    Sourth,
    West,
}
```

元组形式关联值

```rust
enum NonamedShape {
    Square(u32),
    Rectangle(u32, u32),
    Circle(u32),
}
```

记录形式关联值

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

混合形式

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


## <span class="section-num">3</span> 模式解构 {#模式解构}

元组的解构记录的解构可变解构引用解构


## <span class="section-num">4</span> 模式匹配解构 {#模式匹配解构}
