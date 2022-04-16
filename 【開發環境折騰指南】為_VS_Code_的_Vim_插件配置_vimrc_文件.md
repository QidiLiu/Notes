---
title: 【開發環境折騰指南】為 VS Code 的 Vim 插件配置 vimrc 文件
zhihu-url: https://zhuanlan.zhihu.com/p/498570422
zhihu-title-image: img/Develop_Environment-vimrc_for_VSCode_Vim/0.png
zhihu-tags: Visual Studio Code, vimrc, 文本编辑器
---
VS Code 和 Vim 混搭著用既可以享受 VS Code 開箱即用的各種插件，也可以享受純鍵盤操作的便捷。較早的配置教程一般都用 VS Code 的 json 配置文件，顯然不如 vim 原生的 vimrc 用起來舒服。本文記錄了筆者用vimrc配置 VS Code 的 Vim 插件的過程。

設置方法來自stackoverflow的一片問答[^1]。

!!! 注意：雖然能用 vimrc，但說到底這個插件是在模擬 Vim，並沒有真正的 Vim 在底層作為支撐，所以無法支持插件，我只用配了我比較習慣的鍵位映射。

## 1. 在 VS Code 中安裝 Vim 插件

沒啥好說的，搜插件 Vim 然後安裝就完了。

![在 VS Code 中安装 Vim 插件](img/Develop_Environment-vimrc_for_VSCode_Vim/1-1.jpg)

## 2. 寫個 vimrc 文件並保存至 $Home 目錄
所謂 $Home 目錄在 Windows 系統裡就是 C盤 => 用戶(Users) => 你用戶名的文件夾。

比如在我這裡用戶名是qidil，我的 $Home 目錄就如圖所示

![我的 $Home 目录](img/Develop_Environment-vimrc_for_VSCode_Vim/2-1.jpg)

如果你用 Linux 系統或 MacOS 系統，應該也差不多，直接找 /home/<用戶名> 目錄就好。

在 $Home 目錄下新建文件 ".vimrc"（別落下文件名里那個點），然後按你自己的需要修改，然後保存。

![文件 ".vimrc"](img/Develop_Environment-vimrc_for_VSCode_Vim/2-2.jpg)

## 3. 設置 Vim 插件

點擊左下方小齒輪，點擊設置/Settings。

![打開設置](img/Develop_Environment-vimrc_for_VSCode_Vim/2-3.jpg)

在最上方的搜索欄搜索 vimrc。

![搜索 vimrc](img/Develop_Environment-vimrc_for_VSCode_Vim/2-4.jpg)

勾選 "Use key mappings from a .vimrc file"。

![勾選 "Use key mappings from a .vimrc file"。](img/Develop_Environment-vimrc_for_VSCode_Vim/2-5.jpg)

然後就能用了。注意，作者在設置裏也寫了，是鍵盤映射。試驗了一下，儘管我在 .vimrc 裡寫了 set relativenumber，但實際使用中並沒有這個效果。

![試驗效果](img/Develop_Environment-vimrc_for_VSCode_Vim/2-6.jpg)

[^1]: https://stackoverflow.com/questions/63017771/how-to-modify-change-the-vimrc-file-in-vscode stackoverflow - How to modify/change the vimrc file in VsCode?