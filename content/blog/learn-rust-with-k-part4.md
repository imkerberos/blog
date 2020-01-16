+++
title = "陪老 K 学 Rust (四)"
author = ["Evilee"]
date = 2019-12-25
lastmod = 2020-01-16T15:47:18+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1011
+++

借用和引用
<!--more-->


## <span class="section-num">1</span> 词法空间 {#词法空间}

不久以前，Rust 的变量作用域是基于词法的，最近一年（可能）Rust 合并了 `非词法作用域` 生命周期的特性 (NLL, No Lexical Liftime)，使得变量的生命周期不再严格遵循词法域了，关于 NLL 的详细情况可以参考这篇文章：<https://zhuanlan.zhihu.com/p/32884290> .
下面的代码演示了基于词法域的变量生命周期：

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

Fn main() {
    println!("Before x");
    let _x = Foobar(1);
    println!("After x");
    {
        println!("Before y");
        let _y = Foobar(2);
        println!("After y");
    }
    println!("End of main");
}
```

> 在 `x` 和 `y` 变量之前加下划线是为了抑制 Rust 编译器的报错，对于不使用的变量，
> Rust 会发出编译警告。

运行代码可以看出变量 `_x`, `_y` 的生命周期是严格遵循作用域的。

```text
Before x
After x
Before y
After y
Dropping a Foobar: Foobar(2)
End of main
Dropping a Foobar: Foobar(1)
```

如果去掉多余的 `{}`, 猜测一下变量 `_x` 和 `_y` 的生命周期？它们会不是以创建的逆序释放呢？验证一下：

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    println!("Before x");
    let _x = Foobar(1);
    println!("After x");
        println!("Before y");
        let _y = Foobar(2);
        println!("After y");
    println!("End of main");
}
```

```text
Before x
After x
Before y
After y
End of main
Dropping a Foobar: Foobar(2)
Dropping a Foobar: Foobar(1)
```

可以看出，释放是按照创建的 **逆序** 进行的，值得信赖！


## <span class="section-num">2</span> 借用和引用 {#借用和引用}

很多情况下，我们希望在不转移值的所有权(不改变变量的属主）的情况下使用变量。很简单，Rust 提供了一种叫做 `引用` 的机制来满足我们的需求。 `借用` 和 `引用` 是一回事，只是概念的侧重点不一致。 `借用` 是针对 `所有权机制` 而言的。 `引用` 是形式，是针对变量使用的方式而言的。

> 通常变量变量的使用方式遵循两种形式： `值拷贝` 和 `引用`. `值拷贝` 是通过复制一个新的值进行使用，在参数传递（通常的 `值传参`)，赋值等操作中使用. `引用` 是通过值复制值所在的地址进行使用的，典型的应用就是在 `引用传参`, 值共享等场景。

编译下面的代码:

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn uses_foobar(foobar: Foobar) {
    println!("I consumed a Foobar: {:?}", foobar);
}

fn main() {
    let x = Foobar(1);

    uses_foobar(x);
    uses_foobar(x);
}
```

编译器会输出如下的错误：

```text
error[E0382]: use of moved value: `x`
  --> l15.rs:19:17
   |
16 |     let x = Foobar(1);
   |         - move occurs because `x` has type `Foobar`, which does not implement the `Copy` trait
17 |
18 |     uses_foobar(x);
   |                 - value moved here
19 |     uses_foobar(x);
   |                 ^ value used here after move

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`.
```

第二个`uses_foobar(x);` 使用了所有权已经转移的值。根据编译器的建议，我们可以使用几种方法来修复：

1.  对于 `Foobar` 类型，我们实现 `Copy` trait.
2.  对于 `uses_foobar` 函数，我们使用 `引用传参` 的方式 `借用` `Foobar(1)` 的所有权，如同在 `Drop` trait 里面的 `drop` 函数的第一个参数 `self` 那样。


## <span class="section-num">3</span> 同时引用 {#同时引用}

不象所有权属主，一个值可以同时被多次以 `引用` 的方式使用。如下代码段:

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
    let x: Foobar = Foobar(1);
    let y: &Foobar = &x;
    println!("Before uses_foobar");
    uses_foobar(&x);
    uses_foobar(y);
    println!("After uses_foobar");
}
```

在这里， `Foobar(1)` 两次被以引用的方式使用，一次是作为 `引用参数` 直接传递给
`uses_foobar`, 另外一次是被变量 `y` 以应用的方式使用，并以参数的方式传递给
`uses_foobar`. 在这段代码中，局部变量 `y` 的类型是显示声明的，而不是使用的 =类型
= 推断的方式。代码输出如下：

```text
Before uses_foobar
I consumed a Foobar: Foobar(1)
I consumed a Foobar: Foobar(1)
After uses_foobar
Dropping a Foobar: Foobar(1)
```

代码可以正常运行的原因在于。

1.  多次的 **只读** 引用不会引发数据竟态。
2.  值本身的生命周期要比引用的生命周期长，也就是说，变量 `x` 要比变量 `y` 的生命周期长。

`std::mem::drop` 函数可以主动触发值的失效操作。使用此函数来结束变量 `x` 的值的生命周期。

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
    let x: Foobar = Foobar(1);
    let y: &Foobar = &x;
    println!("Before uses_foobar");
    uses_foobar(&x);
    std::mem::drop(x);
    uses_foobar(y);
    println!("After uses_foobar");
}
```

编译器检查出借用的生命周期超出了其所有权属主的生命周期。

```text
error[E0505]: cannot move out of `x` because it is borrowed
  --> l17.rs:19:20
   |
16 |     let y: &Foobar = &x;
   |                      -- borrow of `x` occurs here
...
19 |     std::mem::drop(x);
   |                    ^ move out of `x` occurs here
20 |     uses_foobar(y);
   |                 - borrow later used here

error: aborting due to previous error

For more information about this error, try `rustc --explain E0505`
```


## <span class="section-num">4</span> 可变引用 {#可变引用}

当然，我们也可以以 `可变引用` 的方式来使用某个值，为了避免数据出现竟态，Rust 不允许同时出现多个 `可变引用` 或者在被可变引用的情况下以其他方式（包括 `只读引用`
）访问。

```rust
fn main() {
    let x: Foobar = Foobar(1);
    let y: &mut Foobar = &mut x;
    println!("Before uses_foobar");
    uses_foobar(&x); // 编译报错
    std::mem::drop(x);
    uses_foobar(y);
    println!("After uses_foobar");
}
```

```text
error[E0596]: cannot borrow `x` as mutable, as it is not declared as mutable
  --> l17.rs:16:26
   |
15 |     let x: Foobar = Foobar(1);
   |         - help: consider changing this to be mutable: `mut x`
16 |     let y: &mut Foobar = &mut x;
   |                          ^^^^^^ cannot borrow as mutable

error[E0502]: cannot borrow `x` as immutable because it is also borrowed as mutable
  --> l17.rs:18:17
   |
16 |     let y: &mut Foobar = &mut x;
   |                          ------ mutable borrow occurs here
17 |     println!("Before uses_foobar");
18 |     uses_foobar(&x); // 编译报错
   |                 ^^ immutable borrow occurs here
19 |     std::mem::drop(x);
20 |     uses_foobar(y);
   |                 - mutable borrow later used here

error[E0505]: cannot move out of `x` because it is borrowed
  --> l17.rs:19:20
   |
16 |     let y: &mut Foobar = &mut x;
   |                          ------ borrow of `x` occurs here
...
19 |     std::mem::drop(x);
   |                    ^ move out of `x` occurs here
20 |     uses_foobar(y);
   |                 - borrow later used here

error: aborting due to 3 previous errors

Some errors have detailed explanations: E0502, E0505, E0596.
For more information about an error, try `rustc --explain E0502`
```
