+++
title = "陪老 K 学 Rust (七)"
author = ["Evilee"]
lastmod = 2020-01-06T18:54:02+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
+++

结构体和枚举的解构和模式匹配。
<!--more-->

Rust 的结构体和枚举有一些新的特性，主要涉及到关联值、解构，模式匹配和解构。使用结构体是 Rust 中定义新数据类型的唯一方式，结构体的定义方式有两种：

1.  元组形式
2.  记录形式

除了可以访问内部字段以外，两种结构体都支持解构其字段（如果有的话）。


## <span class="section-num">1</span> 基本定义和使用形式 {#基本定义和使用形式}

元组和记录形式的基本定义和基本访问形式见如下代码段。

```rust
#[derive(Debug)]
struct NoFieldTuple;

#[derive(Debug)]
struct OneFieldTuple(i32);

#[derive(Debug)]
struct TwoFieldTuple(i32, u32);

#[derive(Debug)]
struct OneFieldRecord {
    index: u32,
}

#[derive(Debug)]
struct TwoFieldRecord {
    index: u32,
    value: i32,
}

fn main() {
    let no_field_tuple = NoFieldTuple;
    println!("{:?}", no_field_tuple);
    let one_field_tuple = OneFieldTuple(1);
    println!("{:?}", one_field_tuple);
    let mut two_field_tuple = TwoFieldTuple(2, 3);
    println!("{:?}", two_field_tuple);
    two_field_tuple.0 = 4;
    two_field_tuple.1 = 5;
    println!("{:?}", two_field_tuple);

    let one_field_record = OneFieldRecord{index: 0};
    println!("{:?}", one_field_record);
    let mut two_field_record = TwoFieldRecord{index: 1, value: 3,};
    println!("{:?}", two_field_record);
    two_field_record.value = 4;
    println!("{:?}", two_field_record);
}
```

运行输出：

```text
NoFieldTuple
OneFieldTuple(1)
TwoFieldTuple(2, 3)
TwoFieldTuple(4, 5)
OneFieldRecord { index: 0 }
TwoFieldRecord { index: 1, value: 3 }
TwoFieldRecord { index: 1, value: 4 }
```

元组和记录的区别：

1.  元组形式可以用空元组来定义结构，而记录形式不可以。
2.  元组使用索引来访问字段，记录使用标签来访问字段。


## <span class="section-num">2</span> 解构 {#解构}

解构 (destruct) 是指把值从某些解构中提取出来，并且绑定到新的变量上。为了更好得理解解构，我们写一个辅助性的函数 `print_type_of` 来打印变量的类型，并且使用最简单的 `元组` 来进行演示.

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}
```


### <span class="section-num">2.1</span> 解构的基本形式 {#解构的基本形式}

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


### <span class="section-num">2.2</span> 解构出可变绑定 {#解构出可变绑定}

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


### <span class="section-num">2.3</span> 解构出引用 {#解构出引用}

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

> 1.  因为如果使用 `*x` 的形式在解构出一个 `可变引用` 的情况下，其语法会变成： `mut
>           *x` 或者 `*mut x`.
> 2.  从形式的一致性来说： `let p = &mut x;` 这种绑定也符合 `解构` 的一般形式。
> 3.  `*` 实际上是一个 `deref` 的运算符，如果用 `*` 表示 `ref` 解构的话，是有语法歧义的。

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


### <span class="section-num">2.4</span> 生命周期和解构 {#生命周期和解构}


## <span class="section-num">3</span> 结构体的形式 {#结构体的形式}

元组形式

```rust
struct NoField;
struct OneField(i32);
struct TowField(u32, i32);
```

使用方式: 索引方式解构方式: 可变解构，引用解构，可变引用解构. 多余的解构，不足的解构.
匹配方式: 可变匹配，引用匹配，可变引用匹配。

记录形式

```rust
struct NoField {}
struct OneField {
    age: i32,
}
struct TowField {
    age: i32,
    score: i32,
}
```


## <span class="section-num">4</span> 枚举的形式 {#枚举的形式}

经典形式

```rust
enum Direction {
    North,
    East,
    Sourth,
    West,
}
```

元组形式关联值

```rust
enum NonamedShape {
    Square(u32),
    Rectangle(u32, u32),
    Circle(u32),
}
```

记录形式关联值

```rust
enum NamedShape {
    Square {
        width: u32,
    },
    Rectangel {
        width: u32,
        height: u32,
    },
    Circle {
        radio: u32,
    },
}
```

经典形式可以当作是元组形式的特殊形式，毕竟 `struct NoFieldTuple;` 是`struct NoField ();` 的简写。

混合形式

```rust
enum HybridShape {
    Dot,
    Square(u32),
    Rectangle {
        width: u32,
        height: u32
    },
    Circle(u32),
}
```


## <span class="section-num">5</span> 模式解构 {#模式解构}

元组的解构记录的解构可变解构引用解构


## <span class="section-num">6</span> 模式匹配解构 {#模式匹配解构}
