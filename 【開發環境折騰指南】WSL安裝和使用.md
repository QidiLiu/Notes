---
title: 【開發環境折騰指南】WSL安裝和使用（含PCL编译安装教程）
zhihu-title-image: img/Develop_Environment-0_common_images/0.png
zhihu-tags: 编程
zhihu-url: https://zhuanlan.zhihu.com/p/508281855
---
WSL為Windows下的Linux子系統。本教程適用於Windows 10 （2004版或更新版本）和Windows 11。

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
```

保存並退出后運行它即可逐個安裝，中間會有很多次詢問是否安裝，輸入Y即可
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

[^1]: https://docs.microsoft.com/zh-tw/windows/wsl/install Microsoft - 安裝 WSL
[^2]: https://blog.csdn.net/qq_326324545/article/details/88955605 CSDN - Ubuntu 更新软件的命令
[^3]: https://blog.csdn.net/songbinxu/article/details/80435665 CSDN - Ubuntu 常用解压与压缩命令
[^4]: https://blog.csdn.net/qq_38806886/article/details/88920201 CSDN - ubuntu几种不同的下载指令（apt-get,wget,git clone,pip etc）
[^5]: https://github.com/PointCloudLibrary/pcl/releases/ Github - PointCloudLibrary/pcl - Releases
[^6]: https://blog.csdn.net/qq_33144323/article/details/80036970 CSDN - ubuntu如何删除文件夹？
[^7]: https://pcl.readthedocs.io/projects/tutorials/en/latest/compiling_pcl_posix.html Point Cloud Library - Compiling PCL from source on POSIX compliant systems