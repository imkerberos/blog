+++
title = "陪老 K 学 Rust (二)"
author = ["Evilee"]
date = 2019-12-20
lastmod = 2020-01-09T15:27:52+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1010
+++

万年的 `Hello World!`.
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

<a id="代码--程序一"></a>
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
`return`. 这不是错误。Rust 是一门基于表达式的语言，也就是说，任何 Rust 语句都是一个表达式，表达式的特点就是可以对其进行求值。语句分为两种：

1.  声明语句: 是一种特殊的表达式， `let`, `use`, `crate`, `fn`, `struct`, `trait`, `impl` 等等。这些使用其副作用的语句，其值为 `()`.
2.  表达式语句: 由一个表达式和分号共同组成的语句。在一个表达式后面添加 `;` 就构成了表达式语句。当使用 `;` 把表达式强制变成语句之后，则此表达式语句的值被抑制，强制变为 `()`.

> 思考
>
> 既然 Rust 中一切都是表达式，为什么还要在表达式的后面添加一个 `;` 使之变成表达式语句？

既然在 Rust 中一切皆表达式，则 `{}` 组成的代码块也是表达式。由 `{}` 组成的代码块的表达式的值就是 `{}` 最后一个表达式的值。

```rust
let a = {
    let inner = 2;
    inner * inner
}
```

上面代码中 `a` 的值是 4. 但是如果 ` inner * inner ` 用
`;` 强制转换成语句后， `a` 的值和类型都变成了 `()`.

函数的返回值也一样，在 Rust 的函数体中，最后一个表达式的值作为函数的返回值。 `return` 语句通常用在提前返回的情况下。


## <span class="section-num">4</span> 数字类型 {#数字类型}

Rust 中的数字类型都是明确的，并且类型之间只能使用 `as` 进行显示转换，不允许类似
C 语言那样的隐式转换。Rust 的类型名称也比较有规律：

<div class="ox-hugo-table striped table-striped noboldheader">
<div></div>

| 单字节 | 双字节 | 四字节 | 八字节 | 十六字节 | 四/八字节 |
|-----|-----|-----|-----|------|-------|
| i8  | i16 | i32 | i64 | i128 | isize |
| u8  | u16 | u32 | u64 | u128 | usize |

</div>

再也不用费劲记忆 `short`, `int`, `long`, `longlong` 是多少字节了。:)
`iszie` 和 `usize` 比较特殊一点，想来是为了方便和 C 进行混合编程。


## <span class="section-num">5</span> 循环打印数字 {#循环打印数字}

```rust
fn main() {
    let i = 1;

    loop {
        println!("i == {}", i);
        if i >= 10 {
            break;
        } else {
            i += 1;
        }
    }
}
```

```rust
fn main() {
    let i = 1;

    while i <= 10 {
        println!("i == {}", i);
        i += 1;
    }
}
```

以上代码都有编译错误，主要就是需要注意可变变量和不变变量。 比较奇怪的一点是既然有 `while` 了，为什么还增加一个 `loop`? 一种说法是对于循环来说， `loop` 更方便编译器检查错误，因为只要其中不包含 `break` 语句，就会被编译器检查出来，但是 `while` 语句的条件检查只有在运行期才能知道，编译期是无法知道的，也就无法在编译期进行检查。

```rust
fn main() {
    for i in 1..11 {
        println!("i == {}", i);
    }
}
```

`for` 语句和 C 长得不一样了， Rust 的 `for` 变成了 `迭代` 的形式。
