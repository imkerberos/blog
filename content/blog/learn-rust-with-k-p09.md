+++
title = "陪老 K 学 Rust (九)"
author = ["Evilee"]
date = 2020-01-09
lastmod = 2020-01-16T16:38:06+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
+++

枚举
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

使用 `match` 进行模式匹配：

```rust
#[derive(Debug)]
enum HybridShape {
    Square(u32),
    Rectangle{ width:u32, height:u32},
    Circle(u32),
}

fn main() {
    use HybridShape::*;
    let shape = Rectangle{width: 20, height: 40};
    match shape {
        Square(width) => {
            println!("match square, width: {}", width);
        },
        Rectangle{width, height} => {
            println!("match Rectangle{{width: {}, height: {}}}", width, height);
        },
        Circle(radio) => {
            println!("match Circle(u32), radio: {}", radio);
        }
    }
}
```

运行输出：

```text
warning: variant is never constructed: `Square`
 --> r44.rs:3:5
  |
3 |     Square(u32),
  |     ^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: variant is never constructed: `Circle`
 --> r44.rs:5:5
  |
5 |     Circle(u32),
  |     ^^^^^^^^^^^

match Rectangle{width: 20, height: 40}
```

> `use HybridShape::*` 这句代码是在函数 `main` 中引入
> `HybridShape` 的名字空间，否则我们使用 `HybridShape` 的内部类型时需要显式指明，如 `HybridShape::Square`, `HybridShape::Rectangle` 等。

<!--quoteend-->

> 因为直接使用 `Rectangle{width: 20, height:40}` 构造的
> `shape`, 故而编译器可以检测出其具体类型为 `HybridShape::Rectangle`, 警告我们第三行和第 5 行 `Square` 和 `Circle` 这两种类型的变体是从来不会被匹配到的，这里只是演示代码，所以忽略即可。


## <span class="section-num">4</span> 模式匹配与绑定 {#模式匹配与绑定}

在匹配的同时进行绑定：

```rust
#[derive(Debug)]
enum HybridShape {
    Square(u32),
    Rectangle{ width:u32, height:u32},
    Circle(u32),
}

fn main() {
    use HybridShape::*;
    let shape = Circle(50);
    match shape {
        Square(width) => {
            println!("match square, width: {}", width);
        },
        Rectangle{width, height} => {
            println!("match Rectangle{{width: {}, height: {}}}", width, height);
        },
        Circle(radio @ 0..=100) => {
            println!("match Circle(u32 @ 0..=100), radio: {}", radio);
        }
        Circle(radio) => {
            println!("match Circle(u32), radio: {}", radio);
        }
    }
}
```

运行输出：

```text
 --> r45.rs:3:5
  |
3 |     Square(u32),
  |     ^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: variant is never constructed: `Rectangle`
 --> r45.rs:4:5
  |
4 |     Rectangle{ width:u32, height:u32},
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

match Circle(u32 @ 0..=100), radio: 50
```

同前一段代码类似，忽略警告。
