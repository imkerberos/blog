+++
title = "开始之前的环境配置"
author = ["Eviler"]
date = 2019-12-18
lastmod = 2019-12-18T19:21:26+08:00
tags = ["Rust"]
categories = ["计算机"]
draft = true
+++

我的环境是 MacOS, 并且使用 HomeBrew 来管理软件的安装。

安装 `rust-init`

```text
brew install rustup-init
```

修改环境变量，在 `~/.zshrc` 文件中添加如下配置：

```text
export CARGO_HOME=/usr/local/var/cargo
export RUSTUP_HOME=/usr/local/var/rustup
export RUSTUP_DIST_SERVER="https://mirrors.ustc.edu.cn/rust-static"
export RUSTUP_UPDATE_ROOT="https://mirrors.ustc.edu.cn/rust-static/rustup"
export PATH="${PATH}:${CARGO_HOME}/bin"
```

以上的配置解释：
`cargo` 是 Rust 的包管理软件， `CARGO_HOME` 用来配置 cago 包的安装目录，我更喜欢安装在 `/usr/local/var/cargo` 目录下。
`rustup` 是
`RUST_DIST_SERVER` 和 `RUST_UPDATE_ROOT`: 避免 GFW 的干扰，使用中科大的镜像。

使配置生效：

```text
source ~/.zshrc
```

```text
rustup default stable
rustup component add rls
```

1.  安装 `Visual Studio Code`
2.  安装 `Rust Extension Pack` 插件。
3.  安装 `CodeLLDB` 插件。
4.  安装 `Code Runner` 插件。
