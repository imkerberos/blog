+++
title = "陪老 K 学 Rust (十一)"
author = ["Evilee"]
date = 2020-01-16
lastmod = 2020-01-21T16:16:28+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
+++

`Result` 枚举
<!--more-->

`Option` 可以帮助开发者解决 `空值` 问题，但是 `空值` 并不能帮开发者进行错误处理，在绝大部分情况下，一旦代码出错，开发者需要根据错误的详细信息进行处理，做现场恢复还是使用备选方案，亦或进行日志输出以帮助解决问题。 这些方案，用 `空值` 是无法满足需求的。

但假如把 `Option` 的 `None` 替换成具体的错误信息 `Err`,开发人员根据 `Err` 来进行错误处理，这样 `Result` 这个枚举就产生了。形如：

```rust
enum Result<T,E> {
    Ok(T),
    Err(E),
}
```

当然，针对 `Option` 的 `unwrap` 和 `unwrap_or` 等就不能用于 `Result` 了。

Rust 中并没有 `Exception` 机制，函数的出错全靠 `Result` 来进行判断，这样就要求开发者：

1.  永远不要调用 `panic`, 给调用你 API 的开发者以处理的机会。
2.  永远不要调用导致 `panic` 的代码，如 `unwrap`.
3.  如果可能，函数尽量使用 `Result` 作为返回值，以方便其他开发者处理。

`Result` 中的两个关联值，其中一个是泛型类型为 `T` 的 `Ok`, 另外一个是泛型类型为
`E` 的 `Err`. `T` 类型比较好理解，就是值的类型， `E` 类型呢？是否应该是某些特定的类型？


## <span class="section-num">1</span> 以前的代码从 `Option` 转换成 `Result` {#以前的代码从-option-转换成-result}

在 `Option` 的样例代码 `除法计算` 中，用 `None` 表示无法计算，通过 `division` 的代码实现，开发者可以确定 `None` 是因为除数为 0 而导致无法计算，但是单从
`division` 的原型来看，是无法得知无法计算的原因的。从一个良好的接口设计角度来说，
`fn dvision(dvidend: i32, divisor: i32) -> Result<i32, &str>` 返回一个错误的详细描述会更好。

```rust
fn division(dividend: i32, divisor: i32) -> Result<i32,  &'static str> {
    if divisor == 0 {
        Err("divisor is zero")
    } else {
        Ok(dividend / divisor)
    }
}

fn main() {
    let r1 = division(4, 2);
    println!("4 / 2 = {:?}", r1);
    let r0 = division(4, 0);
    println!("4 / 0 = {:?}", r0);

    match r1 {
        Err(e) => {
            println!("error: {:?}", e);
            println!("error is: {:?}", r1.unwrap_err());
        },
        Ok(value) => {
            println!("r1's value is: {}", value);
            println!("r1's value is: {}", r1.unwrap());
        }
    }

    match r0 {
        Err(e) => {
            println!("error: {:?}", e);
            println!("error is: {:?}", r0.unwrap_err());
        },
        Ok(value) => {
            println!("r0's value is: {}", value);
            println!("r0's value is: {}", r0.unwrap());
        }
    }
}
```

运行输出：

```text
4 / 2 = Ok(2)
4 / 0 = Err("divisor is zero")
r1's value is: 2
r1's value is: 2
error: "divisor is zero"
error is: "divisor is zero
```


## <span class="section-num">2</span> `Result` 处理的方案 {#result-处理的方案}

在使用 `Result` 的情况下，如果每个函数调用都要使用 `match` 进行匹配，其代码的优美程度并不比`ret, err := some_call(arg1, arg2)` 更好。甚至还有可能形成这样的波动拳：

```rust
struct MyError {
    ErrorCode0,
    ErrorCode1,
    ErrorCode2,
    ErrorCode3,
    ErrorCode4,
    ErrorCode5,
    ErrorCode6,
}

fn foo() -> Result<(), MyError>{
    return match call_fn0() {
        Err(e1) => ErrorCode0,
        Ok(v1) => match call_fn1(v1) {
            Err(e2) => ErrorCode1,
            Ok(v2) => match call_fn2(v2) {
                Err(e3) => ErrorCode2,
                Ok(v3) => match call_fn3(v3) {
                    Err(e4) => ErrorCode3,
                    Ok(v4) => match call_fn4(v4) {
                        Err(e5) => ErrorCode4,
                        Ok(v5) => (),
                    }
                }
            }
        }
    }
}
```

这种代码看上去简直丑到姥姥家了，所以 Rust 的错误处理方案也是围绕 `Result` 改了又该，光我听说的就包括： `error_chain`, `failure` 等。


## <span class="section-num">3</span> 在 `main` 函数中使用 `Result` {#在-main-函数中使用-result}
