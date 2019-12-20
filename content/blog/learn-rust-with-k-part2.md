+++
title = "跟老 K 一起学 Rust (二)"
author = ["Eviler"]
date = 2019-12-18
lastmod = 2019-12-20T19:19:18+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
+++

<!--more-->


## <span class="section-num">1</span> 宏 {#宏}

```rust
fn main() {
    println!("Hello, world!");
}
```

很简单的 `main` 函数，跟 C 语言的 `hello world` 程序差不多。 `fn` 表示 `main` 是一个函数，它没有参数，也没有返回值（严格来讲，其返回值是 `()` ）。 `println!` 看上去是一个函数，实际上是一个宏，宏是一段运行在编译器上的代码。对，跟 C/C++ 的宏类似，但是从功能上来说，Rust 的宏比 C/C++ 的宏更加强大。 宏和函数的区别可以通过如下的例子来理解。

假如我们有一个 `println` 函数，它类似于 C 语言的 `printf` 函数，接受可以格式化的字符串参数，可能会这样调用：

```rust
println("This is a string format print: name: {}, value: {}", name, value);
```

从编译器的角度来看， `println` 函数的第一个参数是一个字符串，其内部的插值占位符
`{}`, 编译器是无法理解的，这样的后果就是我们即使向 `println` 函数中传入 3 或者 4
个参数，编译器在编译阶段也不会报错。但是宏不一样，我们可以编写一段代码来操纵编译器，使之能理解 `println` 函数的第一个字符串参数内部的占位符，从而对后面的参数个数以及类型进行检查，一旦码农传入了非法的参数，在编译阶段就可以检查出错误来。那这段代码就是 `println!` 宏，而且比 C 语言中的 `printf` 更强大，因为 `printf` 函数是无法对参数进行合法性检查的。

> 思考：
>
> 是不是可以编写一个执行数据库检查的宏： `execute_sql!("select name, age from user_table where age < {}", min_age);` 不仅能对格式化的参数合法性进行检查，甚至能对其内部的 SQL 语句的合法性进行检查？


## <span class="section-num">2</span> `Trait` 和 `Display` {#trait-和-display}

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


## <span class="section-num">3</span> 分号 {#分号}

```rust
impl std::fmt::Display for Person {
    fn fmt(&self, fmt: &mut std::fmt::Formatter) -> std::result::Result<(), std::fmt::Error> {
        write!(fmt, "{} ({} yeas old)", self.name, self.age)
    }
}
```

这段代码中的函数 `fmt` 函数体中只有一个语句： `write!`, 而且这一个语句的后面 **没有**
分号！并且此函数明确标明了返回一个 `Result` 类型的值，但是函数体内部并没有
`return`. 这不是错误。Rust 是一个基于表达式的语言，也就是每一个 Rust 的语句都可以 **求值**. 在 Rust 中，语句分为两种：

1.  声明语句
2.  表达式语句


## <span class="section-num">4</span> 数字类型 {#数字类型}


## <span class="section-num">5</span> 循环打印数字 {#循环打印数字}
