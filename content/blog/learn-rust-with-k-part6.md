+++
title = "陪老 K 学 Rust (六)"
author = ["Evilee"]
date = 2010-01-02
lastmod = 2020-01-08T16:17:05+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1003
+++

克隆和拷贝来了。
<!--more-->


## <span class="section-num">1</span> 从引用的所有权居然没有被转移开始 {#从引用的所有权居然没有被转移开始}

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
    let x = Foobar(0);
    let y = &x;
    uses_foobar(y);
    uses_foobar(y);
}
```

运行输出：

```text
I consumed a Foobar: Foobar(0)
I consumed a Foobar: Foobar(0)
Dropping a Foobar: Foobar(0)
```

可以连续两次调用 `uses_foobar`, 什么情况？不是说所有权会转移的吗？那变量 `y` 绑定值（ `x` 的地址）的所有权不会被转移吗？

做一个简单的实验：

```rust
fn uses_i32(i: i32) {
    println!("I consumed an i32: {}", i);
}

fn main() {
    let x = 1;
    uses_i32(x);
    uses_i32(x);
}
```

运行输出：

```text
I consumed an i32: 1
I consumed an i32: 1
```

在上述代码段中，难道 `x` 的所有权没有被转移？

再实验一下：

```rust
#[derive(Debug)]
struct Foobar(i32);

fn uses_foobar(foobar: Foobar) {
    println!("I consumed a Foobar: {:?}", foobar);
}

fn main() {
    let x = Foobar(1);

    uses_foobar(x);
    uses_foobar(x);
}
```

编译报错：

```text
error[E0382]: use of moved value: `x`
  --> l25.rs:12:17
   |
9  |     let x = Foobar(1);
   |         - move occurs because `x` has type `Foobar`, which does not implement the `Copy` trait
10 |
11 |     uses_foobar(x);
   |                 - value moved here
12 |     uses_foobar(x);
   |                 ^ value used here after move

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`.
```

仔细阅读错误输出， ``move occurs because `x` has type `Foobar`, which does not
implement the `Copy` trait``, 看来是 `Foobar` 没有实现 `Copy` trait. 那基本可以确定前面两段代码中的 `y` 可能已经实现了 `Copy` trait, 所以在编译期间才没有所有权转移的报错信息。

Rust 中有一个特定的 trait: `Copy`, 这个 trait 可以标识某些数据类型可以按值传递，通常，基于效率方面的考虑，按值传递这种方式适合的数据类型在被复制的时候应该不能浪费很多的资源。在上例中， `i32` 和 `地址` 这两种数据类型因为实现了 `Copy`
trait, 在作为参数传递给函数时，实际上是拷贝了一个新的值给函数，函数所拥有的所有权是被复制出来的新值的所有权。

对于 `Foobar` 数据结构，如果需要按值传递的话，可以显式使用 `Clone` trait.

```rust
#[derive(Debug, Clone)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(self: &mut Self) {
        println!("Dropping: {:?}", self);
    }
}

fn uses_foobar(foobar: Foobar) {
    println!("I consumed a Foobar: {:?}", foobar);
}

fn main() {
    let x = Foobar(1);

    uses_foobar(x.clone());
    uses_foobar(x);
}
```

运行输出:

```text
I consumed a Foobar: Foobar(1)
Dropping: Foobar(1)
I consumed a Foobar: Foobar(1)
Dropping: Foobar(1)
```

`Clone` trait 和 `Debug` trait 一样，都是可以自动继承的。对于复合数据类型来说，自动继承的条件是：组成复合数据类型的子数据类型必须满足 `Clone` trait.

对于 Rust 来说， `Copy` trait 必须实现 `Clone` trait. 这里并不是说 `Copy` trait
需要使用 `Clone` trait 的 `clone` 函数去复制对象，而是说可以 `Copy` 的对象是可以被 `Clone` 的，实际上 `Copy` trait 是编译器在内存中按位复制一个新的值。 `Copy`
trait 只是一个标志，内部没有需要实现的方法，这个标志存在意义在于告知编译器：我这个数据类型是可以按值传递的，请在需要的时候 `按位复制` 一个新的值。既然有了
`Clone`, 为何不用 `Clone` 替代呢？因为在某些情况下，数据类型虽然实现了 `Clone`,
但是 `Clone` 一个新值的代价非常大。

```rust
#[derive(Debug, Clone, Copy)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(self: &mut Self) {
        println!("Dropping: {:?}", self);
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

编译输出：

```text
error[E0184]: the trait `Copy` may not be implemented for this type; the type has a destructor
 --> l26.rs:1:24
  |
1 | #[derive(Debug, Clone, Copy)]
  |                        ^^^^ Copy not allowed on types with destructors

error: aborting due to previous error

For more information about this error, try `rustc --explain E0184`
```

编译器报错： `Copy` trait 不能出现在具有 `Drop` trait 的数据类型上。 仔细想想也有道理， `Drop` 是数据在销毁时的回收动作，如果一个数据类型在销毁时会有资源需要回收，一方面说明它被复制时的消耗可能比较大，另一方面说明这个数据类型在使用时必然要对所释放的资源进行初始化，编译器默认的 `Copy` 动作(`按位复制`)并不能初始化这些资源，强行使用是不正确的。去掉 `Drop` trait 即可。

```rust
#[derive(Debug, Clone, Copy)]
struct Foobar(i32);

fn uses_foobar(foobar: Foobar) {
    println!("I consumed a Foobar: {:?}", foobar);
}

fn main() {
    let x = Foobar(1);

    uses_foobar(x);
    uses_foobar(x);
}
```

结论：只有可以 `按位复制` 的数据类型才能实现 `Copy` trait. 能否按位复制，要看写代码的人自己判断。通常可以按照以下几个规则进行：

1.  基本数据类型，整型，浮点型等。
2.  内部实现类型都可以 `Copy` 的复合数据类型。
3.  不需要初始化资源的类型，比如在堆上申请内存空间，打开文件描述符或者 socket 等。


## <span class="section-num">2</span> 引用和指针 {#引用和指针}

回过头来，我们看看引用、借用的问题。变量 `y` 是 `&mut Foobar` 类型，这是一个引用，也就是一个指针。这个值指向其所引用的值的地址，所以这个地址明显是可以 `按位复制`
的，其目标值如果需要初始化资源或者释放资源，则由目标值的属主负责处理，故而这里才称之为 `借用`.
