+++
title = "陪老 K 学 Rust (八)"
author = ["Evilee"]
lastmod = 2020-01-16T15:47:16+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1007
+++

自定义数据类型：结构体
<!--more-->
Rust 的结构体和枚举有一些新的特性，主要涉及到关联值、解构，模式匹配和解构。使用结构体是 Rust 中定义新数据类型的唯一方式，结构体的定义方式有两种：

1.  元组形式
2.  记录形式

除了可以访问内部字段以外，两种结构体都支持解构其字段（如果有的话）。


## <span class="section-num">1</span> 基本形式 {#基本形式}

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


## <span class="section-num">2</span> 解构 {#解构}

理解了元组的解构以后，元组结构的解构就比较容易理解了，一一对应即可。

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

struct Foobar(i32, u32);
struct Greet {
    f1: i32,
    f2: u32,
}

fn main() {
    let foobar0 = Foobar(1, 2);
    let Foobar(x, y) = foobar0;
    print_type_of(&x);
    print_type_of(&y);

    let greet0 = Greet{f1: 1, f2: 2};
    let Greet{f1: v1, f2: v2} = greet0;
    print_type_of(&v1);
    print_type_of(&v2);

    let greet1 = Greet{f1: 3, f2: 4};
    let Greet{f1: f1, f2: f2} = greet1;
    print_type_of(&f1);
    print_type_of(&f2);

    let greet1 = Greet{f1: 5, f2: 6};
    let Greet{f1, f2} = greet1;
    print_type_of(&f1);
    print_type_of(&f2);
}
```

编译输出和运行输出：

```text
warning: the `f1:` in this pattern is redundant
  --> r35.rs:23:15
   |
23 |     let Greet{f1: f1, f2: f2} = greet1;
   |               ---^^^
   |               |
   |               help: remove this
   |
   = note: `#[warn(non_shorthand_field_patterns)]` on by default

warning: the `f2:` in this pattern is redundant
  --> r35.rs:23:23
   |
23 |     let Greet{f1: f1, f2: f2} = greet1;
   |                       ---^^^
   |                       |
   |                       help: remove this

i32
u32
i32
u32
i32
u32
i32
u32
```

元组形式的解构和记录形式的解构形式上是类似的。

> 对于记录形式的结构，在字段名称和解构变量名称一致的情况下，`let Greet{f1: f1, f2: f2} = greet1;` 这种形式简写为：
> `let Greet{f1, f2} = greet1;`

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

struct Foobar(i32, u32);
struct Greet {
    f1: i32,
    f2: u32,
}

fn main() {
    let mut foobar = Foobar(1, 2);
    let Foobar(mut e1, ref mut e2) = foobar;
    println!("e1: {}, e2: {}", e1, e2);
    print_type_of(&e1);
    print_type_of(&e2);
    e1 = 0;
    println!("new e1: {}", e1);

    let greet = Greet{f1: 3, f2: 4};
    let Greet{ref f1, mut f2} = greet;
    println!("f1: {}, f2: {}", f1, f2);
    f2 = 0;
    println!("new f2: {}", f2);
}
```

运行输出:

```text
e1: 1, e2: 2
i32
&mut u32
new e1: 0
f1: 3, f2: 4
new f2: 0
```

> 1.  元组结构使用 `Type()` 的方式解构，与构造时的语法对应。
> 2.  记录结构使用 `Type{}` 的方式解构，与构造时的语法对应。


## <span class="section-num">3</span> 模式匹配与解构 {#模式匹配与解构}

> 1.  元组结构使用 `()` 进行匹配和解构，与构造时的语法对应。
> 2.  记录结构使用 `{}` 进行匹配和解构，与构造时的语法对应。

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

struct Foobar(i32, u32);
struct Greet {
    f1: i32,
    f2: u32,
}

fn main() {
    let foobar = Foobar(0, 1);

    match foobar {
        Foobar(ref x0, y@0..=1) => {
            println!("match Foobar(ref x0, y@0..1) x0 is: {}, y1 is: {}", x0, y);
        },
        Foobar(_, y1) if y1 == 2 => {
            println!("match Foobar(_, y1) if y1 == 2, y1 is: {}", y1);
            print_type_of(&y1);
        },
        _ => {
            println!("default match");
        },
    }

    let greet = Greet{f1: 2, f2: 4};
    match greet {
        Greet{f1: v1, f2: v2} => {
            println!("match Greet{{f1: v1, f2: v2}}, v1: {}, v2: {}", v1, v2);
            print_type_of(&v1);
            print_type_of(&v2);
        },
    }

    let greet = Greet{f1: 5, f2: 6};
    match greet {
        Greet{f1, f2} => {
            println!("match Greet{{f1, f2}}, f1: {}, f2: {}", f1, f2);
            print_type_of(&f1);
            print_type_of(&f2);
        },
    }

    let greet = Greet{f1: 7, f2: 8};
    match greet {
        Greet{f1, f2} if f1 < 0 => {
            println!("match Greet{{f1, f2}}, f1: {}, f2: {}", f1, f2);
            print_type_of(&f1);
            print_type_of(&f2);
        },
        _ => {
            println!("match _");
        }
    }

    let greet = Greet{f1: 7, f2: 8};
    match greet {
        Greet{f1: v0@ 0..= 10, f2: v1 @ 0 ..= 10} => {
            println!("match Greet{{f1, f2}}, f1: {}, f2: {}", v0, v1);
            print_type_of(&v0);
            print_type_of(&v1);
        },
        _ => {
            println!("match _");
        }
    }
}
```

运行输出:

```text
match Foobar(ref x0, y@0..1) x0 is: 0, y1 is: 1
match Greet{f1: v1, f2: v2}, v1: 2, v2: 4
i32
u32
match Greet{f1, f2}, f1: 5, f2: 6
i32
u32
match _
match Greet{f1, f2}, f1: 7, f2: 8
i32
u32
```


## <span class="section-num">4</span> 模式匹配与绑定 {#模式匹配与绑定}

在使用 `@` 绑定时，记录结构必须重新绑定新的变量名称。

```rust
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

struct Greet {
    f1: i32,
    f2: u32,
}

fn main() {
    let greet = Greet{f1: 1, f2: 2};
    match greet {
        Greet{f1@0..=10, f2@0..=10} => {
            println!("match Greet{{f1, f2}}, f1: {}, f2: {}", f1, f2);
            print_type_of(&f1);
            print_type_of(&f2);
        },
        _ => {

        },
    }
}
```

编译错误:

```text
error: expected `,`
  --> r42.rs:13:15
   |
13 |         Greet{f1@0..=10, f2@0..=10} => {
   |               ^^

error[E0425]: cannot find value `f1` in this scope
  --> r42.rs:14:63
   |
14 |             println!("match Greet{{f1, f2}}, f1: {}, f2: {}", f1, f2);
   |                                                               ^^ not found in this scope

error[E0425]: cannot find value `f2` in this scope
  --> r42.rs:14:67
   |
14 |             println!("match Greet{{f1, f2}}, f1: {}, f2: {}", f1, f2);
   |                                                                   ^^ not found in this scope

error[E0425]: cannot find value `f1` in this scope
  --> r42.rs:15:28
   |
15 |             print_type_of(&f1);
   |                            ^^ not found in this scope

error[E0425]: cannot find value `f2` in this scope
  --> r42.rs:16:28
   |
16 |             print_type_of(&f2);
   |                            ^^ not found in this scope

error: aborting due to 5 previous errors

For more information about this error, try `rustc --explain E0425`.
```
