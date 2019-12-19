+++
title = "跟老 K 一起学 Rust (二)"
author = ["Eviler"]
date = 2019-12-18
lastmod = 2019-12-19T12:00:51+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
+++

## <span class="section-num">1</span> `Trait` 和 `Display` {#trait-和-display}

<!--more-->

<a id="code-snippet--程序一"></a>
```rust
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let alice = Person {
        name: String::from("Alice"),
        age: 30,
    };
    println!("Person: {}", alice);
}
```

编译报错：

```text
error[E0277]: `Person` doesn't implement `std::fmt::Display`
  --> t001.rs:11:28
   |
11 |     println!("Person: {}", alice);
   |                            ^^^^^ `Person` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Person`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`
```

错误的原因是 Person 没有实现 `std::fmt::Display` Trait. 实现这个 Trait 就能够修复这个错误。

```rust
impl std::fmt::Display for Person {
    fn fmt(&self, fmt: &mut std::fmt::Formatter) -> std::result::Result<(), std::fmt::Error> {
        write!(fmt, "{} ({} yeas old)", self.name, self.age)
    }
}
```

结论：

1.  Rust 中没有面向对象的概念， `trait` 也不是 `class`, **没有继承**!
2.  Rust 使用 `{}` 进行字符串插值时，被插值参数必须要实现 `std::fmt::Display` Trait.
3.  `&self` 是 `self: &Self` 的语法糖。
4.  `()` 类似 C 语言中的 `void`, 不同的是 `()` 既是类型，也是值。
5.  命名约定： 宏都以 `!` 结尾。
6.  与 C++ 不同，Rust 用 `::` 来表示域，C++ 用 `:` 。
7.  `&` 表示使用 `引用` 的方式传参，这一点和 C++ 类似。

> 作为一个老鸟，肯定会思考：既然字符串插值的占位符是 `{}`, 那如果要打印原始的 `{}` 该如何转义呢？
>
> 猜一下， 是 `{{{}` ? 不美观，而且看样占位符实际上是两个字符: `{` 和 `}`, 美观点也应该是： `{{}}`.
>
> Right!


## <span class="section-num">2</span> 分号 {#分号}


## <span class="section-num">3</span> 数字类型 {#数字类型}


## <span class="section-num">4</span> 循环打印数字 {#循环打印数字}
