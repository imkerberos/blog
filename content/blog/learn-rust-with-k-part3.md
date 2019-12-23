+++
title = "陪老 K 一起学 Rust (三)"
author = ["Eviler"]
lastmod = 2019-12-23T18:56:09+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1001
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
        println!("enter scope");
        let x = Foobar(1);
        println!("before uses_foobar");
        uses_foobar(x);
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


## <span class="section-num">3</span> 词法空间 {#词法空间}


## <span class="section-num">4</span> 借用和引用 {#借用和引用}


## <span class="section-num">5</span> 同时引用 {#同时引用}


## <span class="section-num">6</span> 可变引用 {#可变引用}
