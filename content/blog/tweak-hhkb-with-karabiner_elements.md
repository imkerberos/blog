+++
title = "HHKB + Karabiner-Elements 神器配置"
author = ["Eviler"]
date = 2019-12-16
lastmod = 2019-12-18T14:06:37+08:00
tags = ["HHKB", "MacOS", "Karabiner", "Vim", "Emacs"]
categories = ["计算机"]
draft = false
+++

`Karabiner-Elements` 是 MacOS 系统下键盘映射的神器, 以前的名称叫做 `Karabiner`, 自从 MacOS 升级到 10.12 以后, 由于键盘驱动的修改导致了 `Karabiner` 不能正常工作,
作者自己由开发了一个新版本, 并且名字改成了 `Karabiner-Elements`. 现在其版本为:
`12.8.0`, 功能已经非常完善了. 那 `Karabinner-Elements` 有什么神奇的功能呢? 下面就以我的 HHKB 键盘为例, 逐一道来.
<!--more-->


## HHKB 和标准 QWERTY 的键盘布局比较 {#hhkb-和标准-qwerty-的键盘布局比较}

{{< figure src="/ox-hugo/hhkb-layout.png" caption="&#22270;1&nbsp; HHKB 键盘布局" width="600" >}}

{{< figure src="/ox-hugo/mac-qwerty.jpg" caption="&#22270;2&nbsp; QWERTY 键盘布局" width="600" >}}

可以看出几个不同点

1.  HHKB 的功能键和数字键是合二为一的, 为了对应 `1 2 3 4` 等数字键和 `F1 F2 F3
       F4` 等功能键, 标准 QWERTY 键盘的 `` ` `` 和 `~` 被移动到了最右侧.
2.  `Control` 键替代了 `Capslock`, 标准的 `Capslock` 被 `Fn + Tab` 取代.
3.  没有方向键和翻页键.

下面我们就使用 `Karabinder-Elements` 让 HHKB 更顺手.


## Karabiner-Elements 配置 {#karabiner-elements-配置}

首先<https://pqrs.org/osx/karabiner/>下载这个软件, 当前版本是 12.8.0. 安装完成以后,
可以使用 GUI 界面进行简单的配置以及导入互联网上由其他用户分享的复杂配置. 我们自己调教 HHKB 的话, 不免进行比较复杂的配置, 所以还是直接打开 Karabiner-Elements 的配置文件直接修改比较好. 另外这些修改可以被 Karabiner-Elements 实时更新, 不用重启软件,还是相当方便的. 配置文件在 `~/.config/karabiner/karabiner.json` 文件中.


## 配置举例 {#配置举例}


### 右侧 `Command` 改成 `Option`, `Option` 改成 `Control` {#右侧-command-改成-option-option-改成-control}

由于 HHKB 右侧(包括苹果笔记本的键盘上)没有 `Control` 按键, 非常不方便, 反倒是右侧 `Command` 按键用的比较少, 这里把右侧的 `Command` 改成 `Option`, `Option` 改成右 `Control`

```json
"simple_modifications": [
    {
        "from": {
            "key_code": "right_command"
        },
        "to": {
            "key_code": "right_option"
        }
    },
    {
        "from": {
            "key_code": "right_option"
        },
        "to": {
            "key_code": "right_control"
        }
    }
],
```


### `Alt + hjkl` 模拟方向键, `Ctrl-a`, `Ctrl-e` 行首, 行尾 等 {#alt-plus-hjkl-模拟方向键-ctrl-a-ctrl-e-行首-行尾-等}

```json
{
    "description": "Change left_option + hjklnp to arrow for HHKB.",
    "manipulators": [
        {
            "from": {
                "key_code": "h",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "left_arrow"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "l",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "right_arrow"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "j",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "down_arrow"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "k",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "up_arrow"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "a",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "home"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "e",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "end"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "p",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "page_up"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "n",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "page_down"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "open_bracket",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "page_up"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "close_bracket",
                "modifiers": {
                    "mandatory": [
                        "left_option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "page_down"
                }
            ],
            "type": "basic"
        }
    ]
}
```


### `Esc` 模拟 `` ` `` {#esc-模拟}

使用 `Command + Esc` 模拟 `` Command + ` ``, MacOS 的窗口切换热键.

```json
{
    "description": "Simulate CMD+` use CMD+Esc for HHKB",
    "manipulators": [
        {
            "from": {
                "key_code": "escape",
                "modifiers": {
                    "mandatory": [
                        "left_command"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "grave_accent_and_tilde",
                    "modifiers": "left_command"
                }
            ],
            "type": "basic"
        }
    ]
}
```

使用 `Shift + Esc` 输入 `~`

```json
{
    "description": "right_shift + esc to evavluate normal keyboard \"~\" for HHKB",
    "manipulators": [
        {
            "from": {
                "key_code": "escape",
                "modifiers": {
                    "mandatory": [
                        "right_shift"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "grave_accent_and_tilde",
                    "modifiers": "right_shift"
                }
            ],
            "type": "basic"
        }
    ]
}
```

快速双击 `Esc` 发送 \`\` 模拟 vim 中的 mark 切换, 如果是单击 `Esc` 则不变, 发送
`Esc`.

```json
{
    "description": "Double click escape to double '`' for HHKB.",
    "manipulators": [
        {
            "conditions": [
                {
                    "name": "standalone_escape_pressed",
                    "type": "variable_if",
                    "value": 1
                }
            ],
            "from": {
                "key_code": "escape",
                "modifiers": {
                }
            },
            "to": [
                {
                    "key_code": "grave_accent_and_tilde"
                },
                {
                    "key_code": "grave_accent_and_tilde"
                }
            ],
            "type": "basic"
        },
        {
            "from": {
                "key_code": "escape",
                "modifiers": {
                }
            },
            "to": [
                {
                    "set_variable": {
                        "name": "standalone_escape_pressed",
                        "value": 1
                    }
                }
            ],
            "to_delayed_action": {
                "to_if_canceled": [
                    {
                        "set_variable": {
                            "name": "standalone_escape_pressed",
                            "value": 0
                        }
                    },
                    {
                        "key_code": "escape"
                    }
                ],
                "to_if_invoked": [
                    {
                        "set_variable": {
                            "name": "standalone_escape_pressed",
                            "value": 0
                        }
                    },
                    {
                        "key_code": "escape"
                    }
                ]
            },
            "type": "basic"
        }
    ]
}
```


### `Capslock` 和 `Control` 复用 {#capslock-和-control-复用}

单击 `Control` 发送 `Capslock`, 如果是组合键, 发送 `Control` 的组合键.

```json
{
    "description": "Post caps_lock if left_control is pressed alone for HHKB.",
    "manipulators": [
        {
            "from": {
                "key_code": "left_control",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "left_control"
                }
            ],
            "to_if_alone": [
                {
                    "hold_down_milliseconds": 100,
                    "key_code": "caps_lock"
                }
            ],
            "type": "basic"
        }
    ]
}
```

标准 QWERTY 键盘上的 `Capslock` 按键, 单独按 `Capslock` 发送  `Capslock`, 如果按
`Capslock + x` 则发送 `Control + x` 组合键. 把标准键盘的 `Capslock` 的功能和
HHKB 保持一致, 如果你喜欢把 Capslock 当成 `Escape`, 这里修改下即可.

```js
{
    "description": "Post caps_lock if caps_lock is pressed alone otherwise post left_control for HHKB.",
    "manipulators": [
        {
            "from": {
                "key_code": "caps_lock",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "left_control"
                }
            ],
            "to_if_alone": [
                {
                    "hold_down_milliseconds": 100,
                    "key_code": "caps_lock"
                }
            ],
            "type": "basic"
        }
    ]
}
```


### 模拟 iOS 的双击 Shift 切换输入法 {#模拟-ios-的双击-shift-切换输入法}

双击左侧的 `Shift` 发送 `Capslock` 切换大小写, 否则当作标准 `Shift`.

```json

{
    "from": {
        "key_code": "left_shift",
        "modifiers": {
            "optional": [
                "any"
            ]
        }
    },
    "to": [
        {
            "set_variable": {
                "name": "left_shift_pressed",
                "value": 1
            }
        },
        {
            "key_code": "left_shift"
        }
    ],
    "to_delayed_action": {
        "to_if_canceled": [
            {
                "set_variable": {
                    "name": "left_shift_pressed",
                    "value": 0
                }
            }
        ],
        "to_if_invoked": [
            {
                "set_variable": {
                    "name": "left_shift_pressed",
                    "value": 0
                }
            }
        ]
    },
    "type": "basic"
}
```


### 双击 `右 Shift` 模拟 appcode 的智能搜索 {#双击-右-shift-模拟-appcode-的智能搜索}

Xcode 中双击右 `Shift` 发送 `Shift + Command + o` 进行智能搜索(模拟 idea).

```json
{
    "description": "Emulate AppCode double shift to Search Everywhere in Xcode",
    "manipulators": [
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "com.apple.dt.Xcode"
                    ],
                    "type": "frontmost_application_if"
                },
                {
                    "name": "right_shift_pressed",
                    "type": "variable_if",
                    "value": 1
                }
            ],
            "from": {
                "key_code": "right_shift",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "o",
                    "modifiers": [
                        "right_shift",
                        "right_command"
                    ]
                }
            ],
            "type": "basic"
        },
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "com.apple.dt.Xcode"
                    ],
                    "type": "frontmost_application_if"
                }
            ],
            "from": {
                "key_code": "right_shift",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "set_variable": {
                        "name": "right_shift_pressed",
                        "value": 1
                    }
                },
                {
                    "key_code": "right_shift"
                }
            ],
            "to_delayed_action": {
                "to_if_canceled": [
                    {
                        "set_variable": {
                            "name": "right_shift_pressed",
                            "value": 0
                        }
                    }
                ],
                "to_if_invoked": [
                    {
                        "set_variable": {
                            "name": "right_shift_pressed",
                            "value": 0
                        }
                    }
                ]
            },
            "type": "basic"
        }
    ]
}
```

VSCode 中使用双击右 `Shift` 发送 `Command + p` 模拟智能搜索.(idea)

```json
{
    "description": "Emulate AppCode double shift to Search Everywhere in VSCode",
    "manipulators": [
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "com.microsoft.VSCode"
                    ],
                    "type": "frontmost_application_if"
                },
                {
                    "name": "right_shift_pressed",
                    "type": "variable_if",
                    "value": 1
                }
            ],
            "from": {
                "key_code": "right_shift",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "p",
                    "modifiers": [
                        "right_command"
                    ]
                }
            ],
            "type": "basic"
        },
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "com.microsoft.VSCode"
                    ],
                    "type": "frontmost_application_if"
                }
            ],
            "from": {
                "key_code": "right_shift",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "set_variable": {
                        "name": "right_shift_pressed",
                        "value": 1
                    }
                },
                {
                    "key_code": "right_shift"
                }
            ],
            "to_delayed_action": {
                "to_if_canceled": [
                    {
                        "set_variable": {
                            "name": "right_shift_pressed",
                            "value": 0
                        }
                    }
                ],
                "to_if_invoked": [
                    {
                        "set_variable": {
                            "name": "right_shift_pressed",
                            "value": 0
                        }
                    }
                ]
            },
            "type": "basic"
        }
    ]
}
```

iTerm 中双击右 `Shift` 发送 `Ctrl + r` 进行搜索历史, 配合 peco 使用更佳.

```json
{
    "description": "Emulate AppCode double shift to Search History in iTerm2",
    "manipulators": [
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "com.googlecode.iterm2"
                    ],
                    "type": "frontmost_application_if"
                },
                {
                    "name": "right_shift pressed",
                    "type": "variable_if",
                    "value": 1
                }
            ],
            "from": {
                "key_code": "right_shift",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "r",
                    "modifiers": [
                        "right_control"
                    ]
                }
            ],
            "type": "basic"
        },
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "com.googlecode.iterm2"
                    ],
                    "type": "frontmost_application_if"
                }
            ],
            "from": {
                "key_code": "right_shift",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "to": [
                {
                    "set_variable": {
                        "name": "right_shift pressed",
                        "value": 1
                    }
                },
                {
                    "key_code": "right_shift"
                }
            ],
            "to_delayed_action": {
                "to_if_canceled": [
                    {
                        "set_variable": {
                            "name": "right_shift pressed",
                            "value": 0
                        }
                    }
                ],
                "to_if_invoked": [
                    {
                        "set_variable": {
                            "name": "right_shift pressed",
                            "value": 0
                        }
                    }
                ]
            },
            "type": "basic"
        }
    ]
}
```


### 在终端中重度配合 tmux {#在终端中重度配合-tmux}

在终端中（iTerm 和 Mac 终端）配 tmux 使用, 我的 tmux 快捷键是 `Alt - z`, 由于这个配置比较多，这里只截取一部分的配置，其他的配置都是类似的。

```json
{
    "description": "Tmux in Terminal",
    "manipulators": [
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "com.googlecode.iterm2",
                        "com.apple.Terminal"
                    ],
                    "type": "frontmost_application_if"
                }
            ],
            "from": {
                "key_code": "r",
                "modifiers": {
                    "mandatory": [
                        "left_command"
                    ]
                }

            },
            "to": [
                {
                    "key_code": "z",
                    "modifiers": [
                        "left_option"
                    ]
                },
                {
                    "key_code": "r"
                }
            ],
            "type": "basic"
        }
    ]
}
```


## 配置调试技巧 {#配置调试技巧}

可以打开 Karabiner 的配置界面中最后的选项卡 `log` 来查看配置是否正确, 如果有错误,
Karabinder-Elements 会自己输出一条红色的错误日志.

如何获取某个应用窗口的标识？ Karabiner-Elements 内置了一个软件
`Karabiner-EventViewer`, 可以用来查看当前的窗口标识, 键盘按键的名字以及配置内自定义的变量等. 调试起来还是很方便的.
