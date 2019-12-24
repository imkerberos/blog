+++
title = "陪老 K 学 Rust (一)"
author = ["Eviler"]
date = 2019-12-18
lastmod = 2019-12-24T16:48:23+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1004
+++

开始之前的环境配置， 本节是繁琐的准备工作，已经完成的同学可以跳过。
<!--more-->


## <span class="section-num">1</span> 安装工具链 {#安装工具链}

我的环境是 MacOS, 并且使用 HomeBrew 来管理软件的安装。

修改环境变量，在 `~/.zshrc` 文件中添加如下配置：

```text
export CARGO_HOME=/usr/local/var/cargo
export RUSTUP_HOME=/usr/local/var/rustup
export RUSTUP_DIST_SERVER="https://mirrors.ustc.edu.cn/rust-static"
export RUSTUP_UPDATE_ROOT="https://mirrors.ustc.edu.cn/rust-static/rustup"
export PATH="${PATH}:${CARGO_HOME}/bin"
if (command -v rustc > /dev/null 2>&1); then
    export RUST_SRC_PATH="$(rustc --print sysroot)/lib/rustlib/src/rust/src"
fi
```

以上的配置解释：

-   `cargo` 是 Rust 的包管理软件， `CARGO_HOME` 用来配置 cago 包的安装目录，我更喜欢安装在 `/usr/local/var/cargo` 目录下。
-   `rustup` 是 Rust 工具链管理命令行工具。
-   `RUST_DIST_SERVER` 和 `RUST_UPDATE_ROOT`: 避免 GFW 的干扰，使用中科大的镜像。
-   `RUST_SRC_PATH` Rust 源代码路径，对标准库的功能进行文档提示或者补全之用。

使配置生效：

```text
source ~/.zshrc
```

安装 `rust-init`

```text
brew install rustup-init
rustup-init
```

安装 Rust 稳定版本的编译器等工具链并设置为默认工具链。

```text
rustup default stable
```

Rust 的编译工具链命名遵循规范： `<channel>[-<date>][-<host>]`. 各个部分说明如下：

```text
<channel>       = stable|beta|nightly|<version>
<date>          = YYYY-MM-DD
<host>          = <target-triple>
```

如： `stable`, `stable-x86_64-pc-windows-msvc`, `nightly-2019-11-04` 等都是合法的工具链名称。特别注意的是 `channel`, `stable` 表示是稳定版本， `nightly` 表示为每日构建版本。部分实验性的功能或者特性只有在 `nightly` 版本中支持。有些第三方库在构建的时候可能要求你的工具链是 `nightly` 版本。但是截至到现在（2019-12-18)
`stable` 版本的特性已经足够我们学习的了。：）

```text
rustup component add rls clippy rust-analysis rust-src rustfmt
```

安装一些辅助用的工具：

-   `rls` 全称是 Rust Language Server, 就是支持微软的 `lsp` 的语言服务器，对编辑器进行语法提示，语义级别的检索以及智能提示等功能。
-   `clippy` Rust 语法检查工具。
-   `rust-analysis` Rust 分析器。
-   `rust-src` Rust 源码。
-   `rustfmt` Rust 源代码格式化工具。

安装完成以后不要忘了检查一下是否安装成功：

```text
╭ kerberos@kmacbookh   ~ 
╰ cargo version
cargo 1.38.0 (23ef9a4ef 2019-08-20)
╭ kerberos@kmacbookh   ~ 
╰ rustc --version
rustc 1.38.0 (625451e37 2019-09-23)
╭ kerberos@kmacbookh   ~ 
╰ rls --version
rls 1.38.0 (7b0a20b 2019-08-11)
╭ kerberos@kmacbookh   ~ 
╰ rustfmt --version
rustfmt 1.4.4-stable (0462008d 2019-08-06)
```


## <span class="section-num">2</span> 安装编辑器 {#安装编辑器}

推荐使用 `Visual-Studio-Code` 作为 Rust 的编辑器，既有语法高亮，配合一些 Rust 插件还能进行智能提示以及调试，还是相当舒心的。

```text
brew cask install visual-studio-code
code --install-extension Swellaby.rust-pack
code --install-extension vadimcn.vscode-lldb
code --install-extension formulahendry.code-runner
```

-   安装 `Visual Studio Code`
-   安装 `Rust Extension Pack` 插件。
-   安装 `CodeLLDB` 插件。
-   安装 `Code Runner` 插件。

比较正式的项目用 `cargo new --bin tutor01` 这种方式合适一点，但是学习的话，都是一些简短的样例代码，用 `cargo` 来创建就有些臃肿，不如直接用 `CodeRunner` 跑单个文件好。


## <span class="section-num">3</span> 创建学习目录 {#创建学习目录}

我打算在 `~/ws/playground/rust` 目录下进行学习并且编写实验性质的代码：

```text
mkdir -p ~/ws/playgroud/rust
cd ~/ws/playground/rust
```

由于 `rustup` 可以根据项目指定不同的工具链版本，这里我们就使用 `stable`:

```text
echo "stable" > rust-toolchain
```

在 `rust-toolchain` 文件中显示指明我们使用 `stable` 的工具链（尽管前面我们仅仅安装了 `stable` 工具链）如果以后你的系统工具链安装成 `nightly` 的话，也不会影响这个工程。相反，如果你想实验某些 `nightly` 的功能的话，完全可以另外开辟一个目录，并在其中创建 `rust-toolchain` 文件，在里面声明 `nightly` 工具链的版本即可。

说了这么多，最后我们以经典的 `hello world` 来结束这么繁琐的设置工作，以证明我们终于可以开始写代码了！

```text
cd ~/ws/playground/rust && code .
```

新建一个文件叫做 `hello.rs` (所有 Rust 的源文件的扩展名都是 `rs`). 输入源代码:

```rust
fn main() {
    println!("Hello, World");
}
```

点击 `CodeRunner` 的运行按钮，就看到 VSCode 的输出了。

{{< figure src="/ox-hugo/rust-hello-world.jpg" caption="&#22270;1&nbsp; rust hello world" width="512" >}}
