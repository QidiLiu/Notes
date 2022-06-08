---
title: 【開發環境折騰指南】OpenCV-4.6.0(C++)編譯安裝和VSCode(Windows)開發環境配置
zhihu-title-image: img/Develop_Environment-0_common_images/0.png
zhihu-tags: 编程, 算法工程师, 计算机视觉
zhihu-url: https://zhuanlan.zhihu.com/p/526029329
---
本文參考自[這篇文章](https://blog.csdn.net/qq_45022687/article/details/120241068)[^1]，適用於Windows系統下OpenCV4.6.0的編譯安裝。

## 1. 下載安裝必要軟件

1. 安裝[CMake](https://cmake.org/download/)，在下載網頁裏找latest release下的Windows x64 Installer，下載後按默認配置進行安裝即可，安裝完將Cmake的bin目錄加入環境變量
2. 下載OpenCV-4.6.0（我寫這篇文章時最新版是這個，不怕折騰的同學也可以試試最新版），進入[這個鏈接](https://github.com/opencv/opencv/releases)找到最新版本或我現在裝的這個OpenCV 4.6.0，在Assets下點"opencv-4.6.0-vc14_vc15.exe"進行下載，下載下來的其實是一個自解壓程序，選一個你想放opencv的地方作爲解壓路徑，以下是路徑的一些要求：
    - 盡量保存到系統盤某個位置（C盤）
    - 這個位置不應該有權限限制（如果你複製文件到這個位置顯示需要管理員權限，就算有權限限制）
    - 路徑裏不可以有中文
3. 下載安裝VSCode和mingw-w64，VSCode沒啥特殊要求，但mingw-w64的版本必須是x86_64-posix-seh，具體安裝方法可參考我的[這篇筆記](https://zhuanlan.zhihu.com/p/502655608)[^2]，記得將mingw-w64下的bin目錄加入環境變量

## 2. OpenCV編譯與相關環境配置

1. 在OpenCV的路徑下的文件夾"opencv/build/x64/"下新建文件夾"/mingw"
2. 打開CMake，在第一欄"Where is the source code"裏選擇你安裝的opencv路徑下的sources文件夾，在下面一欄"Where to build the binaries"裏選擇你剛才新建的mingw文件夾
3. 點Configure，在下拉菜單中選擇"MinGW Makefiles"，然後在下面的四個選項中選擇"Specify native compilers"，然後點擊"Next"
4. 填編譯器的位置，這裏我們用的是mingw-w64，C的編譯器選擇mingw-w64路徑下的bin裏的gcc.exe，C++選擇同路徑下的g++.exe，然後點擊"Finish"
5. 據説下一步需要科學上網，建議帶著電腦出境再繼續下面的配置[^1]
6. 出境后連上網點"Configure"，Cmake開始生成Cmake配置文件並自動下載依賴
7. 這一次Configure完成後改動一些選項，以下是我的配置，除了下面的配置，其他保持默認狀態
    - 取消勾選
        - BUILD_opencv_python3
        - BUILD_opencv_python_bindings_generator
        - BUILD_opencv_python_tests
        - WITH_IPP
        - WITH_MSMF
        - ENABLE_PRECOMPILED_HEADERS（如果有的话）
        - CPU_DISPATCH（點擊一下清空）
    - 需要勾選
        - BUILD_opencv_world
        - WITH_OPENGL
        - BUILD_EXAMPLES
8. 修改完上述選項后點Configure，這次Configure后應該還有幾項是紅的，再點Configure才全部變白
9. 全部變白后就可以點Generate了，出現Generating done就算配置完成了
10. 打開文件夾"opencv/build/x64/mingw"，會發現裏面已經不是空文件夾了，並且有了個Makefile，按住Shift鍵在這個文件夾裏點鼠標右鍵，選擇用powershell打開
11. 輸入"mingw32-make"，回車（如果想快點可以啟用多綫程編譯，比如我輸入的是"mingw32-make -j 4"，如果你的電腦支持8綫程，後面跟的就是"-j 8"），這個過程大概要半小時到1小時，取決於電腦性能，中間可能會報一些警告，但如果沒中斷，就無所謂，編譯到100%且最後一行不是顯示Error就説明編譯成功了
12. 輸入"mingw32-make install"，回車（這一步不會花很長時間）
13. 將"opencv/build/x64/mingw/"下的bin目錄加入環境變量

## 3. VSCode配置

我懶得再寫了，主要就是修改項目文件夾裏"./vscode"下的三個文件，直接複製在下面了，具體講解請參考我的[這篇筆記](https://zhuanlan.zhihu.com/p/502655608)[^2]。我的opencv和mingw-w64被裝在home目錄下一個叫Utils的（自建的）文件夾裏，你需要根據自己的安裝路徑對下面三個文件做一些改動

c_cpp_properties.json
```json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**",
                "C:/Users/qidil/Utils/opencv/build/x64/mingw/install/include",
                "C:/Users/qidil/Utils/opencv/build/x64/mingw/install/include/opencv2"
            ],
            "defines": [],
            "compilerPath":"C:/Users/qidil/Utils/mingw64/bin/g++.exe",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64"
        }
    ],
    "version": 4
}
```

launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", // 配置类型，这里只能为cppdbg
            "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${workspaceFolder}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，一般设置为false
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录，一般为${workspaceRoot}即代码所在目录 workspaceRoot已被弃用，现改为workspaceFolder
            "environment": [],
            "externalConsole": true, // 调试时是否显示控制台窗口，一般设置为true显示控制台
            "MIMode": "gdb",
            "miDebuggerPath": "C:/Users/qidil/Utils/mingw64/bin/gdb.exe", // miDebugger的路径，注意这里要与MinGw的路径对应
            "preLaunchTask": "g++", // 调试会话开始前执行的任务，一般为编译程序，c++为g++, c为gcc
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                }
            ]
        }
    ]
}
```

tasks.json
```json
{
    "version": "2.0.0",
    "command": "g++",
    "args": [
        "-g",
        "${file}",
        "-o",
        "${fileBasenameNoExtension}.exe",
        "C:/Users/qidil/Utils/opencv/build/x64/mingw/bin/libopencv_world460.dll",
        "-I",
        "C:/Users/qidil/Utils/opencv/build/x64/mingw/install/include",
        "-I",
        "C:/Users/qidil/Utils/opencv/build/x64/mingw/install/include/opencv2"
    ],
    "problemMatcher": {
        "owner": "cpp",
        "fileLocation": [
            "relative",
            "${workspaceFolder}"
        ],
        "pattern": {
            "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
            "file": 1,
            "line": 2,
            "column": 3,
            "severity": 4,
            "message": 5
        }
    },
    "group": {
        "kind": "build",
        "isDefault": true
    }
}
```

然後這個項目的C++代碼就可以直接用F5調試了。

[^1]: https://blog.csdn.net/qq_45022687/article/details/120241068 河旬 - VScode搭建Opencv（C++开发环境）
[^2]: https://zhuanlan.zhihu.com/p/502655608 劉啟迪 - 【開發環境折騰指南】為 VSCode 配置 C++ 開發環境