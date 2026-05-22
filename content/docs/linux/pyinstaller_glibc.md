# GLIBC_2.33 not found 

* 代表你目前的樹莓派 32 位元系統（通常是舊版的 Raspberry Pi OS 11 Bullseye 或更早版本）內建的核心 C 函式庫（glibc）最高只支援到 2.31，
* 然而你安裝的最新版 pyinstaller（或你安裝的第三方 Python 套件如 pydantic-core、grpcio 等）在發行時，是在需要 GLIBC 2.33 以上的新系統中編譯出來的。 [1, 2, 3] 
  
由於 glibc 是 Linux 系統的最底層核心，絕對不能強行手動升級（否則會導致整個樹莓派系統崩潰、無法開機）。請選擇以下兩種最佳解決方法之一：
------------------------------
## 方案 A：最徹底的解決方法（推薦）
將樹莓派系統直接升級到 Raspberry Pi OS 12 (Bookworm) 或更新版本。

* 原因：新版的 Bookworm 系統（無論 32 位元或 64 位元）自帶的 glibc 版本為 2.36 以上，完全可以相容並執行需要 2.33 的打包檔案。
* 作法：建議使用電腦上的 Raspberry Pi Imager 燒錄工具，拿一張新的記憶卡重新燒錄最新版的 Raspberry Pi OS (32-bit)，並將程式移過去打包。 [1] 

------------------------------
## 方案 B：不想重灌系統？改採「降級相容」法
如果你必須留在現有的舊系統上，可以透過降級 PyInstaller 以及重新現場編譯過時套件來解決這個相容性斷層。
## 步驟 1：徹底解除安裝最新的 PyInstaller
先清除已經安裝、不相容的最新版本：
```
pip3 uninstall pyinstaller -y
```
## 步驟 2：安裝相容舊版系統的 PyInstaller
安裝 v5.x 版本，這個版本對舊版系統（GLIBC 2.31）有很好的相容性： [4] 
```
pip3 install "pyinstaller<6.0"
```
## 步驟 3：揪出並處理「隱藏的罪魁禍首」（重要）
有時候拋出 GLIBC_2.33 not found 的不一定是 PyInstaller 本身，而是你程式裡 import 的第三方二進位套件（例如 pydantic-core、grpcio、cryptography 等）。當初你用 pip 安裝它們時，pip 貼心地自動下載了官方編譯好的 .whl（輪子檔），而那個檔案正是在高版本系統下編譯的。 [1, 2] 
解決辦法：強制在樹莓派本地端「現場重新編譯」這些套件。以最常踩雷的套件為例：

# 1. 先卸載那些可能引起問題的套件
```
pip3 uninstall pydantic pydantic-core grpcio -y
```
# 2. 安裝編譯所需的系統工具
```
sudo apt update
sudo apt install build-essential python3-dev -y
```
# 3. 使用 --no-binary 參數，強制 pip 下載原始碼並在你的樹莓派（GLIBC 2.31 環境）現場編譯
```
pip3 install pydantic pydantic-core grpcio --no-binary :all:
```
(注意：強制編譯會花費較長的時間，請耐心等候。)
## 步驟 4：使用「資料夾模式」重新打包
重新打包時，建議加上 --clean 清除快取，並改用 -D（--onedir 資料夾模式），避免單一檔案模式（-F）在 32 位元環境釋放記憶體時出錯： [4] 
```
pyinstaller --clean -D your_script.py
```
打包成功後，進入 dist/your_script/ 資料夾，執行裡面的可執行檔即可順利運作。
------------------------------
如果你在執行步驟 3 時不確定是哪一個 Python 套件引起 glibc 錯誤，請告訴我你完整的程式碼 import 了哪些套件，我能幫你進一步分析！
```
[1] [https://forums.raspberrypi.com](https://forums.raspberrypi.com/viewtopic.php?t=367111)
[2] [https://github.com](https://github.com/mavlink/MAVSDK-Python/issues/487)
[3] [https://github.com](https://github.com/WiringPi/WiringPi/discussions/207)
[4] [https://blog.csdn.net](https://blog.csdn.net/weixin_56868328/article/details/129032599)
```
