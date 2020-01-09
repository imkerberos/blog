+++
title = "陪老 K 学 Rust (九)"
author = ["Evilee"]
date = 2020-01-09
lastmod = 2020-01-09T11:40:08+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1002
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
