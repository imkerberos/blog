+++
title = "陪老 K 学 Rust (五)"
author = ["Evilee"]
date = 2019-12-26
lastmod = 2020-01-02T23:56:49+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1003
+++

可变与不变
<!--more-->


## <span class="section-num">1</span> 赋值与绑定 {#赋值与绑定}

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

在很多语言中（其实也包括 Rust）， `let` 的含义并不是声明一个变量，而是进行一个值 `绑定`
操作，也就是把一个值和一个名称关联起来，从这一点上来说 `绑定` 比 `赋值` 更形象。


## <span class="section-num">2</span> 可变与不变 {#可变与不变}

还记得 C 关于 `const` 关键字的 `常量指针` 与 `指针常量` 的问题吗？`const char * const p = &str;`, 我们就以分析 `const` 的方法来分析 `mut` 关键字:

1.  `let mut x: Foobar = Foobar(0);` 这种形式中， `mut` 修饰的是绑定关系还是值本身？ `mut` 只修饰变量，即修饰变量和值的绑定关系，不修饰值本身。
2.  `let mut y: &mut Foobar = &mut x;` 这种引用形式中，第一个 `mut` 限定的是绑定关系，也就是 `y` 可以是 `x` 的引用绑定，也可以是其他值的引用绑定。 第二个 `mut` 限定的是被应用的值本身，即值本身的内容是否可以被此引用修改。第三个 `mut` 的作用等同于第二个 `mut`, 在使用类型推断的情况下，这一点就更为明显：`let mut y = &mut x;`.


### <span class="section-num">2.1</span> 修改值 {#修改值}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    println!("{:?}", x);
    x.0 = 10;
    println!("{:?}", x);
}
```

编译输出:

```text
  --> l20.rs:13:5
   |
11 |     let x = Foobar(0);
   |         - help: consider changing this to be mutable: `mut x`
12 |     println!("{:?}", x);
13 |     x.0 = 10;
   |     ^^^^^^^^ cannot assign

error: aborting due to previous error
```

结论：非 `mut` 绑定不能修改值的内容。

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let mut x = Foobar(0);
    println!("{:?}", x);
    x.0 = 10;
    println!("{:?}", x);
}
```

编译运行输出：

```text
Foobar(0)
Foobar(10)
Dropping a Foobar: Foobar(10
```

结论： `mut` 绑定可以修改值的内容。


### <span class="section-num">2.2</span> 修改绑定关系 {#修改绑定关系}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    let y = Foobar(1);
    println!("{:?}", x);
    x = y;
    println!("{:?}", x);
}
```

编译输出：

```text
error[E0384]: cannot assign twice to immutable variable `x`
  --> l20.rs:14:5
   |
11 |     let x = Foobar(0);
   |         -
   |         |
   |         first assignment to `x`
   |         help: make this binding mutable: `mut x`
...
14 |     x = y;
   |     ^ cannot assign twice to immutable variable

error: aborting due to previous error

For more information about this error, try `rustc --explain E0384`
```

结论：非 `mut` 绑定不能修改绑定关系。

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let mut x = Foobar(0);
    println!("{:?}", x);
    let y = Foobar(1);
    println!("{:?}", y);
    x = y;
    println!("{:?}", x);
    x.0 = 10;
    println!("{:?}", x);
}
```

编译运行输出：

```text
Foobar(0)
Foobar(1)
Dropping a Foobar: Foobar(0)
Foobar(1)
Foobar(10)
Dropping a Foobar: Foobar(10)
```

结论： `mut` 绑定可以修改绑定关系，并且可以修改值的内容。这个修改与 `y` 原来是否是 `mut` 无关。


### <span class="section-num">2.3</span> 重置绑定 {#重置绑定}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    println!("{:?}", x);
    let x = Foobar(1);
    println!("{:?}", x);
}
```

编译运行输出：

```text
Foobar(0)
Foobar(1)
Dropping a Foobar: Foobar(1)
Dropping a Foobar: Foobar(0
```

结论： 无论是否是 `mut` 绑定，都可以重新绑定。


### <span class="section-num">2.4</span> 可变性修改 {#可变性修改}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    println!("{:?}", x);
    let mut y = x;
    println!("{:?}", y);
    y.0 = 1;
    println!("{:?}", y);
}
```

运行输出：

```text
Foobar(0)
Foobar(0)
Foobar(1)
Dropping a Foobar: Foobar(1)
```

根据以上代码，下面的 mutable move 也就很好理解了。

```rust
#[derive(Debug)]
struct Foobar(i32);

fn main() {
    let x = Foobar(1);
    foo(x);
}

fn foo(mut x: Foobar) {

    x.0 = 2; // changes the 0th value inside the product

    println!("{:?}", x);
}
```

运行输出：

```text
Foobar(2)
```


### <span class="section-num">2.5</span> 不变引用不变值绑定 {#不变引用不变值绑定}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    let y = &x; // let y: &Foobar = &x;
    println!("{:?}", x);
    println!("{:?}", y);
}
```

编译运行输出：

```text
Foobar(0)
Foobar(0)
Dropping a Foobar: Foobar(0)
```

> `println!("{:?}", x)` 难道不会接管 `x` 的所有权吗？注意：
> println! 是宏而不是函数，你焉不知这个宏看上去是用的 `x`, 在背后用的是 `&x` 呢？


### <span class="section-num">2.6</span> 不变引用可变值绑定 {#不变引用可变值绑定}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let mut x = Foobar(0);
    println!("{:?}", x);
    let y = &mut x; // let y: &mut Foobar = &mut x;
    println!("{:?}", y);
    y.0 = 10;
    println!("{:?}", y);
}
```

编译运行输出:

```text
Foobar(0)
Foobar(0)
Foobar(10)
Dropping a Foobar: Foobar(10)
```

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let mut x = Foobar(0);
    println!("{:?}", x);
    let y = &mut x; // let y: &mut Foobar = &mut x;
    println!("{:?}", y);
    let mut z = Foobar(1);
    y = &mut z;
    println!("{:?}", y);
}
```

编译报错：

```text
error[E0384]: cannot assign twice to immutable variable `y`
  --> l20.rs:16:5
   |
13 |     let y = &mut x; // let y: &mut Foobar = &mut x;
   |         -
   |         |
   |         first assignment to `y`
   |         help: make this binding mutable: `mut y`
...
16 |     y = &mut z;
   |     ^^^^^^^^^^ cannot assign twice to immutable variable

error: aborting due to previous error

For more information about this error, try `rustc --explain E0384`.
```

结论： `y` 是不变引用，其引用的值被 `mut` 修饰为可变。即： `y` 的绑定关系不能修改，但是 `y` 指向的值可以被修改。


### <span class="section-num">2.7</span> 可变引用不变值绑定 {#可变引用不变值绑定}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    println!("{:?}", x);
    let mut y = &x; // let mut y: &Foobar = &mut x;
    println!("{:?}", y);
    let z = Foobar(1);
    y = &z;
    println!("{:?}", y);
}
```

运行输出：

```text
Foobar(0)
Foobar(0)
Foobar(1)
Dropping a Foobar: Foobar(1)
Dropping a Foobar: Foobar(0)
```

结论：可变引用可以改变绑定关系， `y` 并不特殊，也遵循可变绑定和不变绑定。


### <span class="section-num">2.8</span> 可变引用可变值绑定 {#可变引用可变值绑定}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let mut x = Foobar(0);
    println!("{:?}", x);
    let mut y = &mut x; // let mut y: &Foobar = &mut x;
    println!("{:?}", y);
    let mut z = Foobar(1);
    y = &mut z;
    println!("{:?}", y);
}
```

运行输出：

```text
Foobar(0)
Foobar(0)
Foobar(1)
Dropping a Foobar: Foobar(1)
Dropping a Foobar: Foobar(0)
```

结论：可变引用可以改变绑定关系， `y` 并不特殊，也遵循可变绑定和不变绑定。


### <span class="section-num">2.9</span> 不变引用的共享性 {#不变引用的共享性}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    let y = &x; // let y: &Foobar = &x;
    let z = &x; // let z: &Foobar = &x;
    println!("{:?}", x);
    println!("{:?}", y);
    println!("{:?}", z);
}
```

运行输出：

```text
Foobar(0)
Foobar(0)
Foobar(0)
Dropping a Foobar: Foobar(0)
```

结论： `x`, `y`, `z` 随便用。


### <span class="section-num">2.10</span> 可变引用的排他性 {#可变引用的排他性}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let mut x = Foobar(0);
    let y = &mut x; // let y: &mut Foobar = &mut x;
    let z = &x; // let z: &Foobar = &x;
    println!("{:?}", x);
    println!("{:?}", y);
    println!("{:?}", z);
}
```

编译报错：

```text
error[E0502]: cannot borrow `x` as immutable because it is also borrowed as mutable
  --> l20.rs:13:13
   |
12 |     let y = &mut x; // let y: &Foobar = &x;
   |             ------ mutable borrow occurs here
13 |     let z = &x; // let z: &Foobar = &x;
   |             ^^ immutable borrow occurs here
14 |     println!("{:?}", x);
15 |     println!("{:?}", y);
   |                      - mutable borrow later used here

error[E0502]: cannot borrow `x` as immutable because it is also borrowed as mutable
  --> l20.rs:14:22
   |
12 |     let y = &mut x; // let y: &Foobar = &x;
   |             ------ mutable borrow occurs here
13 |     let z = &x; // let z: &Foobar = &x;
14 |     println!("{:?}", x);
   |                      ^ immutable borrow occurs here
15 |     println!("{:?}", y);
   |                      - mutable borrow later used here

error: aborting due to 2 previous errors

For more information about this error, try `rustc --explain E0502`.
```

结论：

1.  `println!` 宏的确是转换成了引用。
2.  在 `y` 可变借用了 `x`, 以后， `println!` 的不变引用被拒绝。


### <span class="section-num">2.11</span> 强制不变引用和强制可变引用 {#强制不变引用和强制可变引用}

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let mut x = Foobar(0);
    let y = &x; // let y: &Foobar = &x;
    let z = &x; // let z: &Foobar = &x;
    println!("{:?}", x);
    println!("{:?}", y);
    println!("{:?}", z);
}
```

编译运行输出:

```text
warning: variable does not need to be mutable
  --> l20.rs:11:9
   |
11 |     let mut x = Foobar(0);
   |         ----^
   |         |
   |         help: remove this `mut`
   |
   = note: `#[warn(unused_mut)]` on by default

Foobar(0)
Foobar(0)
Foobar(0)
Dropping a Foobar: Foobar(0)
```

除了一个 `x` 的未使用的 `mut` 限定意外，运行没毛病，也就是： **可以以不变的方式引用可变绑定**. 那我们反过来，以可变的方式应用不变绑定呢？

```rust
#[derive(Debug)]
struct Foobar(i32);

impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}

fn main() {
    let x = Foobar(0);
    let y = &mut x; // let y: &mut Foobar = &mut x;
    println!("{:?}", y);
}
```

编译报错：

```text
error[E0596]: cannot borrow `x` as mutable, as it is not declared as mutable
  --> l20.rs:12:13
   |
11 |     let x = Foobar(0);
   |         - help: consider changing this to be mutable: `mut x`
12 |     let y = &mut x; // let y: &mut Foobar = &mut x;
   |             ^^^^^^ cannot borrow as mutable

error: aborting due to previous error

For more information about this error, try `rustc --explain E0596`.
```

结论： **不能** 把不变绑定强制转换成可变引用。

> 扩展一下思路，在函数参数传递的场景下， `mut` 的原则又是什么呢？
>
> 1.  `fn uses_foobar(foobar: &Foobar)`
> 2.  `fn uses_foobar(mut foobar: &Foobar)`
> 3.  `fn uses_foobar(foobar: &mut Foobar)`
> 4.  `fn uses_foobar(mut foobar: &mut Foobar)`


## <span class="section-num">3</span> 再论可变与不变 {#再论可变与不变}

由以上的栗子可知： `Foobar` 自身完全没有权利决定自己的内容是可变的还是不变的，其内容能否可变，取决于在其被绑定时的绑定方式。象 `Foopbar` 这种元组还不是特别明显，以 `struct` 作为参考：

```rust
struct Greet {
    age: i32,
    score: i32
}

fn main() {
    let f1 = Greet{age: 18, score: 60};
    let mut f2 = Greeg{age: 20, score: 80};
}
```

<div class="src-block-caption">
  <span class="src-block-number">&#20195;&#30721; 1</span>:
  Hello
</div>

在以上代码中，实际上 `Greet` 的字段都是默认可变的。:( 这听上去怎么和 Rust 的值默认不变相矛盾？

在其他一些语言中， `let` 和 `var` 来分别代表不变绑定和可变绑定（如：swift），并且可变和不可变的作用是单一的，只用来限定绑定关系是否可变。值本身的内容由值的类型来决定，这么说有些抽象，还是拿 `Greet` 的栗子来说：

```swift
struct Greet {
    let age: Int32,
    var score: Int32,
}

func main() {
    let f1 = Greet(age: 10, score: 60)
    f1.score = 80 // Ok, 因为 score 是 var, 可变的.
    f1 = Greet(age: 20, score: 80) // Nope, 因为 f1 是 let, 不变的，不能改变绑定关系。

    var f2 = Greet(age: 10, score: 60)
    f2.age = 10 // Nope: 虽然 f2 是可变的，但是 age 在 struct 内部是不变的。
    f2 = Greet(age: 20, score: 80) // Ok, f2 可以重复绑定。
}
```

相对比来说， swift 的模型貌似更符合一个正常的心智模型，而 Rust 确是怪怪的，私自以为 rust 对于 `mut` 的处理非常不合理，一个数据类型是否可变居然不取决于其自身的设计。在设计之初，没有不可变的选择。:(, 相反在这一点上 `swift` 更加合理。


## <span class="section-num">4</span> 胡乱说的模型 {#胡乱说的模型}

如果 Rust 代码的语法是这样的，可能一致性更好一些：

```rust
fn main() {
    let x: mut Foobar = mut Foobar(0);
    let mut y: mut Foobar = mut Foobar(1);
    let mut z = mut Foobar(3);
    let o: Foobar = Foobar(4);
}
```

这样，第一个 `mut` 修饰绑定关系，第二个 `mut` 修饰内容就和 `借用/引用` 保持一致了。：） 可惜现实不是这样的，我们姑且把 `let x: mut Foobar = mut Foobar(0);` 这种看成是默认的语法糖吧。
