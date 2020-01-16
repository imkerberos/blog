+++
title = "陪老 K 学 Rust (九)"
author = ["Evilee"]
date = 2020-01-09
lastmod = 2020-01-16T15:38:29+08:00
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


## <span class="section-num">5</span> Option 枚举 {#option-枚举}

`Option` 是一个枚举，虽然在实现上，其并无特殊之处，但是在地位上，它和 `Result`
都比较特殊，以至于 Rust 都对其制作了一些语法糖来方便开发者使用。

`Option` 的诞生是为了解决空值问题。我们知道，在 C/C++ 甚至 Java, Javascript 中，开发者都面临者 `空值` 问题带来的挑战。在 C/C++ 里，开发者需要小心处理 `空指针`,
但是仍然难以避免出现错误，以至于后来部分语言都弃用了 `指针` 这个数据类型。不幸的是，即使如此，Javasript 还需要面对 `空值` 的问题。

Rust 中对于 `空值` (不仅仅是空指针), 吸收了其他某些语言（haskell?）的优点，引入了 `Option` 这种处理方式，简而言之，就是一个新的枚举类型，要么它是一个空值，要么它是一个具体的值。通过使用时需要强制解包（unwrap）来规范开发者使用的策略，避免出现空值引发的错误。

```rust
enum Option<T> {
    None,
    Some(T),
}
```

以上示意代码就是 `Option` 的定义，其中 `Option<T>` 使用了 `泛型` 语法。 以下代码是 `Option` 的一个非常典型的应用场景。

```rust
fn division(dividend: i32, divisor: i32) -> Option<i32> {
    if divisor == 0 {
        None
    } else {
        Some(dividend / divisor)
    }
}

fn main() {
    let r1 = division(4, 2);
    println!("4 / 2 = {:?}", r1);
    let r0 = division(4, 0);
    println!("4 / 2 = {:?}", r0);

    match r1 {
        None => {
            println!("r1 is None");
        },
        Some(value) => {
            println!("r1's value is: {}", value);
        }
    }

    println!("r1's value is: {}", r1.unwrap());
    println!("r0's value is: {}", r0.unwrap());
}
```

运行输出：

```text
4 / 2 = Some(2)
4 / 2 = None
r1's value is: 2
r1's value is: 2
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/libcore/option.rs:378:21
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

divison 函数对输入的两个 `i32` 的值进行除法操作，并返回结果。如果我们在 C 语言中，会如何处理呢？我们需要输出两个值：一个值表示输入是否有效，另外一个值表示输出的结果。并且需要对返回值的有效性进行判断，因为一旦忘记有效性的判断，就引发逻辑性的错误（因为 ret 的值是无效的）。

```c
#include<stdio.h>
int division(int dividend, int divisor, int* result) {
    if (divisor == 0) {
        return -1;
    }
    if (result) {
        *result = dividend / divisor;
    }

    return 0;
}

int main() {
    int ret;
    int err;
    err = division(4, 2, &ret);
    if (err < 0) {
        printf("divid zero!\n");
    } else {
        printf("4 / 2 = %d\n", ret);
    }

    err = division(4, 0, &ret);
    if (err < 0) {
        printf("divid zero!\n");
    } else {
        printf("4 / 0 = %d\n", ret);
    }
}
```

对于 Rust 而言，不进行 `unwrap` 或者 模式匹配是无法进行下一步操作的，故而对于开发者来说，是相对安全的。

`unwrap` 操作是解出 `Option` 里面的值，如果 `Option` 是空值，则会引发 `panic` 从而导致程序崩溃。 `unwrap` 通常用在对于质量不高或者简短的示例代码中，在这些代码中，我们通常不太关注错误，但是在实际的产品级别的代码中，需要小心对 `Option` 值进行检查或者模式匹配。 除了模式匹配， Rust 还提供了一些更方便的宏来处理 `Option` 值。

1.  `unwrap_or` : 如果是 `None` 则赋予一个默认值。
2.  `unwrap_or_default`: 如果是 `None` 则赋予类型的缺省值。

<!--listend-->

```rust
fn division(dividend: i32, divisor: i32) -> Option<i32> {
    if divisor == 0 {
        None
    } else {
        Some(dividend / divisor)
    }
}

fn main() {
    let r1 = division(4, 2);
    println!("4 / 2 = {:?}", r1);
    let r0 = division(4, 0);
    println!("4 / 2 = {:?}", r0);

    match r1 {
        None => {
            println!("r1 is None");
        },
        Some(value) => {
            println!("r1's value is: {}", value);
        }
    }

    println!("r1's value is: {}", r1.unwrap());
    println!("r0's unwrap_or: {}", r0.unwrap_or(-1));
    println!("r0's unwrap_or_default: {}", r0.unwrap_or_default());
}
```

老使用 `unwrap` 和 `match` 系列的确挺影响心情的，代码上也不好看，还有一种 `函数式` 的链式调用处理方式比较符合正常的心智模型，在以后学习到 Rust 的 `函数式` 特性时再明确。


## <span class="section-num">6</span> Result 枚举 和错误处理 {#result-枚举-和错误处理}

`Option` 可以帮助开发者解决 `空值` 问题，但是 `空值` 并不能帮开发者进行错误处理，在绝大部分情况下，一旦代码出错，开发者需要根据错误的详细信息进行处理，做现场恢复还是使用备选方案，亦或进行日志输出以帮助解决问题。 这些方案，用 `空值` 是无法满足需求的。但假如把 `Option` 的 `None` 替换成具体的错误信息 `Err`,开发人员根据 `Err` 来进行错误处理，这样 `Result` 这个枚举就诞生了。形如：

```rust
enum Result<T,E> {
    Ok(T),
    Err(E),
}
```

当然，针对 `Option` 的 `unwrap` 和 `unwrap_or` 等就不能用于 `Result` 了。

Rust 中并没有 `Exception` 机制，函数的出错全靠 `Result` 来进行判断，这样就要求开发者：

1.  永远不要调用 `panic`, 给调用你 API 的开发者以处理的机会。
2.  永远不要调用导致 `panic` 的代码，如 `unwrap`.
3.  如果可能，函数尽量使用 `Result` 作为返回值，以方便其他开发者处理。


## <span class="section-num">7</span> 波动拳和 `guard` {#波动拳和-guard}
