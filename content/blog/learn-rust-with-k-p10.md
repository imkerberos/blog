+++
title = "陪老 K 学 Rust (十)"
author = ["Evilee"]
date = 2020-01-16
lastmod = 2020-01-16T16:38:59+08:00
tags = ["ssh", "gfw"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
+++

`Option` 枚举
<!--more-->

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

`unwrap` 操作是解出 `Option` 里面的值，如果 `Option` 是空值，则会引发 `panic` 从而导致程序崩溃。 `unwrap` 通常用在对于质量不高或者简短的示例代码中，在这些代码中，我们通常不太关注错误，但是在实际的产品级别的代码中，需要小心对 `Option` 值进行检查或者模式匹配。 除了模式匹配， Rust 还提供了一些更方便的函数来处理 `Option` 值。

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
