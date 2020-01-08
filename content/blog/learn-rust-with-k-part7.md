+++
title = "陪老 K 学 Rust (七)"
author = ["Evilee"]
date = 2020-01-07
lastmod = 2020-01-08T09:55:12+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1002
+++

元组可以说是 Rust 最简单的自定类型。通过元组来理解 `模式匹配` 和 `解构` -现代语言的时尚特性。
<!--more-->

解构 (destruct) 是指把值从某些解构中提取出来，并且绑定到新的变量上。为了更好得理解解构，我们写一个辅助性的函数 `print_type_of` 来打印变量的类型，并且使用最简单的 `元组` 来进行演示.

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}
```


## <span class="section-num">1</span> 解构的基本形式 {#解构的基本形式}

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let (x, y, z) = (0, 1, 2);
    print_type_of(&x);
    print_type_of(&y);
    print_type_of(&z);
    println!("x:{}, y:{}, z:{}", x, y, z);
}
```

运行输出：

```text
i32
i32
i32
x:0, y:1, z:2
```

不关心的解构字段用 `_` 进行占位，如：

```rust
fn main() {
    let (x, _, z) = (0, 1, 2);
    println!("x:{}, z:{}", x, z);
}
```

运行输出:

```text
x:0, z:2
```

> 解构要诀：
>
> 1.  使用 `let`, `=` 左右两边的类型一致。

由于变量数量不足而引起的类型不匹配：

```rust
fn main() {
    let (x, y) = (0, 1, 2);
    println!("x:{}, y:{}", x, y);
}
```

编译报错：

```text
error[E0308]: mismatched types
 --> r28.rs:2:9
  |
2 |     let (x, y) = (0, 1, 2);
  |         ^^^^^^ expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected type `({integer}, {integer}, {integer})`
             found type `(_, _)`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0308`
```

> 有些编程语言在解构变量数量不足时，最后一个变量会解构所有剩余的元组元素，从而变成一个元组，但是 Rust 不会。


## <span class="section-num">2</span> 解构出可变绑定 {#解构出可变绑定}

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let (x, mut y, z) = (0, 1, 2);
    print_type_of(&x);
    print_type_of(&y);
    print_type_of(&z);
    println!("x:{}, y:{}, z:{}", x, y, z);
    y = 4;
    println!("{:}", y);
}
```

运行输出

```text
i32
i32
i32
x:0, y:1, z:2
4
```


## <span class="section-num">3</span> 解构出引用 {#解构出引用}

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let (x, *y, z) = (0, 1, 2);
    print_type_of(&x);
    print_type_of(&y);
    print_type_of(&z);
    println!("x:{}, y:{}, z:{}", x, y, z);
}
```

我们妄图使用 `*y = i32` 的形式解构出一个 `&i32`, 编译器报错：

```text
error: expected pattern, found `*`
 --> r28.rs:2:13
  |
2 |     let (x, *y, z) = (0, 1, 2);
  |             ^ expected pattern

error: aborting due to previous error
```

???, 原来想解构出 `引用` 的语法形式是 `ref`, 为什么 **不是** `*x` 的形式？

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let (x, ref y, z) = (0, 1, 2);
    print_type_of(&x);
    print_type_of(&y);
    print_type_of(&z);
    println!("x:{}, y:{}, z:{}", x, y, z);
}
```

运行输出:

```text
i32
&i32
i32
x:0, y:1, z:2
```

如果要解构出一个 `ref mut` 呢？

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let (x, ref mut y, z) = (0, 1, 2);
    print_type_of(&x);
    print_type_of(&y);
    print_type_of(&z);
    println!("x:{}, y:{}, z:{}", x, y, z);
}
```

运行输出:

```text
i32
&mut i32
i32
x:0, y:1, z:
```

如果要解构出一个 `mut ref` 呢？

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let (x, mut ref y, z) = (0, 1, 2);
    print_type_of(&x);
    print_type_of(&y);
    print_type_of(&z);
    println!("x:{}, y:{}, z:{}", x, y, z);
}
```

不要太想当然 :(, 编译器报错。

```text
error: the order of `mut` and `ref` is incorrect
 --> r29.rs:6:13
  |
6 |     let (x, mut ref y, z) = (0, 1, 2);
  |             ^^^^^^^ help: try switching the order: `ref mut`

error: aborting due to previous error
```


## <span class="section-num">4</span> 绑定与解构 {#绑定与解构}

从形式的一致性来说： `let p = &mut x;` 这种绑定也符合 `解构` 的一般形式。

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let mut v = 10;
    print_type_of(&v);
    let p = &v;
    print_type_of(&p);
    let ref p = v;
    print_type_of(&p);
    let ref mut p = v;
    print_type_of(&p);
}
```

运行输出:

```text
i32
&i32
&i32
&mut i32
```


## <span class="section-num">5</span> 解构与生命周期 {#解构与生命周期}

假设一个元组由数个元素组成，如果进行解构的话，其所有权是否会被转移？答案是 **会**,
看代码：

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop (&mut self) {
        println!("Dropping {:?}", self);
    }
}

fn main() {
    let x = (Foobar(0), );
    let (foobar,) = x;
    println!("{:?}", foobar);
    println!("{:?}", x);
}
```

编译器报错：x 的所有权已经被转移。

```text
error[E0382]: borrow of moved value: `x`
  --> r33.rs:14:22
   |
12 |     let (foobar,) = x;
   |          ------ value moved here
13 |     println!("{:?}", foobar);
14 |     println!("{:?}", x);
   |                      ^ value borrowed here after partial move
   |
   = note: move occurs because `x.0` has type `Foobar`, which does not implement the `Copy` trait

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`.
```

如果使用引用解构的话，则不会，符合生命周期的心智模型。

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop (&mut self) {
        println!("Dropping {:?}", self);
    }
}

fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let x = (Foobar(0), );
    let (ref foobar,) = x;
    println!("{:?}", foobar);
    println!("{:?}", x);
    print_type_of(&x);
    print_type_of(&foobar);
}
```

```text
Foobar(0)
(Foobar(0),)
(r34::Foobar,)
&r34::Foobar
Dropping Foobar(0)
```


## <span class="section-num">6</span> 模式匹配与解构 {#模式匹配与解构}

除了解构之外，元组还可以在使用 `match` 关键字进行模式匹配的同时进行解构。

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let v = (0, 3);
    match v {
        (x, 1) => {
            print_type_of(&x);
            println!("match (x, 1)");
        },
        (x, 2) => {
            print_type_of(&x);
            println!("match (x, 2)");
        },
        (x, 3) => {
            print_type_of(&x);
            println!("match (x, 3)");
        },
        _ => {
            println!("not match any");
        }
    }
}
```

<div class="src-block-caption">
  <span class="src-block-number">&#20195;&#30721; 1</span>:
  模式匹配和解构
</div>

```text
i32
match (x, 3)
```

> 注意使用 `_` 进行默认匹配来全覆盖匹配的所有分支。

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let v = (0, 3);
    match &v {
        (x, 1) => {
            print_type_of(&x);
            println!("match (x, 1)");
        },
        (x, 2) => {
            print_type_of(&x);
            println!("match (x, 2)");
        },
        (x, 3) => {
            print_type_of(&x);
            println!("match (x, 3)");
        },
        _ => {
            println!("not match any");
        }
    }
}
```

<div class="src-block-caption">
  <span class="src-block-number">&#20195;&#30721; 2</span>:
  模式匹配与解构：引用（一）
</div>

```text
&i32
match (x, 3)
```

使用 `ref` 来引用解构。

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let v = (0, 3);
    match v {
        (ref x, 1) => {
            print_type_of(&x);
            println!("match (x, 1)");
        },
        (ref x, 2) => {
            print_type_of(&x);
            println!("match (x, 2)");
        },
        (ref x, 3) => {
            print_type_of(&x);
            println!("match (x, 3)");
        },
        _ => {
            println!("not match any");
        }
    }
}
```

<div class="src-block-caption">
  <span class="src-block-number">&#20195;&#30721; 3</span>:
  模式匹配与解构：引用（二）
</div>

```text
&i32
match (x, 3)
```

> 注意以上两段代码的不同点。

如果要解构可变引用呢？

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let mut v = (0, 3);
    match v {
        (ref mut x, 1) => {
            print_type_of(&x);
            println!("match (x, 1)");
        },
        (ref mut x, 2) => {
            print_type_of(&x);
            println!("match (x, 2)");
        },
        (ref mut x, 3) => {
            print_type_of(&x);
            println!("match (x, 3)");
        },
        _ => {
            println!("not match any");
        }
    }
}
```

<div class="src-block-caption">
  <span class="src-block-number">&#20195;&#30721; 4</span>:
  模式匹配与解构：引用（三）
</div>

```text
&mut i32
match (x, 3)
```
