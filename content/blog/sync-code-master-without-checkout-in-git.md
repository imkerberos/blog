+++
title = "git 不 checkout 主干分支的情况下如何同步代码"
author = ["Evilee"]
date = 2021-03-11
lastmod = 2021-03-11T21:09:38+08:00
draft = false
creator = "Emacs 27.1 (Org mode 9.5 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

git 的项目, 如果项目代码很臃肿, 分支之间的切换就比较耗时间, 这时后, 把当前开发分支的代码合并进主干并更新需要经过漫长的等待. 而且这种分支切换将导致的文件修改时间变更.
再次编译的时候, omg... 想想, checkout 时候都需要漫长等待, 何况 compile 乎.
<!--more-->
一个命令解决痛点:
代码同步: 1. 更新远端主干 2. 合并主干到当前分支 3. 合并当前分支到主干 4. 推送主干到远端

\#+begin\_src sh
git fetch origin develop/iPhone:develop/iPhone && git merge develop/iPhone && git fetch . $(git branch --show-current):develop/iPhone && git push
\#+end\_quote


## 不 checkout 进行 pull 的指令 {#不-checkout-进行-pull-的指令}

\#+begin\_src sh
git fetch origin remote\_branch:local\_branch
\#+end\_quote


## 不 checkout 进行合并的指令 {#不-checkout-进行合并的指令}

\#+begin\_src sh
git fetch . source\_branch:destnation\_branch
\#+end\_quote


## 获取当前分支的指令 {#获取当前分支的指令}

\#+begin\_src sh
echo $(git branch --show-current)
\#+end\_quote
