+++
title = "陪老 K 一起学 Rust (三)"
author = ["Eviler"]
date = 2019-12-24
lastmod = 2019-12-24T16:41:19+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1002
+++

从简单的栗子开始。

<!--more-->


## <span class="section-num">1</span> 从简单的栗子开始 {#从简单的栗子开始}

```rust
#[derive(Debug)]
struct Foobar(i32);

fn uses_foobar(foobar: Foobar) {
    println!("I consumed a Foobar: {:?}", foobar);
}

fn main() {
    let x = Foobar(1);
    uses_foobar(x);
}
```

`#[derive(Debug)]` 是一个编译器程序，这里让 `Foobar` 继承
`Debug` trait, 以便于后面的 `uses_foobar` 函数中调用 `println!` 使用 `{:?}` 占位符进行插值打印。

`struct Foobar(i32);` 将一个 `元组` 封装成新的数据类型 `Foobar`.

运行程序可以看到打印输出。

```text
I consumed a Foobar: Foobar(1)
```

现在修改一下 `main` 函数，我们仅仅是希望打印两次 `x` 的值，所以调用了两次
`uses_foobar` 。

```rust
fn main() {
    let x = Foobar(1);

    uses_foobar(x);
    uses_foobar(x);
}
```

编译代码，可以看到编译器报错了：

```text
error[E0382]: use of moved value: `x`
  --> l11.rs:11:16
   |
9  |     let x = Foobar(1);
   |         - move occurs because `x` has type `Foobar`, which does not implement the `Copy` trait
10 |     use_foobar(x);
   |                - value moved here
11 |     use_foobar(x);
   |                ^ value used here after move

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`
```


## <span class="section-num">2</span> `Drop` trait {#drop-trait}

Rust 的值在超出其作用域以后会被释放，内存也会被回收，这没什么槽点，因为在 C 语言中，所有的栈上的 `局部变量` 也是这样的。 我们按照自己的思维定势来分析下发生了什么。

```rust
fn main() {
    let x = Foobar(1); // 局部变量 x, 没问题。

    uses_foobar(x); // 传入 x 使用。
    uses_foobar(x); // 传入 x 使用。
}
// main 函数退出， x 被释放回收。
```

`Drop` trait 是 Rust 中变量释放时运行的清理代码。其实现如下:

```rust
impl Drop for Foobar {
    fn drop(&mut self) {
        println!("Dropping a Foobar: {:?}", self);
    }
}
```

这里提前引入了 `&mut self` 这种参数传递方式，后面很快就解释它。为了更清楚得分析，故意创造几个作用域并在关键点进行打印。

```rust
fn main() {
    println!("before enter scope");
    {
        println!("enter scope");
        let x = Foobar(1);
        println!("before uses_foobar");
        //uses_foobar(x);
        println!("after uses_foobar");
        println!("will leave scope");
    }
    println!("leave scope");
}
```

以上代码输出:

```text
before enter scope
enter scope
before uses_foobar
after uses_foobar
will leave scope
Dropping a Foobar: Foobar(1)
leave scope
```

符合我们的预期，现在加入 `uses_foobar` 调用：

```rust
fn main() {
    println!("before enter scope");
    {
        println!("enter scope"); let x = Foobar(1); println!("before uses_foobar"); uses_foobar(x);
        println!("after uses_foobar");
        println!("will leave scope");
    }
    println!("leave scope");
}
```

输出：

```text
before enter scope
enter scope
before uses_foobar
I consumed a Foobar: Foobar(1)
Dropping a Foobar: Foobar(1)
after uses_foobar
will leave scope
leave scop
```

看 `Dropping` 的时机，两个代码段明显不一样，而且第二个代码段的输出明显和我们预想的不一样，发生了什么？

从感觉上来说，两段代码中 `x` 的释放时机应该没有区别. 但实际上，在第二段代码中，
x 在 `uses_foobar(x);` 之后就被释放了。这就是 Rust 所特有的所有权系统所起的作用。

众所周知，语言中的变量的生命周期都是基于词法域的。在 Rust 中，除了变量具有生命周期， `值` 也有生命周期，每个 `值` 都 **有且只有** 一个其属主变量。 一旦 `值` 的属主变量的生命周期结束，则值的生命周期也结束。当然，如果任何情况下， `值` 的生命周期和 `变量` 的生命周期一致的话，所有权系统也就没有存在的必要了。既然其存在，就必然有一些情况下， `值` 的生命周期和其属主变量的生命周期不一致。其中很常见的一种情况就是：把 `值` 从其属主变量赋值给了另外一个变量，则新的变量就变成了 `值` 的属主变量， `值` 的生命周期就保持和新的属主变量的生命周期保持一致。从这个角度来解释上面的代码段二就是:

把 `x` 传递给 `uses_foobar` 函数时， `Foobar(1)` 的属主从变量 `x` 变成了函数
`fn uses_foobar(foobar: Foobar)` 的形参 `foobar`,
在 `uses_foobar` 函数体结束后，形参 `foobar` 的生命结束， `Foobar(1)` 的生命周期也随着 `foobar` 的生命周期结束而结束，故而调用了 `Drop` trait. 而代码一中，
`Foobar(1)` 的属主变量从未改变过，一直是 `x`, 所以在 x 退出其词法域而结束其生命周期时， `Foobar(1)` 的生命周期才结束。

那搞得如此复杂的目的在于什么呢？或者说 Rust 搞这一套复杂的机制是为了解决什么问题呢？主要为了两个目的：

1.  对于值，可以严格判定其生命周期，一旦其属主根据词法域结束生命周期后，就可以调用 `drop` 自动释放，从而做到了对于内存管理的 `零抽象`. 想想在 C/C++ 语言中的
    `malloc` 和 `free`, 一方面，我们不得不时刻紧记要 `free` 内存，以避免内存泄漏。另一方面，我们还要时刻注意不要过度 `free`, 从而造成野指针（好吧，其实现在 C++
    有智能指针了）。另外这种自动内存的管理不是通过 `引用计数` 或者 `GC` 来进行的，而是在编译期就可以确定的，避免使用一个保持 `引用计数` 或者 `GC` 能正常运行的运行时。
2.  对于多线程情况下，可以严格控制值的访问，避免出现多个线程代码同时访问同一个变量而引发的 BUG. 这种竟态往往是 BUG 出现的主要因素并且难以避免，难以复现，难以调试。往往我们需要借助静态代码分析工具来仔细分析，还不一定能够全部避免。按照这种所有权机制所提供的策略编写代码确可以 100% 避免这种情况，虽然这种策略看上去非常死板，不够灵活。但是作为一个码农来说，写正确的代码才是第一位的。当然不遵循这种策略可能也能写出安全的代码，但是遵循这种策略确一定能写出安全的代码，并且还自带静态分析工具，我们何乐而不为呢？
