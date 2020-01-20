+++
title = "陪老 K 学 Rust (十一)"
author = ["Evilee"]
date = 2020-01-16
lastmod = 2020-01-20T16:21:08+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
+++

`Result` 枚举
<!--more-->

`Option` 可以帮助开发者解决 `空值` 问题，但是 `空值` 并不能帮开发者进行错误处理，在绝大部分情况下，一旦代码出错，开发者需要根据错误的详细信息进行处理，做现场恢复还是使用备选方案，亦或进行日志输出以帮助解决问题。 这些方案，用 `空值` 是无法满足需求的。

但假如把 `Option` 的 `None` 替换成具体的错误信息 `Err`,开发人员根据 `Err` 来进行错误处理，这样 `Result` 这个枚举就产生了。形如：

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

`Result` 中的两个关联值，其中一个是泛型类型为 `T` 的 `Ok`, 另外一个是泛型类型为
`E` 的 `Err`. `T` 类型比较好理解，就是值的类型， `E` 类型呢？是否应该是某些特定的类型？


## <span class="section-num">1</span> 以前的代码从 `Option` 转换成 `Result` {#以前的代码从-option-转换成-result}


## <span class="section-num">2</span> 在 `main` 函数中使用 `Result` {#在-main-函数中使用-result}


## <span class="section-num">3</span> `Result` 处理的方案 {#result-处理的方案}

实际上，Rust 的错误处理方案也是改了又该，一直不完美。无论是 error\_chain 还是
