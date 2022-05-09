---
title: 【開發環境折騰指南】WSL安裝和使用（含PCL和OpenCV编译安装教程）
zhihu-title-image: img/Develop_Environment-0_common_images/0.png
zhihu-tags: 计算机视觉, 点云库PCL, OpenCV
zhihu-url: https://zhuanlan.zhihu.com/p/508281855
---
WSL為Windows下的Linux子系統。本教程適用於Windows 10 （2004版或更新版本）和Windows 11。

在WSL下編譯安裝了PCL和OpenCV，也裝好了需要圖像界面的組件，經測試能正常用OpenCV的imshow()，但不能調用攝像頭。

## 1. 安裝

1. 右鍵點擊開始圖標，以管理員模式打開Powershell，輸入以下指令即可開始自動安裝[^1]。
    ```bash
    wsl --install
    ```
2. 顯示安裝完成後，重新啓動電腦，重啓后電腦會自動繼續WSL的安裝
3. 安裝完成後自動開始設定Ubuntu系統的用戶名和密碼

## 2. 使用

#### Ubuntu更新軟件的命令

一些用於Ubuntu包管理系統apt的命令[^2]。

更新軟件源並升級所有軟件：
```bash
sudo apt-get update && sudo apt-get upgrade
```

更新某個軟件
```bash
sudo apt-get upgrade <軟件名>
```

安裝某個軟件包
```bash
sudo apt-get install <軟件名>
```
列出所有已安裝的軟件包
```bash
sudo apt list --installed
```

刪除某個軟件包
```bash
sudo apt-get remove <軟件名>
```

#### 解壓與壓縮命令

.tar.gz常見於類unix系統，可在Linux或MacOS直接解壓[^3]。

解壓命令
```bash
tar -zxvf FileName.tar.gz               # 解壓到當前路徑
tar -C DesDirName -zxvf FileName.tar.gz # 解壓到目標路徑
```

將DirName和其下所有文件（夾）壓縮
```bash
tar -zcvf FileName.tar.gz DirName
```

#### 下載命令

wget用來從指定的url下載文件[^4]。
```bash
wget <url>
```

#### 刪除文件夾

rm用於刪除文件[^6]。最常用的是刪除一個非空目錄：
```bash
sudo rm -rf <文件夾名>
```

## 3. 編譯安裝PCL庫

#### 編譯前的準備工作

安裝PCL庫的所有依賴，可以建立一個.sh文件，把安裝依賴的過程保存進去
```bash
vim pcl_dependences.sh
```

將以下pcl_dependences.sh的内容複製進去（如果不會用vim，也可以不建立這個.sh文件，直接輸入進行安裝）
```bash
sudo apt-get update
sudo apt-get install -y git build-essential linux-libc-dev
sudo apt-get install -y cmake cmake-gui
sudo apt-get install -y libusb-1.0-0-dev libusb-dev libudev-dev
sudo apt-get install -y mpi-default-dev openmpi-bin openmpi-common
sudo apt-get install -y libpcap-dev
sudo apt-get install -y libflann-dev
sudo apt-get install -y libeigen3-dev
sudo apt-get install -y libboost-all-dev
sudo apt-get install -y libvtk7-dev libvtk7-qt-dev
sudo apt-get install -y libqhull* libgtest-dev
sudo apt-get install -y freeglut3-dev pkg-config
sudo apt-get install -y libxmu-dev libxi-dev
sudo apt-get install -y mono-complete
sudo apt-get install -y libopenni-dev libopenni2-dev
sudo apt-get install -y libgtk2.0-dev
sudo apt-get install -y xfce4
```

保存並退出后運行它即可逐個安裝，最後一項是安裝圖形界面，安裝時會詢問Default display manager，推薦選選gdm3[^11]。
```bash
sh pcl_dependences.sh
```

衆所周知，從源碼安裝開源庫最難的地方就是在编译前安裝好合適的依賴。上面寫的這個所謂的pcl_dependences.sh早晚會過時，那什麽東西不過時呢？官網的安裝文檔[^7]。讀文檔可知，必要的依賴有4個：Eigen, Boost, flann, vtk。我個人建議在編譯前先看看cmake的輸出，看看這四個庫裝好沒有，裝好了再繼續。

如果你用的是windows 11，我沒記錯的話，安裝完wsl會有原生的GUI支持，但Windows 10的wsl并沒有，所以還需要裝個GWSL，直接在windows 10自帶的Microsoft Store就能找到，一鍵安裝，非常方便，每次用wsl的時候打開它，ip啥的全給你整好，啥也不用折騰。

如果你像我一樣采用了GWSL作爲WSL的GUI支持方案的話，打開GWSL需要設置一下：點擊GWSL Distro Tools，然後點擊Display/Audio Auto-Exporting，否則像pcl_viewer這樣的程序不能彈出窗口。

#### 下載源碼

打開[PCL在Github上的Releases頁面](https://github.com/PointCloudLibrary/pcl/releases/)，在最新的版本中找到Assets裏的source.tar.gz，右鍵點擊，複製鏈接[^5]。用wget下載源碼（以下命令是1.12.1版本的例子，實際應該用你剛才複製的鏈接），順便一提，從Windows系統複製的鏈接在wsl裏右鍵就能粘貼
```bash
wget https://github.com/PointCloudLibrary/pcl/releases/download/pcl-1.12.1/source.tar.gz
```

解壓到當前目錄下（不用擔心，解壓後只有一個叫pcl的目錄，目錄下是所有文件）
```bash
tar -zxvf source.tar.gz
```

建議把剛才的pcl_dependences.sh移動到剛解壓的目錄pcl下，並刪除壓縮文件
```bash
mv pcl_dependences.sh pcl
rm source.tar.gz
```

#### 編譯

進入目錄pcl
```bash
cd pcl
```

創建build目錄并進入
```bash
mkdir build && cd build
```

用cmake指令生成makefile文件（最後那倆點別省略，它表示上級路徑）
```bash
cmake -DCMAKE_BUILD_TYPE=None \
 -DCMAKE_INSTALL_PREFIX=/usr/local \
 -DBUILD_GPU=ON \
 -DBUILD_apps=ON \
 -DBUILD_examples=ON ..
```

如果CMake沒有成功生成makefile文件，那大概率是某個依賴沒裝好（畢竟安裝所需的依賴是會隨版本變化而變化的，上邊的那個pcl_dependences.sh適用於1.12.1版pcl，卻不一定適合更新版本，你要做的就是耐心讀CMake反饋的信息，看卡在了哪個依賴上，然後安裝這個依賴的最新版本后再試試生成makefile文件）

開始編譯（j后數字代表用幾個CPU綫程進行編譯，比如我的CPU是雙核四綫程，所以用-j4）
```bash
sudo make -j4
```

編譯時間較長，在我電腦上大概花了一个半小時，如果電腦性能比較强時間會短一點。

編譯完後安裝
```bash
sudo make install
```

安裝完可以測試一下。
```bash
pcl_viewer ../test/car6.pcd
```

## 4. 編譯安裝OpenCV

由於WSL不支持USB串口通訊，OpenCV安裝好是沒法調用電腦自帶的攝像頭的，需要攝像頭的請另請高明。

OpenCV的依賴跟OpenCV 有些重複的，我把重複的不重複的都寫到上面那個pcl_dependences.sh裏了，OpenCV必要的是前兩行和最後兩行，當然GWSL也要裝，GWSL裏的Display/Audio Auto-Exporting也要開，上面pcl編譯安裝裏講過了我就不贅述了。

剛剛發現WSL（Ubuntu）不自帶gdb，可以自己安裝，這個是調試程序用的。
```bash
sudo apt install gdb
```

下載和編譯很像pcl，同樣是從github上下載最新版本的source code，解壓，打開後在裏面創建一個build文件夾，打開build文件夾，用“cmake ..”生成編譯文件，用“make -j4”編譯，編譯完成後用“sudo make install”安裝。

記一下一個容易出錯的點：如果xfce和GWSL裝好了，OpenCV的C++版安裝好了，測試的時候imshow沒出窗口也沒報錯，有可能是忘了加“cv::waitKey(0);”，沒加的話程序顯示窗口后立即關閉，所以看不到窗口。

## 5. VSCode遠程連接WSL的C++開發環境搭建（以OpenCV爲例）

VS Code的開發團隊做了針對WSL的適配，能以遠程連接的方式用Windows環境下安裝的VS Code調試WSL下的程序。

不用在WSL裏裝VS Code，直接打開Windows下安裝的VS Code，安裝以下4個插件（如果你用VS Code打開WSL的文件夾好像會自動提示安裝，具體什麽樣我忘了，直接手動安裝應該也一樣）：
- Remote: WSL
- Remote: SSH
- Remote: SSH: Editing Configuration Files
- Remote: Containers

在WSL下輸入code，系統會自動安裝一個用來打開Windows下VS Code的東西，具體是啥咱不懂，總之裝好后就打開了自動連接好WSL的VS Code。這時候打開插件欄你會發現插件被分成兩種，原版的和WSL般的，安裝C/C++ Extension Pack，如果之前裝過，插件市場裏會顯示install in WSL，也就是在WSL也要裝一遍。

這時候環境已經配的差不多了，爲了方便調試，我們還得配置.vscode/的那套文件，這裏以OpenCV的調試爲例，需要在.vscode/下創建三個文件，tasks.json、launch.json、c_cpp_properties.json。其中c_cpp_properties是C/C++ IntelliSense的配置文件，需要添加OpenCV的頭文件路徑（如下），其它兩個跟編譯和調試相關，具體項目的解釋可以參考我之前講過的[這篇文章](https://zhuanlan.zhihu.com/p/502655608)[^12]，唯一不太一樣的是在編譯指令裏加了-I和一大串東西，-I和後跟的路徑用來include一些外部庫（這裏是OpenCV），後面一大串-lopencv_***是用來連接opencv各個庫的。下面的配置只適用於WSL（路徑格式跟Windows不太一樣）。

c_cpp_properties.json
```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/local/include/opencv4"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/g++",
            "cStandard": "c11",
            "cppStandard": "c++11",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```

tasks.json
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "g++",
            "args": [
                "-fdiagnostics-color=always",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}",
                "-g",
                "-Wall",
                "-static-libgcc",
                "-static-libstdc++",
                "-fexec-charset=UTF-8",
                "-std=c++11",
                "-I",
                "/usr/local/include/opencv4",
                "-lopencv_core",
                "-lopencv_imgcodecs",
                "-lopencv_imgproc",
                "-lopencv_calib3d",
                "-lopencv_dnn",
                "-lopencv_features2d",
                "-lopencv_flann",
                "-lopencv_gapi",
                "-lopencv_highgui",
                "-lopencv_ml",
                "-lopencv_objdetect",
                "-lopencv_photo",
                "-lopencv_stitching",
                "-lopencv_video",
                "-lopencv_videoio"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared",
                "showReuseMessage": false,
                "clear": false
            },
            "problemMatcher": "$gcc"
        },
        {
            "label": "run",
            "type": "shell",
            "dependsOn": "build",
            "command": "${fileDirname}/${fileBasenameNoExtension}",
            "group": {
                "kind": "test",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": true,
                "panel": "shared",
                "showReuseMessage": false,
                "clear": false
            }
        }
    ]
}
```

launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
            "MIMode": "gdb",
            "miDebuggerPath": "/bin/gdb",
            "preLaunchTask": "build"
        }
    ]
}
```

配置好就能用F5愉快地進行調試了。

## 6. 生成SSH密鑰并加入GitHub

WSL的Git和Windows系統是分開的，安裝後需要重新設定[^8]。

如果還沒設定git的用戶名和郵箱，需要先設置好，注意一定要與Github的用戶名和郵箱一致，用戶名多加一個空格都會出問題。
```bash
git config --global user.name  "xxxx"
git config --global user.email  "xxxx"
```

生成SSH密鑰，生成密鑰后系統會先提示你設定密鑰保存位置，建議直接按回車設定爲默認位置。然後會提示你設定密碼，如果不想設就啥也別填，按兩次回車跳過。
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

然後將生成的SSH密鑰添加到ssh-agent。所謂ssh-agent相當於一個密鑰的管理員，首先要把這個管理員“召喚”出來。
```bash
eval "$(ssh-agent -s)"
```

如果輸入上一行指令後系統輸出了Agent pid ***，説明管理員召喚成功了，這時就可以用以下指令將剛才生成的SSH密鑰加入ssh-agent，如果剛才沒有用默認路徑，就需要自己修改一下路徑。
```bash
ssh-add ~/.ssh/id_ed25519
```

如果輸入后顯示Identity added blablabla...就説明添加成功了。

下一步就是登錄Github然後點擊右上角的頭像，在彈出菜單裏選Settings。然後在左邊Access一欄下選SSH and GPG keys。然後點綠色按鈕New SSH key，Title填一個自定義的名字，Key填WSL路徑下.ssh/下的id_ed25519.pub裏的内容，也就是把剛才生成的SSH密鑰的.pub文件裏的内容都複製進來，添加後輸入密碼，確定，就完成了。

如果你和我一樣，是個萬裏挑一的倒霉蛋，在設定完後死活不能clone自己的項目，顯示連接到github.com的22端口連接超時，不要慌，説明你的網做了一些保護機制，把github屏了[^9][^10]。怎麽解決呢？在.ssh/下新建一個名爲config的文件，寫入以下内容，就行了。
```bash
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_ed25519
Port 443
```

順便一提，除了克隆自己的項目，還有種驗證github ssh key設置成功的方法是輸入以下指令，如果返回Hi ***! You've successfully blablabla...就説明設定成功了。
```bash
ssh -T git@github.com
```

[^1]: https://docs.microsoft.com/zh-tw/windows/wsl/install Microsoft - 安裝 WSL
[^2]: https://blog.csdn.net/qq_326324545/article/details/88955605 CSDN - Ubuntu 更新软件的命令
[^3]: https://blog.csdn.net/songbinxu/article/details/80435665 CSDN - Ubuntu 常用解压与压缩命令
[^4]: https://blog.csdn.net/qq_38806886/article/details/88920201 CSDN - ubuntu几种不同的下载指令（apt-get,wget,git clone,pip etc）
[^5]: https://github.com/PointCloudLibrary/pcl/releases/ Github - PointCloudLibrary/pcl - Releases
[^6]: https://blog.csdn.net/qq_33144323/article/details/80036970 CSDN - ubuntu如何删除文件夹？
[^7]: https://pcl.readthedocs.io/projects/tutorials/en/latest/compiling_pcl_posix.html Point Cloud Library - Compiling PCL from source on POSIX compliant systems
[^8]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh Github - Connecting to GitHub with SSH
[^9]: https://stackoverflow.com/questions/15589682/ssh-connect-to-host-github-com-port-22-connection-timed-out stackoverflow - ssh: connect to host github.com port 22: Connection timed out
[^10]: https://zhuanlan.zhihu.com/p/502052781 知乎 - 【開發環境折騰指南】Git和Github配置
[^11]: https://blog.dimojang.com/672.html 配置使用 GWSL 实现 Windows 无缝运行 Linux GUI 应用
[^12]: https://zhuanlan.zhihu.com/p/502655608 知乎 - 【開發環境折騰指南】為 VSCode 配置 C++ 開發環境