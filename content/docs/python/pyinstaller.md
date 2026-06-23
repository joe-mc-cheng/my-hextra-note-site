
---
title: "python pyinstaller 筆記"
date: 2026-06-04
weight: 2
---

# pyinstaller 樹莓派打包執行錯誤 GLIBC_2.33 not found

如果您直接在樹莓派 OS 內打包還會噴 GLIBC_2.33 not found 錯誤，代表問題出在您用 pip 安裝的某個第三方 Python 套件（例如 grpcio、pydantic-core 或 opencv 等）。 [1, 2] 
發生這種情況，是因為該套件的維護者在編譯成二進位檔（Wheel 檔）時，使用的是較新的 Linux 系統（內建 GLIBC 2.33+），而您的樹莓派 OS（例如基於 Debian 11 Bullseye 的舊版系統）系統內建的 GLIBC 只有 2.31。當 PyInstaller 把這個不相容的套件打包進去後，執行時就會報錯。 [2, 3, 4] 
請嘗試以下步驟來徹底解決：
## 步驟一：找出是哪個套件引起（最關鍵）
在樹莓派上打開終端機，不要執行打包好的檔案，而是直接用 Python 執行您的原始碼：

python3 your_script.py


* 如果直接運行就噴錯：畫面上會直接顯示是 import 哪一個套件時引入了 GLIBC_2.33 的錯誤（例如：from grpc._cython import cygrpc）。
* 如果是打包後才噴錯：代表該套件在一般執行時沒事，但 PyInstaller 把它的外部 .so 檔抓進來後在執行環境相衝。 [1] 

## 步驟二：解決套件相容性（三選一）## 方案 A：強制從原始碼編譯該套件（最推薦）
既然官方編譯好的二進位檔不相容您的系統，您可以強制 pip 在您的樹莓派上現場編譯，這樣它就會完美相容您樹莓派目前的 GLIBC 版本。
以最常引發此錯誤的 grpcio 為例（請將名稱替換為您出錯的套件）： [5, 6] 
```
# 1. 先卸載目前的套件
pip uninstall grpcio grpcio-status
# 2. 加上 --no-binary 參數，強制下載原始碼在樹莓派本地端編譯安裝
pip install grpcio==1.44.0 --no-binary=grpcio
pip install grpcio-tools==1.44.0 --no-binary=grpcio-tools
```
(注意：使用 --no-binary 在樹莓派上編譯會消耗較多 CPU 與時間，可能需要等待 10~20 分鐘)。 [5, 6] 
## 方案 B：將樹莓派系統升級至最新版（最省事）
如果您的樹莓派 OS 是舊版的 Bullseye (Debian 11)，其 GLIBC 只支援到 2.31。
直接重新燒錄或升級到最新的 Raspberry Pi OS 12 (Bookworm)，新版系統內建的 GLIBC 版本較高，就能完美原生支援這些新套件，PyInstaller 也能流暢打包，不會再跳出錯誤。 [4, 7] 

## 方案 C：降級 Python 套件版本
如果您不想重新編譯，也不想重灌系統，可以嘗試將該第三方套件降級到 2022 年以前發布的舊版本，那時的 Wheel 檔通常還沒依賴 GLIBC 2.33。
--> 目前降級 pyinstaller 可以解決這個問題。


------------------------------
## 步驟三：清除舊快取並重新打包
當您更新或重新編譯了問題套件後，請務必刪除先前 PyInstaller 殘留的快取，否則它會繼續使用舊的錯誤檔案：
```
# 刪除舊的暫存與打包資料夾
rm -rf build dist *.spec
# 重新執行打包
pyinstaller --onefile your_script.py
```

探討：
* 您程式碼中 import 了哪些第三方庫？
* 輸入 ldd --version 後，畫面上顯示的樹莓派內建 GLIBC 版本是多少？


[1] [https://github.com](https://github.com/mavlink/MAVSDK-Python/issues/487)
[2] [https://forums.raspberrypi.com](https://forums.raspberrypi.com/viewtopic.php?t=367111)
[3] [https://groups.google.com](https://groups.google.com/g/pyinstaller/c/BsciphyIaw0)
[4] [https://github.com](https://github.com/WiringPi/WiringPi/discussions/207)
[5] [https://stackoverflow.com](https://stackoverflow.com/questions/71054519/glibc-2-33-not-found-in-raspberry-pi-python)
[6] [https://stackoverflow.com](https://stackoverflow.com/questions/71054519/glibc-2-33-not-found-in-raspberry-pi-python)
[7] [https://raspberrypi.stackexchange.com](https://raspberrypi.stackexchange.com/questions/148719/problem-with-glibc-when-running-executable-on-raspberry-pi-3-model-b)


----------------
# python 3.7 pyinstaller 打包可以執行，但是 3.10打包成功但是有以下錯誤：

```
[18746] Module object for pyimod02_importers is NULL! 
Traceback (most recent call last):
  File "PyInstaller/loader/pyimod02_importers.py", line 22, in <module>
  File "pathlib.py", line 14, in <module> 
  File "urllib/parse.py", line 39, in <module> 
ModuleNotFoundError: No module named 'ipaddress' 
Traceback (most recent call last): 
   File "PyInstaller/loader/pyiboot01_bootstrap.py", line 17, in <module> 
ModuleNotFoundError: No module named 'pyimod02_importers' 
[18746] Failed to execute script 'pyiboot01_bootstrap' due to unhandled exception!
```

這個錯誤通常是因為 PyInstaller 的版本太舊，無法正確解析 Python 3.10+ 新增或調整過的內建核心基礎模組（通常是 ipaddress 或 pathlib 內部變更），導致 PyInstaller 自帶的引導載入器（Bootloader）崩潰，拋出 pyimod02_importers is NULL 的假象錯誤。 
請按照以下順序操作，即可快速修復這個問題：

## 步驟一：升級 PyInstaller 核心版本（最有效）這通常是新版 Python (3.10、3.11+) 與舊版 PyInstaller 不相容的已知 Bug。請直接將 PyInstaller 升級到支援新版 Python 核心語法的最新版本：

```
bash
# 升級到最新的穩定版 PyInstaller
pip install --upgrade pyinstaller
```

## 步驟二：強制清除 PyInstaller 的快取殘留PyInstaller 非常頑固，如果您沒有清理先前的編譯快取，即便升級了套件，它依然會抓取舊的、錯誤的引導結構檔進行打包。請務必執行清理：
```
bash
# 1. 刪除專案內的暫存與 spec 設定檔
rm -rf build dist *.spec

# 2. 刪除系統使用者資料夾底下的 PyInstaller 全域快取（核心關鍵）
rm -rf ~/.cache/pyinstaller
```

## 步驟三：重新打包您的程式清理乾淨後，再次執行打包指令即可順利運行：
```
bash
pyinstaller --onefile your_script.py
```

💡 為什麼 Python 3.7 沒事，Python 3.10 就會出錯？因為 Python 3.10 在標準庫（如 urllib.parse、pathlib）中引入了對核心模組 ipaddress 的底層依賴優化。舊版的 PyInstaller（例如 5.x 以下版本）在掃描 Python 3.10 基礎環境時，沒把這些基礎模組識別完整，導致最終生成的執行檔在初始化解壓（Bootstrap）階段，找不到基礎模組而直接崩潰。升級 PyInstaller 是最根本且完美的解決方案。 

緣由：ubuntu 18.04 ---> 22.04，
1. Ubuntu 22.04 python=3.10 打包後執行會出錯
2. Ubuntu 18.04 python=3.7 打包後執行OK，複製到 Ubuntu 22.04 也是可以執行。

| Ubuntu 版本 | 默认 Python 版本 |
|---|---|
| Ubuntu 24.04 LTS (Noble Numbat) | Python 3.12 |
| Ubuntu 22.04 LTS (Jammy Jellyfish) | Python 3.10 |
| Ubuntu 20.04 LTS (Focal Fossa)  | Python 3.8 |
| Ubuntu 18.04 LTS (Bionic Beaver)  | Python 3.6 |
| Ubuntu 16.04 LTS (Xenial Xerus) | Python 3.5 |


