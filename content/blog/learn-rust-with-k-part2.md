+++
title = "程序 1"
author = ["Eviler"]
date = 2019-12-19
lastmod = 2019-12-18T23:15:41+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
+++

程序 1
<!--more-->

```rust
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let alice = Person {
        name: String::from("Alice"),
        age: 30,
    };
    println!("Person: {}", alice);
}
```

编译报错：

<p class="verse">
error[E0277]: \`Person\` doesn't implement \`std::fmt::Display\`<br />
&nbsp;&nbsp;--> t001.rs:11:28<br />
&nbsp;&nbsp;&nbsp;|<br />
11 |     println!("Person: {}", alice);<br />
&nbsp;&nbsp;&nbsp;|                            ^^^^^ \`Person\` cannot be formatted with the default formatter<br />
&nbsp;&nbsp;&nbsp;|<br />
&nbsp;&nbsp;&nbsp;= help: the trait \`std::fmt::Display\` is not implemented for \`Person\`<br />
&nbsp;&nbsp;&nbsp;= note: in format strings you may be able to use \`{:?}\` (or {:#?} for pretty-print) instead<br />
&nbsp;&nbsp;&nbsp;= note: required by \`std::fmt::Display::fmt\`<br />
<br />
error: aborting due to previous error<br />
<br />
For more information about this error, try \`rustc --explain E0277\`<br />
</p>
