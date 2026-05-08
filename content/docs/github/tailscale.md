---
title: Github CD with Tailscale
description: 利用TailScale VPN 來執行Github 的自動部署
---

Tailscale 結合 GitHub Actions 的 CD (持續部署) 功能，是一個非常強大的自動化工具，它允許您將代碼直接部署到一個不需要暴露在公網上的私人網路中。 [1, 2] 
這代表您的 CI/CD 流程可以安全地存取位於家庭 NAS、公司內部伺服器或私有雲中的設備，而無需開啟危險的防火牆連接埠。 [3, 4] 
------------------------------
## 1. 核心概念：Tailscale + GitHub Actions

* Tailscale: 建立一個安全的專屬網路 (Tailnet)，無論設備在哪裡，都能像在同一個區域網路內一樣互通。
* GitHub Actions (CD): 一個自動化流程，當您推送 (push) 代碼時，會自動進行編譯、測試和部署。
* 結合效益: GitHub 的機器人 (Runner) 在部署時，會先安裝 Tailscale，以私人 IP 直接連線到您的伺服器進行部署。 [1, 2, 5, 6, 7] 

------------------------------
## 2. 應用場景

* 自動部署到 NAS: 程式碼推送到 GitHub 後，自動更新家中 NAS 上的 Docker 服務。
* 私有伺服器部署: 將軟體直接部署到 AWS、GCP 或 Azure 的私有子網路中。
* 跨區域維護: 無需跳板機，直接維護遠端分公司的設備。 [3, 5, 8] 

------------------------------
## 3. 如何設定 (實作流程)
這是一個利用 GitHub Actions 實作 CI/CD 的典型流程： [1] 

   1. 取得 Tailscale Auth Key:
   * 登入 Tailscale Admin Console。
      * 進入 Settings > Keys。
      * 產生一個新的 Auth key (建議開啟 Reusable 如果需要重複使用，建議勾選 Ephemeral 若為暫時性機器)。
   2. 設定 GitHub Secrets:
   * 在您的 GitHub 儲存庫 (Repository) 中，前往 Settings > Secrets and variables > Actions。
      * 新增一個 Secret，命名為 TAILSCALE_AUTHKEY，內容為剛剛產生的 Key。
   3. 編寫 GitHub Actions 流程檔案 (.github/workflows/main.yml):
   在您的 YAML 檔案中加入 Tailscale 的 action：
   ```
   steps:
     - name: Checkout
       uses: actions/checkout@v3
   
     # 這是關鍵步驟：連接 Tailscale
     - name: Tailscale
       uses: tailscale/github-action@v2
       with:
         authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
   
     # 之後的步驟即可用私有 IP (100.x.x.x) SSH 或部署到伺服器
     - name: Deploy to Server
       run: |
         ssh user@100.x.x.x "cd /app && git pull && docker-compose up -d"
   
   [1] 
```
------------------------------
## 4. 關鍵亮點 (為什麼要用它？)

* 極致安全: 伺服器無需暴露在公網 (0.0.0.0/0)，防禦駭客掃描。
* 無需設定防火牆: 不需要繁瑣的路由器連接埠轉發 (Port Forwarding)。
* 完全免費 (對個人/小團隊): Tailscale 提供免費方案，GitHub Actions 也提供免費額度。 [3, 9] 

這使得將現代化 DevOps 流程引入家庭實驗室或小型公司內部網路變得非常容易。
```
[1] [https://kucw.io](https://kucw.io/blog/github-actions-intro/)
[2] [https://hackmd.io](https://hackmd.io/@hpc-ntcu/HynKi-nRh)
[3] [https://community.qnap.com](https://community.qnap.com/t/tailscale-for-remote-connection-to-the-nas/5376?tl=zh_TW)
[4] [https://wellstsai.com](https://wellstsai.com/post/tailscale/)
[5] [https://www.reddit.com](https://www.reddit.com/r/digitalnomad/comments/18ozf7k/what_the_heck_is_tailscale_and_why_should_i_use_it/?tl=zh-hant)
[6] [https://ithelp.ithome.com.tw](https://ithelp.ithome.com.tw/articles/10357933)
[7] [https://medium.com](https://medium.com/%E6%8A%80%E8%A1%93%E7%AD%86%E8%A8%98/%E4%BD%95%E8%AC%82-ci-cd-%E5%88%A9%E7%94%A8-github-actions-%E5%81%9A%E4%B8%80%E5%80%8B%E7%B0%A1%E5%96%AE%E7%9A%84-ci-cd-2d55e6dabeed)
[8] [https://www.checkpoint.com](https://www.checkpoint.com/tw/cyber-hub/cloud-security/devsecops/what-is-a-ci-cd-pipeline/)
[9] [https://docs.github.com](https://docs.github.com/zh/get-started/learning-about-github/githubs-plans)
```

# python flask的專案，在github 上有一個repository，push之後，如何讓此專案的 deploy 到特定 TailScale 的主機上

要實現這個流程，核心在於讓 GitHub 的「虛擬工人」臨時加入你的 Tailscale 網路，然後透過 SSH 把代碼傳給你的主機。
以下是針對 Python Flask 專案的具體操作步驟：
------------------------------
## 1. 準備工作 (主機端)
確保你的目標主機（那台要跑 Flask 的機器）已經：

* 安裝並登入 Tailscale。
* 安裝了 SSH Server 並允許連線。
* 主機上已有專案資料夾（例如 /home/user/my-flask-app）。

------------------------------
## 2. 在 GitHub 設定密鑰 (Secrets)
為了安全，不要把密碼或金鑰寫在程式碼裡。請到 GitHub Repository 的 Settings > Secrets and variables > Actions 新增以下三個：

* TAILSCALE_AUTHKEY：從 Tailscale 控制台產生的 Auth Key。
* SERVER_SSH_KEY：你主機的 SSH 私鑰 (Private Key)。
* SERVER_USER：登入主機的 帳號名稱 (例如 ubuntu 或 root)。

------------------------------
## 3. 建立自動化腳本 (GitHub Action)
在你的專案中建立資料夾 .github/workflows/，並新增一個檔案 deploy.yml，內容如下：
```
name: Deploy Flask App
on:
  push:
    branches: [ main ] # 當 main 分支有更新時觸發
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: 檢出程式碼
        uses: actions/checkout@v4

      - name: 連接 Tailscale
        uses: tailscale/github-action@v2
        with:
          authkey: ${{ secrets.TAILSCALE_AUTHKEY }}

      - name: 透過 SSH 部署至主機
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: 100.x.y.z  # <--- 填入你主機的 Tailscale 私有 IP
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /home/user/my-flask-app
            git pull origin main
            source venv/bin/activate
            pip install -r requirements.txt
            sudo systemctl restart flask_app  # 重新啟動 Flask 服務
```
------------------------------
## 4. 具體運行流程是什麼？

   1. 你 Push 代碼： 你在電腦上輸入 git push。
   2. GitHub 啟動工人： GitHub 派出一台虛擬電腦 (Runner)。
   3. 工人加入你的私網： 虛擬電腦執行 tailscale/github-action，輸入 Auth Key 後，它變成了你私有網路裡的一員。
   4. 工人直接對話： 因為在同一個網路，工人現在可以用 100.x.y.z 這個私有 IP 找到你的主機。
   5. 下達指令： 工人透過 SSH 登入主機，執行 git pull 更新代碼，並重啟你的 Flask 服務。

------------------------------
## 💡 關鍵小技巧

* 如何重啟 Flask？ 建議在 Linux 主機上用 systemd 把 Flask 寫成一個服務 (Service)，這樣腳本只要一行 systemctl restart 就能完成更新。
* 安全性： 因為連線是走 Tailscale 隧道，你主機的防火牆 不需要 對全世界開啟 22 號連接埠 (SSH)，只需允許來自 Tailscale 介面的連線。

如果你需要針對特定的環節深入了解：

* 如何設定 systemd 服務 自動執行 Flask
* 如何產生並設定 SSH Key
* Tailscale Auth Key 的有效期限設定

# 如何產生並設定 SSH Key

設定 SSH Key 的目的，是為了讓 GitHub Actions 的「工人」在不輸入密碼的情況下，安全地登入你的 Tailscale 主機。
請按照以下三個步驟操作：
------------------------------
## 1. 在你的電腦（或伺服器）產生金鑰對
打開終端機（Terminal），輸入以下指令。系統會詢問儲存路徑和密碼，請連按三下 Enter 跳過（保持密碼為空，方便自動化）：
```
ssh-keygen -t ed25519 -C "github-actions-deploy"
```
這會產生兩個檔案：

* 私鑰 (Private Key): id_ed25519（絕對不能外洩，要給 GitHub）。
* 公鑰 (Public Key): id_ed25519.pub（要放在你的目標主機上）。

------------------------------
## 2. 設定目標伺服器 (接收端)
你需要把公鑰的內容存入主機，讓它「認得」這把鑰匙。

   1. 查看公鑰內容：
   
       cat ~/.ssh/id_ed25519.pub (複製這串以 ssh-ed25519 開頭的文字)
   
   2. 登入你的 Tailscale 主機。
   3. 將內容貼到 ~/.ssh/authorized_keys 檔案的最後一行：
   
       nano ~/.ssh/authorized_keys# 貼上後儲存退出
   
   4. 確保權限正確：
   ```
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```
   
------------------------------
## 3. 設定 GitHub Repository (發送端)
現在要給 GitHub 私鑰，讓它有權限登入。

   1. 查看私鑰內容：
       cat ~/.ssh/id_ed25519 (複製整段文字，包含 -----BEGIN... 到 ...END-----)
   2. 在 GitHub 專案頁面點選：Settings > Secrets and variables > Actions。
   3. 點擊 New repository secret。
      * Name: SERVER_SSH_KEY
      * Value: 貼上剛才複製的私鑰內容。
   4. 點擊 Add secret 完成。

------------------------------
## 💡 如何測試是否成功？
在你的 GitHub Actions YAML 檔案中，appleboy/ssh-action 就會自動使用這個 SERVER_SSH_KEY 去對接你伺服器上的 authorized_keys。<br>
🚀 下一步建議：
如果你還沒設定好 Flask 的 systemd 服務，我們可以接著處理，這樣部署完才能自動「重啟」你的網頁服務。
如果你想繼續，請告訴我：

* 你的 Flask 進入點檔名 (例如 app.py 或 main.py)？
* 你有使用 Python 虛擬環境 (venv) 嗎？


# 利用 pyinstaller 將進入點編譯成執行檔，並放置到 dist/projectname 目錄之下

既然你的 Flask App 是透過 PyInstaller 打包成獨立執行檔，部署邏輯會從「更新程式碼」轉變為「替換執行檔」。
這種做法更乾淨，因為主機端完全不需要安裝 Python 或依賴套件。以下是修改後的 YAML 流程：
------------------------------
## 1. 修改後的 GitHub Action YAML
這個流程會先在 GitHub 工人端完成打包，再將編譯好的執行檔傳送到你的 Tailscale 主機。
```
name: Build and Deploy Flask Executable
on:
  push:
    branches: [ main ]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: 檢出程式碼
        uses: actions/checkout@v4

      - name: 設定 Python 環境
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: 安裝依賴並打包
        run: |
          pip install -r requirements.txt
          pip install pyinstaller
          # 執行你的打包指令
          pyinstaller --noconfirm your_script.py 
      - name: 連接 Tailscale
        uses: tailscale/github-action@v2
        with:
          authkey: ${{ secrets.TAILSCALE_AUTHKEY }}

      - name: 上傳並重啟服務
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: 100.x.y.z # 你的 Tailscale IP
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            # 停止舊服務
            sudo systemctl stop flask_app || true
            
            # 從 GitHub 工人端抓取新編譯的檔案 (或使用 scp 先傳輸)
            # 註：這裡建議先用 scp 傳輸檔案，或在 script 內處理
```
------------------------------
## 2. 更好的實作建議：先傳輸再重啟
由於 ssh-action 主要是下指令，建議搭配 scp-action 來傳送編譯好的 dist 資料夾：
```
      - name: 傳送執行檔到主機
        uses: appleboy/scp-action@v0.1.7
        with:
          host: 100.x.y.z
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          source: "dist/projectname/*"
          target: "/home/user/deploy/flask_app_new/"
          strip_components: 2

      - name: 覆蓋舊檔並重啟
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: 100.x.y.z
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            # 確保目標路徑存在
            mkdir -p /home/user/flask_app
            # 停止服務
            sudo systemctl stop flask_app
            # 覆蓋執行檔
            cp -r /home/user/deploy/flask_app_new/* /home/user/flask_app/
            # 給予執行權限
            chmod +x /home/user/flask_app/projectname
            # 啟動服務
            sudo systemctl start flask_app
```
------------------------------
## 3. 主機端的 systemd 設定
為了讓部署腳本中的 systemctl restart 生效，你需要在主機建立服務設定檔：

* 位置：/etc/systemd/system/flask_app.service
* 內容範例：
```
[Unit]
Description=Flask PyInstaller Service
After=network.target

[Service]
User=your_user_name
WorkingDirectory=/home/user/flask_app
ExecStart=/home/user/flask_app/projectname
Restart=always

[Install]
WantedBy=multi-user.target
```
------------------------------
## ⚡ 關鍵注意事項

* 作業系統相容性： GitHub 預設工人是 Linux (Ubuntu)。如果你的目標主機是 Windows，runs-on 必須改為 windows-latest。
* 路徑一致性： 請確認 dist/projectname 在 GitHub 打包後的路徑，與你主機存放的路徑對應正確。
* Sudo 權限： systemctl 指令通常需要 sudo。若不想在腳本輸密碼，建議將該使用者加入 sudoers 免密碼清單。

如果你想知道如何設定 sudo 免密碼或是 Windows 平台的打包配置，請告訴我！


# Windows 平台的打包配置
在 Windows 平台上進行 PyInstaller 打包並透過 Tailscale 部署，流程與 Linux 相似，但有幾個關鍵差異：編譯環境必須換成 Windows、路徑表示法不同，以及遠端控制通常使用 PowerShell。
## 1. 修改 GitHub Action YAML
必須將 runs-on 指定為 windows-latest，這樣 GitHub 才會提供 Windows 虛擬機來編譯 .exe。
```
name: Windows Build and Deploy
on:
  push:
    branches: [ main ]
jobs:
  build-and-deploy:
    runs-on: windows-latest  # 🚀 使用 Windows 環境編譯
    steps:
      - name: 檢出程式碼
        uses: actions/checkout@v4

      - name: 設定 Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: 安裝依賴並打包
        run: |
          pip install -r requirements.txt
          pip install pyinstaller
          # 打包成單一執行檔 (--onefile) 或資料夾
          pyinstaller --noconfirm --onefile --name "my_flask_app" your_script.py
      - name: 連接 Tailscale
        uses: tailscale/github-action@v2
        with:
          authkey: ${{ secrets.TAILSCALE_AUTHKEY }}

      - name: 傳送執行檔到 Windows 主機
        uses: appleboy/scp-action@v0.1.7
        with:
          host: 100.x.y.z
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          source: "dist/my_flask_app.exe"
          target: "C:/Deploy/FlaskTemp/" # 🚀 注意 Windows 路徑斜線

      - name: 遠端啟動服務 (PowerShell)
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: 100.x.y.z
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            # 停止舊的進程 (如果存在)
            Stop-Process -Name "my_flask_app" -Force -ErrorAction SilentlyContinue
            # 覆蓋舊檔案
            Copy-Item -Path "C:/Deploy/FlaskTemp/dist/my_flask_app.exe" -Destination "C:/App/my_flask_app.exe" -Force
            # 啟動新程式 (背景執行)
            Start-Process -FilePath "C:/App/my_flask_app.exe" -WindowStyle Hidden
```
------------------------------
## 2. Windows 端的準備工作
要讓上述流程成功，你的 Windows 主機需要完成以下設定：
## 🛠️ 開啟 OpenSSH Server
Windows 10/11 內建有 SSH 功能，但預設關閉：

   1. 設定 > 選用功能 > 搜尋「OpenSSH 伺服器」並安裝。
   2. 開啟 服務 (services.msc)，將 OpenSSH SSH Server 啟動類型改為「自動」並啟動。
   3. 防火牆： 確保 Tailscale 介面允許 22 埠連入（Tailscale 通常會自動處理，但建議確認）。

## 🔑 設定 SSH 公鑰
Windows 的公鑰存放位置與 Linux 不同：

* 一般使用者： C:\Users\YourName\.ssh\authorized_keys
* 管理員權限： C:\ProgramData\ssh\administrators_authorized_keys
* 註：若是管理員帳號，必須編輯 C:\ProgramData\ssh\sshd_config，註解掉最後兩行有關 match Group administrators 的設定，否則金鑰會失效。

------------------------------
## 3. 常見問題與亮點

* UI 視窗： 如果你的 Flask 不想要跳出黑框，打包時加入 --noconsole 參數。
* 背景執行： Windows 上的 Start-Process 會在當前 Session 啟動。如果希望像 Linux 服務一樣永久運行，建議配合 NSSM (Non-Sucking Service Manager) 將 .exe 註冊為 Windows 服務。
* 靜態檔案： 如果有 Templates 或 Static 資料夾，記得在 PyInstaller 指令加上 --add-data "templates;templates"。

------------------------------
📍 提醒：Windows 編譯出的 .exe 只能在 Windows 執行；如果你是在 Linux 主機上運行，請務必切換回 ubuntu-latest。
如果你想了解如何用 NSSM 將 Flask 變成長駐服務，或者遇到 Windows 權限問題，請告訴我！



