---
title: 【開發環境折騰指南】為 VSCode 配置 C++ 開發環境
zhihu-title-image: img/Develop_Environment-VSCode_C++/0.png
zhihu-tags: C/C++, Visual Studio Code, 开发工具
---
本文大多數步驟參考自知乎的一篇配置教程：[知乎 - 挑把趁手的兵器——VSCode配置C/C++学习环境（小白向）](https://zhuanlan.zhihu.com/p/147366852)[^1]。本配置方案適用於**Windows**系統，**VSCode**作爲編輯器打底，**MingGW-w64**裏的**g++**作爲編譯器，一些**VSCode插件**作爲編程輔助。

## 1. 安裝程序和插件

1. 從[VSCode官網](https://code.visualstudio.com/)安裝VSCode[^2]，勾選上加入環境變量的選項。
2. 在VSCode中安裝**插件C/C++**。
3. 下載MinGW-w64：[下載地址](https://sourceforge.net/projects/mingw-w64/files/)，點進去后往下拉，選擇**MinGW-W64 GCC-8.1.0下的x86_64-win32-seh**[^3]。
4. 解壓后將文件夾**mingw64**移動到目錄C:\Program File\下。
5. 將mingw64下的\bin文件夾的完整地址添加到用戶的環境變量Path中。（win10下，進入Path后有個瀏覽按鈕，可以很輕鬆地找到mingw64下的bin目錄並加入Path，不用複製，很方便）
6. 重新啓動電腦。

[^1]: https://zhuanlan.zhihu.com/p/147366852 知乎 - 挑把趁手的兵器——VSCode配置C/C++学习环境（小白向）
[^2]: https://code.visualstudio.com/ VSCode官網
[^3]: https://sourceforge.net/projects/mingw-w64/files/ SOURCEFORGE - MinGW-W64