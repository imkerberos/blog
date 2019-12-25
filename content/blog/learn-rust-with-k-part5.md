+++
title = "陪老 K 学 Rust (五)"
author = ["Eviler"]
lastmod = 2019-12-25T18:53:07+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
+++

可变变量与可变引用
<!--more-->

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn uses_foobar(foobar: &Foobar) {
    println!("I consumed a Foobar: {:?}", foobar);
}

fn main() {
    let mut x = Foobar(0);
    let mut z = Foobar(1);
    let mut y = &mut x;
    uses_foobar(y);
    y.0 = 1;
    uses_foobar(y);

    y = &mut z;
    uses_foobar(y);
    y.0 = 3;
    uses_foobar(y);
}
```

在很多语言中， `let` 的含义并不是声明，而是进行一个值 `绑定`. 实际上就我的理解来说，Rust 中的 `let` 理解成绑定更加贴切。

还记得著名的 C 的 `const` 问题吗？ `const char * const p = &str;`, 我们就以分析 `const` 的方法来分析 `mut` 关键字:

1.  `let mut x: Foobar = Foobar(0);` 这种形式中， `mut` 修饰的是绑定关系还是值本身？ `mut` 既修饰变量和值的绑定关系，又修饰值本身。
2.  `let mut y: &mut Foobar = &mut x;` 这种形式中，第一个
    `mut` 限定的是绑定关系，也就是 `y` 可以是 `x` 的引用绑定，也可以是其他值的引用绑定。 第二个 `mut` 限定的是值本身，即值本身的内容是否可以被修改。第三个
    `mut` 的作用等同于第二个 `mut`, 在使用类型推断的情况下，这一点就更为明显：
    `let mut y = &mut x;`.
