---
title: '我的 Mac 2025'
pubDate: '2025-05-01'
description: '我的 Mac 电脑配置，从编辑器到各种效率软件都有'
---

## APP

- [微信输入法](https://z.weixin.qq.com/)：多端同步剪贴板，应用自动切换中英文
- clash：代理软件
- [ikuuu](https://ikuuu.one/auth/login)：机场，用了很久了只要99/年
- [Raycast](https://www.raycast.com/)：剪切板、快速启动、Spotify状态栏等等
- [Cursor](https://www.cursor.com/cn)：编辑器
    - [VSCodeVim](https://github.com/VSCodeVim/Vim)：vim 扩展
    - [Tokyo Night](https://marketplace.visualstudio.com/items?itemName=enkia.tokyo-night)：主题
- [微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)：微信小程序开发
- [小程序开发者工具](https://opendocs.alipay.com/mini/ide/download)：支付宝小程序开发
- [Gifox](https://gifox.app/)：录屏软件，可以直接导出 GIF 图
- [Chrome](https://www.google.com/intl/zh-CN/chrome/)：浏览器
- [Karabiner-Elements](https://karabiner-elements.pqrs.org/)：键盘映射，主要用来把大写键映射为 Hyper 键
    - Capslock Hyper 配置
        
        ```json
        {
            "description": "一个简单的 Capslock Hyper 配置",
            "manipulators": [
                {
                    "description": "按住 Capslock 进入 Hyper mode，只按一下为",
                    "from": {
                        "key_code": "caps_lock",
                        "modifiers": { "optional": ["any"] }
                    },
                    "to": [
                        {
                            "key_code": "right_shift",
                            "lazy": true,
                            "modifiers": ["right_command", "right_control", "right_option"]
                        }
                    ],
                    "to_if_alone": [
                        {
                            "hold_down_milliseconds": 50,
                            "key_code": "left_shift"
                        }
                    ],
                    "type": "basic"
                },
                {
                    "description": " s = left",
                    "from": {
                        "key_code": "s",
                        "modifiers": { "mandatory": ["right_command", "right_control", "right_shift", "right_option"] }
                    },
                    "to": [{ "key_code": "left_arrow" }],
                    "type": "basic"
                },
                {
                    "description": " d = down",
                    "from": {
                        "key_code": "d",
                        "modifiers": { "mandatory": ["right_command", "right_control", "right_shift", "right_option"] }
                    },
                    "to": [{ "key_code": "down_arrow" }],
                    "type": "basic"
                },
                {
                    "description": " e = up",
                    "from": {
                        "key_code": "e",
                        "modifiers": { "mandatory": ["right_command", "right_control", "right_shift", "right_option"] }
                    },
                    "to": [{ "key_code": "up_arrow" }],
                    "type": "basic"
                },
                {
                    "description": " f = right",
                    "from": {
                        "key_code": "f",
                        "modifiers": { "mandatory": ["right_command", "right_control", "right_shift", "right_option"] }
                    },
                    "to": [{ "key_code": "right_arrow" }],
                    "type": "basic"
                },
                {
                    "description": "caps + ;  移动到行尾",
                    "from": {
                        "key_code": "semicolon",
                        "modifiers": { "mandatory": ["right_command", "right_control", "right_shift", "right_option"] }
                    },
                    "to": [
                        {
                            "key_code": "right_arrow",
                            "modifiers": ["left_command"]
                        }
                    ],
                    "type": "basic"
                },
                {
                    "description": "caps + p  移动到行首",
                    "from": {
                        "key_code": "p",
                        "modifiers": { "mandatory": ["right_command", "right_control", "right_shift", "right_option"] }
                    },
                    "to": [
                        {
                            "key_code": "left_arrow",
                            "modifiers": ["left_command"]
                        }
                    ],
                    "type": "basic"
                }
            ]
        }
        ```
        
- [iCloud](https://www.icloud.com/)：照片备份/同步，用 iPhone 的话没啥可说的，买了 2t 的会员一般够用了
- [Notion](https://www.notion.com/)：笔记知识管理

## 设置

- [Raycast 代替 Spotlight](https://manual.raycast.com/hotkey)
- 键盘快捷键
    - 禁用「选择上一个输入法」
    - 禁用「选择”输入法“菜单中的下一个输入法」
    - 禁用「显示”聚焦“搜索」
    - 禁用「显示”访达“搜索窗口」
    - 禁用「使用大写锁定键切换”ABC“输入法」
    - 开启「自动切换到文稿的输入法」

## 命令行和终端工具

- Homebrew：包管理工具
- Git：版本管理
- Fish：开箱即用、速度也快的一个 shell
- Starship：开箱即用、好看的 shell prompt
- Neovim：有了 cursor 之后 neovim 用的比较少了，但还是装一个
- Lazygit：开箱即用的 git 客户端
- Ghostty：还是因为 cursor 的存在，终端相关的东西用的越来越少了，所以改用这个试试看。开箱即用、好看速度快的终端模拟器
- JetbrainsMono nerd font：编程字体

```bash
brew install git fish nvm neovim lazygit starship
brew install --cask ghostty
brew install --cask font-jetbrains-mono-nerd-font
```
