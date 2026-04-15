Microsoft Windows [版本 10.0.19045.7058]
(c) Microsoft Corporation. 著作權所有，並保留一切權利。

C:\Users\user>hugo version
hugo v0.160.1-d6bc8165e62b29d7d70ede01ed01d0f88de327e6+extended windows/amd64 BuildDate=2026-04-08T14:02:42Z VendorInfo=gohugoio

C:\Users\user>git --version
git version 2.34.1.windows.1

C:\Users\user>d:

D:\>cd project\web

D:\Project\Web>hugo new site my-note-site
Congratulations! Your new Hugo project was created in D:\Project\Web\my-note-site.

Just a few more steps...

1. Change the current directory to D:\Project\Web\my-note-site.
2. Create or install a theme:
   - Create a new theme with the command "hugo new theme <THEMENAME>"
   - Or, install a theme from https://themes.gohugo.io/
3. Edit hugo.toml, setting the "theme" property to the theme name.
4. Create new content with the command "hugo new content <SECTIONNAME>\<FILENAME>.<FORMAT>".
5. Start the embedded web server with the command "hugo server --buildDrafts".

See documentation at https://gohugo.io/.

D:\Project\Web>cd my-note*

D:\Project\Web\my-note-site>git init
Initialized empty Git repository in D:/Project/Web/my-note-site/.git/

D:\Project\Web\my-note-site>git submodule add https://github.com/imfing/hextra.git themes/hextra
Cloning into 'D:/Project/Web/my-note-site/themes/hextra'...
remote: Enumerating objects: 5846, done.
remote: Counting objects: 100% (1427/1427), done.
remote: Compressing objects: 100% (596/596), done.
remote: Total 5846 (delta 1149), reused 831 (delta 831), pack-reused 4419 (from 2)
Receiving objects: 100% (5846/5846), 6.13 MiB | 6.41 MiB/s, done.
Resolving deltas: 100% (3746/3746), done.
warning: LF will be replaced by CRLF in .gitmodules.
The file will have its original line endings in your working directory

D:\Project\Web\my-note-site>

太棒了！你的環境已經完全準備就緒（`hugo` 和 `git` 都運作正常）。現在我們就來建立一個全新的 Hextra 網站。

請跟著以下步驟在終端機（CMD 或 PowerShell）中執行：

---

### 第一步：建立新的 Hugo 專案
首先，決定你要把專案放在哪裡（例如 `D:\Projects` 或 `C:\Users\user\Documents`），然後進入該目錄並執行：

```powershell
# 建立一個名為 my-hextra-site 的資料夾
hugo new site my-hextra-site

# 進入該資料夾
cd my-hextra-site
```

### 第二步：初始化 Git 並加入 Hextra 主題
Hextra 推薦使用 Git Submodule 的方式安裝，這樣以後更新主題會非常方便。

```powershell
# 初始化 Git 儲存庫
git init

# 將 Hextra 加入為子模組
git submodule add https://github.com/imfing/hextra.git themes/hextra
```

### 第三步：配置網站設定
Hugo 預設會產生一個 `hugo.toml` 或 `hugo.yaml`。為了讓 Hextra 運作，我們需要修改它。請用記事本（或 VS Code）打開專案根目錄下的 `hugo.toml`，將內容替換為：

```toml
baseURL = "https://example.org/"     #這裡需要修改成部屬的網址
title = "我的知識庫"
theme = "hextra"

# 啟用 Hextra 必備的輸出格式（為了搜尋功能）
[outputs]
home = ["HTML", "RSS", "JSON"]
```

### 第四步：建立你的第一份 Markdown 文件
在 Hextra 中，主要的內容都放在 `content/` 資料夾。我們來建立一個首頁：

1.  在 `content` 資料夾內建立一個 `_index.md` 檔案。
2.  貼入以下內容：

```markdown
---
title: 首頁
---

# 歡迎來到我的 Hextra 網站！

這是我的第一個使用 **Hugo** 與 **Hextra** 主題建立的頁面。
你可以在這裡開始撰寫你的筆記或文件。
```

### 第五步：本地預覽（最興奮的一刻！）
回到終端機，輸入以下指令啟動預覽伺服器：

```powershell
hugo server -D
```

**看到結果：**
終端機會顯示 `Web Server is available at http://localhost:1313/`。請打開瀏覽器並輸入該網址。你應該就能看到美美的 Hextra 介面了！



---

### 💡 進階提示：如何增加更多頁面？
Hextra 會根據你的資料夾結構自動生成側邊導覽列。例如：
* 建立 `content/docs/page1.md` -> 網站就會出現 `/docs/page1` 的路徑。
* 建立 `content/docs/_index.md` -> 這會變成 `docs` 目錄的介紹頁。

---

### 下一步是什麼？
現在你在電腦上已經看得到網站了，接下來我們需要：
1.  在 GitHub 上開一個新的 **Repository**。
2.  把這份代碼 **Push** 上去。
3.  設定 **GitHub Actions** 讓它自動變成 HTML。

**你準備好將這個網站推送到 GitHub 了嗎？還是想先調整一下網站的內容？**

## 404 錯誤修正

這確實讓人感到困惑！如果你已經在 **Settings > Pages** 中確認 **Source** 是 **GitHub Actions**，但仍然看到 `404` 或 `Failed to create deployment` 的錯誤，這通常是因為 GitHub Actions 需要一個特定的「權限許可」。

雖然你在網頁上開啟了開關，但該 Actions 腳本本身也需要獲得**寫入 (Write)** 權限才能完成發布。

### 🛠 檢查這兩個隱藏的設定

#### 1. 檢查 Workflow 的權限設定
有時候 GitHub 預設會限制自動化腳本的權限。
* 前往 **Settings** > **Actions** > **General**。
* 拉到最下方的 **Workflow permissions** 區塊。
* 確保選中 **「Read and write permissions」**。
* 勾選下方 **「Allow GitHub Actions to create and approve pull requests」**。 <font color="red">!!重要!!</font>
* 點擊 **Save**。



---

#### 2. 檢查 YAML 檔案中的 `permissions` 區塊
請打開你專案中的 `.github/workflows/deploy.yml`，確保裡面包含了這段權限宣告。如果沒有這段，GitHub 即使開了開關也會拒絕連線：

```yaml
permissions:
  contents: read
  pages: write      # 必須有這一行
  id-token: write   # 必須有這一行
```

---

### 🔍 針對錯誤日誌的進階解析
你的錯誤日誌提到：
> `Error: Failed to create deployment (status: 404)`

這在 `Source: GitHub Actions` 模式下，如果權限都對，還有一個可能是 **Hugo 沒有產生任何檔案**。

**請幫我確認一下：**
1. 點開 Actions 裡那個失敗的任務。
2. 點擊 **`build`** 這個步驟。
3. 展開 **`Run hugo --minify`** 的內容。
4. 看看最後有沒有出現類似 `Web Server is available...` 或 `Total in 123 ms` 的成功訊息？
    * 如果看到 **`Error: module "hextra" not found`**，那是因為主題路徑不對，導致 `public` 資料夾是空的，所以發布時會報 404（因為沒東西可以發布）。

---

### 💡 快速修復嘗試
如果你確定設定都改好了，請嘗試做一次「強制空提交」來重新刷一次 Actions：

```powershell
git commit --allow-empty -m "Re-triggering deployment with correct permissions"
git push
```

**如果還是報錯，請點開 Actions 裡的 `build` 步驟，看看 `hugo` 指令執行時有沒有出現任何紅字（尤其是關於 Theme 或 Submodule 的錯誤）？**