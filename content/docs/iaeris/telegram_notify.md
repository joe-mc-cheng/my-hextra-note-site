---
title: "本地空品雲端系統Telegram 通知設置指南"
date: 2026-04-16
weight: 2
---

這份文檔詳細記錄了從建立 Telegram 機器人、取得 Token 到獲取聊天室 ID，並將這兩個資料設定到本地空品雲端的完整步驟。

---

# 🛡️ iAeris 系統：Telegram 通知設置指南

本指南將引導您完成 iAeris 空氣品質監測系統的 Telegram 警報通知設定。

## 步驟 1：利用 @BotFather 建立機器人
首先，您需要建立一個專屬的 Telegram Bot 來發送通知。

1.  **搜尋 BotFather**：在 Telegram App 中搜尋 `@BotFather` 並開啟對話。
2.  **開始指令**：輸入 `/newbot`。
3.  **設定名稱 (Name)**：根據提示輸入機器人的顯示名稱（例如：`iAeris_Notifier`）。
4.  **設定使用者名稱 (Username)**：
    * 這必須是唯一的名稱。
    * **規則**：必須以 `bot` 結尾（例如：`my_iaeris_001_bot`）。
5.  **取得 Bot Token**：
    * 成功建立後，BotFather 會發送一段包含 **HTTP API Token** 的文字。
    * **請妥善保存這串 Token**（格式類似 `123456789:ABCdefGHIjklm...`），這就是稍後要填入系統的 `Bot Token`。

---

## 步驟 2：建立聊天室並取得群組 ID (Chat ID)
機器人需要知道要把訊息傳給「誰」或「哪個群組」。

1.  **建立群組**：在 Telegram 建立一個新的群組（例如命名為「空品警報通知中心」）。
2.  **加入機器人**：將您剛才建立的 **username** (帶有 `bot` 字尾的那個) 邀請進入該群組。
3.  **取得 Chat ID**：
    * 在群組內隨便發送一則訊息（例如輸入 `test`）。
    * 開啟瀏覽器，輸入以下網址（將 `[YOUR_BOT_TOKEN]` 換成剛才取得的 Token）：
        `https://api.telegram.org/bot[YOUR_BOT_TOKEN]/getUpdates`
    * 在回傳的 JSON 網頁內容中，尋找 `"chat":{"id": -XXXXXXXXX}`。
    * **注意**：群組的 ID 通常是以負號 `-` 開頭的數字（例如 `-456789123`）。這串數字即為 `Chat ID`。

---

## 步驟 3：在空品設備系統中完成設定
最後將取得的資訊填入 iAeris 系統的管理頁面。

1.  **登入系統**：以管理權限登入 iAeris 本地雲端系統。
2.  **進入通知設定**：找到左側功能列 **空品上下限設定** > 選擇要發通知的設備 > 左側`設備通知上下限屬性` 
   * 先設定每個因子發通知的上下限規格(超出規格會發通知)。
3.  **填入參數**：
   * 找到下方 `Telegram 通知啟用`，設訂成 True
   * 再下方兩個欄位： `Telegram Token`  `Telegram ID` ，分別填入步驟1 & 2 取得的 Token & Chat ID。
   * 確保勾選「更新設定」相關核取方塊。
4.  **儲存並測試**：按下 **Submit** 更新設定。您可以嘗試觸發一個測試警報，確認群組內是否能即時收到通知。
![Telegram通知設定](../../static/images/iaeris-system/telegram_notify.jpg)

---

## 💡 快速檢查清單
* [ ] 機器人 Username 是否以 `bot` 結尾？
* [ ] 機器人是否已成功加入目標群組？
* [ ] Chat ID 是否包含前面的負號 `-`？
* [ ] 系統設定頁面是否已點擊 Submit 更新？



---
*本文件參考自 iAeris 本地雲端系統技術手冊第 22-23 頁。*