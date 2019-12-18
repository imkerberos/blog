+++
title = "跟老 K 一起学 Rust (二)"
author = ["Eviler"]
date = 2019-12-18
lastmod = 2019-12-18T23:33:08+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
+++

`Trait` 和 `Display`

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
