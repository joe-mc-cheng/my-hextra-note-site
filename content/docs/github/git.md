---
title: "git 命令筆記"
date: 2026-05-29
weight: 2
---

# 以現有的專案目錄，建立遠端 git repository
一個尚未建立 Git 儲存庫（Repository）的專案目錄，你可以透過幾個簡單的指令來完成初始化，並將檔案進行第一次的版本控制提交。

請在終端機（Terminal）中切換到你的專案目錄 `project_xx`，然後依照以下步驟執行：

## 1. 初始化 Git 儲存庫

在專案根目錄下執行此指令，這會建立一個隱藏的 `.git` 資料夾，正式讓這個目錄變成 Git 儲存庫。

```bash
git init

```

---

## 2. 建立 `.gitignore` 檔案（強烈建議）

在將檔案加進 Git 之前，建議先建立一個 `.gitignore` 檔案，用來排除不需要進入版本控制的檔案（例如：`__pycache__/`、`.env` 環境變數、IDE 設定檔或本地資料庫檔案）。

你可以使用 `touch .gitignore` 建立檔案，並根據你的開發語言（如 Python / Node.js）寫入常見的忽略清單。

---

## 3. 將檔案加入暫存區（Staging Area）

確認好忽略清單後，將目錄下的所有檔案加入 Git 的追蹤範圍：

```bash
git add .

```

> **提示：** 如果想確認目前有哪些檔案準備被提交，可以執行 `git status` 查看狀態。

---

## 4. 進行第一次提交（Initial Commit）

將暫存區的檔案正式紀錄到 Commit 歷史中：

```bash
git commit -m "Initial commit"

```

---

## 5. 關聯至遠端儲存庫（選用）

如果你打算將這個專案推送到 GitHub、GitLab 或 Gitea 等遠端平台，可以執行以下指令（請將網址替換為你實際建立的遠端 URL）：

```bash
# 確保預設分支名稱為 main
git branch -M main

# 關聯遠端儲存庫
git remote add origin https://github.com/你的帳號/project_xx.git

# 推送到遠端
git push -u origin main

```


當你不使用 `.gitignore` 時，`git add` 的精準控制就變得非常重要。

你可以透過以下幾種更精準的 `git add` 方式，來取代一網打盡的 `git add .`：

### 1. 單一或多個檔案指定

這是最直接的方法，只將你明確指定的檔案加入暫存區：

```bash
# 加入單一檔案
git add main.py

# 同時加入多個指定檔案（用空格隔開）
git add config.json utils.py models/user.py

```

### 2. 利用萬用字元（Wildcards）與路徑篩選

如果你想加入某個資料夾下的特定格式檔案，可以使用語法篩選：

```bash
# 只加入 src 資料夾下的所有 Python 檔案
git add src/*.py

# 加入特定資料夾（包含裡面所有內容）
git add static/images/

```

### 3. 互動式暫存：`git add -p`（強烈推薦）

如果你連同一個檔案裡面的程式碼都想「選擇性」加入，這個指令是神器。它會進入互動模式，把你的修改拆成一個個程式碼片段（Hunks），逐一問你是否要加入：

```bash
git add -p

```

* 輸入 `y`：加入這個片段。
* 輸入 `n`：跳過這個片段。
* 輸入 `q`：退出互動模式。

---

### ⚠️ 不使用 `.gitignore` 的潛在風險（過來人的真心提醒）

雖然手動 `git add` 可以做到精準控制，但在實際專案開發中，**非常不建議完全不寫 `.gitignore**`，原因有以下幾點：

* **手滑風險（Fat-finger Error）：** 哪天你習慣性順手打了 `git add .` 或 `git add -A`，那些不該上去的敏感金鑰（`.env`）、本機測試資料庫（`.db`、`.sqlite3`）或快取（`__pycache__/`）就會瞬間進到暫存區。
* **雜訊干擾：** 當你執行 `git status` 想看有哪些檔案修改時，終端機會被幾百個未追蹤的快取與系統檔案（如 `.DS_Store`、PyCharm 的 `.idea/`）洗版，讓你很難一眼找到真正需要提交的程式碼。
* **歷史包袱：** 敏感資料（如私鑰、密碼）一旦不小心 Commit 進去，就算後面用 `git rm` 刪除，它們依然會留存在 Git 的歷史紀錄中，必須花大功夫用 `git filter-repo` 或 BFG 才能徹底清除。

> **折衷妙招：** 如果你單純是不想在專案目錄下看到 `.gitignore` 這個檔案，你可以把忽略規則寫在本地儲存庫的專屬設定檔中（這不會被推送到遠端）：
> `.git/info/exclude`
> 它的語法跟 `.gitignore` 完全一樣，但它是你這台電腦獨享的秘密忽略清單！



----------------------------
# git 取消 commit (尚未push), 保留已經修改的檔案

使用以下這行最經典、最安全的指令即可：
```
git reset --soft HEAD~1
```
## 💡 執行後的效果：

* HEAD~1：代表回到「目前 commit 的前一次狀態」。
* --soft：代表保留所有修改過的檔案。
* 執行後，你剛才 commit 的檔案會原封不動地退回到暫存區（Staging Area）（就是畫面中綠色的狀態），你可以直接修改檔案，或是重新執行 git commit。

------------------------------
## 🛠️ 延伸情境小幫手

   1. 如果想把檔案退回到「未暫存」狀態（變成紅色的文字）：
   如果你連暫存都不想要，想讓檔案回到完全還沒 git add 的狀態，可以使用 --mixed（這也是 Git 的預設模式）：
```   
   git reset HEAD~1
```   
   2. 如果是想取消「連續好幾次」的 commit：
   如果你想一次取消最近的 3 次 commit，並保留這 3 次修改的所有程式碼，只要把數字改成 3 即可：
```   
   git reset --soft HEAD~3
```   
   3. 如何確認成功了？
   你可以輸入以下指令檢查狀態，確認檔案是否安全保留在暫存區：
```   
   git status
```   
   


