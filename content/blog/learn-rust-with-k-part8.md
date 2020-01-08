+++
title = "陪老 K 学 Rust (八)"
author = ["Evilee"]
lastmod = 2020-01-08T09:57:29+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
+++

自定义数据类型：结构体。
<!--more-->

Rust 的结构体和枚举有一些新的特性，主要涉及到关联值、解构，模式匹配和解构。使用结构体是 Rust 中定义新数据类型的唯一方式，结构体的定义方式有两种：

1.  元组形式
2.  记录形式

除了可以访问内部字段以外，两种结构体都支持解构其字段（如果有的话）。


## <span class="section-num">1</span> 结构体的基本定义和使用形式 {#结构体的基本定义和使用形式}

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

<div class="src-block-caption">
  <span class="src-block-number">&#20195;&#30721; 1</span>:
  struct 的基本形式
</div>

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


## <span class="section-num">2</span> 解构结构 {#解构结构}

理解了元组的解构以后，元组结构的解构就比较容易理解了，一一对应即可。

```rust

```


## <span class="section-num">3</span> 枚举的形式 {#枚举的形式}

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


## <span class="section-num">4</span> 模式匹配与解构 {#模式匹配与解构}
