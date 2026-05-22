---
title: "google sheets 資料收集"
date: 2026-05-13
weight: 2
---

靜態網頁可以透過特定的方式「收資料」，但因為它本身只支援靜態檔案（HTML、CSS、JS），所以無法直接執行 PHP 或 Node.js 等後端程式來儲存資料庫。 [1, 2] 
要實現收資料（如表單、問卷、留言），您需要搭配後端服務（Serverless）。 [3] 
## 常見的收資料方式

   1. 使用第三方表單服務 (最簡單)
   將 HTML 表單的 action 指向第三方服務，資料會發送到該服務，而非 GitHub。
   * Netlify Forms：適合靜態網站，自動處理表單資料。
      * Formspree 或 Getform：只要填入 API 網址，即可將表單寄到信箱或儲存到 Google Sheets。
   2. 使用 Google Sheets API 或 Firebase (無伺服器後端)
   使用 JavaScript (AJAX) 將資料發送到後端資料庫。
   * Firebase Realtime Database/Firestore：Google 提供的免費雲端資料庫，適合即時收資料。
      * Google Forms + API：將資料寫入 Google 試算表。
   3. GitHub Issues/Discussion API (適合技術型留言)
   透過 GitHub API 將留言作為 Issue 開啟，雖然這需要較高的 API 權限管理。 [4] 

## 總結

* 可以收資料：HTML 表單 + JavaScript + 第三方服務。
* 不可以做的事情：直接在 GitHub Pages 上讀寫 SQL 資料庫、運行 Python/Node.js 後端服務。 [1, 3] 

建議新手：使用 [Formspree](https://formspree.io/) 或 Netlify Forms 是最快的方式。 [4] 
```
[1] [https://ithelp.ithome.com.tw](https://ithelp.ithome.com.tw/articles/10257673)
[2] [https://vocus.cc](https://vocus.cc/article/68bd2642fd897800019dc1f2)
[3] [https://www.reddit.com](https://www.reddit.com/r/learnprogramming/comments/5191vk/possible_to_use_a_database_on_a_static_site/?tl=zh-hant)
[4] [https://www.reddit.com](https://www.reddit.com/r/webdev/comments/g5dsr3/help_putting_file_upload_on_a_static_webpage_how/?tl=zh-hant)
```


## Google Forms + API:將資料寫入 Google 試算表,提供這個方法的範例程式
這是一個非常經典的實作方式。我們會使用 Google Apps Script (GAS) 建立一個微型後端（API 連結點），讓靜態網頁可以透過表單透過 POST 請求將資料存入 Google 試算表。
-----------------------------
## 步驟 1：建立 Google 試算表與腳本

   1. 新建一個 Google 試算表，在第一列填入標題（例如：姓名, 信箱, 訊息）。
   2. 點擊選單的 「延伸功能」 > 「Apps Script」。
   3. 刪除原本程式碼，貼入以下範例：
```
function doPost(e) {
  try {
    // 取得試算表並定位到第一個工作表
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    
    // 從請求中取得資料 (對應表單的 name 屬性)
    var name = e.parameter.name;
    var email = e.parameter.email;
    var message = e.parameter.message;
    var timestamp = new Date(); // 加入時間戳記
    
    // 將資料寫入新的一列
    sheet.appendRow([timestamp, name, email, message]);
    
    // 回傳成功訊息 (解決跨網域 CORS 問題)
    return ContentService.createTextOutput(JSON.stringify({"result":"success"}))
                         .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({"result":"error", "error": error}))
                         .setMimeType(ContentService.MimeType.JSON);
  }
}
```
## 步驟 2：部署 Web 應用程式 (關鍵)

   1. 點擊右上角 「部署」 > 「新部署作業」。
   2. 選取類型為 「Web 應用程式」。
   3. 執行身分：選「我」(Your Email)。
   4. 誰有權限存取：選 「所有人」 (Anyone) — 這步很重要，否則 GitHub Pages 無法存取。
   5. 點擊部署後，複製產生的 Web 應用程式 URL。

------------------------------
## 步驟 3：GitHub Pages HTML 表單範例
在您的 HTML 檔案中，使用 JavaScript 的 fetch API 來發送資料。
```
<form id="myForm">
  <input type="text" name="name" placeholder="您的姓名" required>
  <input type="email" name="email" placeholder="電子信箱" required>
  <textarea name="message" placeholder="留下訊息"></textarea>
  <button type="submit">送出</button>
</form>

<script>
  const scriptURL = '貼上您剛才複製的_Web_應用程式_URL';
  const form = document.getElementById('myForm');

  form.addEventListener('submit', e => {
    e.preventDefault();
    // 顯示載入中狀態
    fetch(scriptURL, { method: 'POST', body: new FormData(form)})
      .then(response => alert("送出成功！"))
      .catch(error => console.error('錯誤！', error.message));
  });
</script>
```
## 💡 注意事項

* 權限：第一次部署時，Google 會要求授權存取試算表，請務必點選「進階」並允許存取。
* 更新程式碼：如果您修改了 GAS 程式碼，必須再次點擊「部署」並選擇 「管理部署作業」 > 「編輯」 > 「版本：新版本」，網址才會生效。
* 資安：此方法網址是公開的，任何人拿到該網址都能寫入資料到您的試算表。若要防止濫用，可加入簡單的驗證碼或檢查欄位邏輯。


## CORS 問題

這個錯誤在 GitHub Pages 搭配 Google Apps Script (GAS) 時非常常見。即便後端程式碼正確，只要 GAS 執行過程中發生任何錯誤（或是權限未開），Google 的伺服器就不會回傳正確的標頭，進而觸發瀏覽器的 CORS 擋箭牌。
請按照以下三個層次進行檢查與修正：
## 1. 修改前端 Fetch 的呼叫方式
當使用 POST 傳送 FormData 時，有時會觸發「預檢請求 (Preflight)」。請嘗試改用較簡單的傳送方式，並移除手動設定的 Header（讓瀏覽器自動處理）：
```
const scriptURL = 'https://script.google.com/macros/s/AKfycbz9q86WKvUfw8uo-PAXJ-Bxva0KoX-RIB7LF_id_6ay2lQmb802TEYEiIrOM9HFjhbj4A/exec';
const form = document.getElementById('myForm');

form.addEventListener('submit', e => {
  e.preventDefault();
  
  // 使用 URLSearchParams 可以減少部分跨網域問題
  const formData = new URLSearchParams(new FormData(form));

  fetch(scriptURL, { 
    method: 'POST', 
    body: formData,
    mode: 'no-cors' // 💡 關鍵：靜態網頁若不需讀取回傳值，可用 no-cors 避開標頭檢查
  })
  .then(() => {
    alert("資料已送出！(備註：no-cors 模式下無法確認是否成功寫入，請檢查試算表)");
    form.reset();
  })
  .catch(error => console.error('Error!', error.message));
});
```
------------------------------
## 2. 檢查 GAS 端的權限（最有可能的原因）
即便你設定了「所有人」，Google 有時會因為安全性問題攔截請求。請確認：

* 部署設定：進入「管理部署作業」，確認「誰有權限存取」顯示為 「所有人」(Anyone)，而不是「任何擁有 Google 帳戶的人」。
* 手動授權：在 GAS 編輯器中，手動點擊執行一次 doPost 函式（雖然會報錯），但這會強迫跳出 「審核權限」 視窗。請務必完成授權流程。

------------------------------
## 3. 確保 doPost 能夠「安全地回傳」
如果 GAS 執行到一半當掉，它就不會回傳 ContentService，導致 CORS 報錯。請將你的 GAS 程式碼包裹在 try...catch 中，並確保無論如何都回傳結果：
```
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000); // 避免多人同時寫入衝突

  try {
    var doc = SpreadsheetApp.openById("你的試算表ID");
    var sheet = doc.getSheets()[0];
    
    // 取得所有欄位標題 (第一列)
    var headRow = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;
    var row = [];

    // 根據標題自動對應資料 (前端 input 的 name 需與標題一致)
    for (var i in headRow) {
      if (headRow[i] == 'Timestamp') {
        row.push(new Date());
      } else {
        row.push(e.parameter[headRow[i]]);
      }
    }

    sheet.getRange(nextRow, 1, 1, row.length).setValues([row]);

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON);
  } 
  catch (f) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': f }))
      .setMimeType(ContentService.MimeType.JSON);
  } 
  finally {
    lock.releaseLock();
  }
}
```
------------------------------
## ⚠️ 極致排錯法
如果以上都失敗，請執行這項測試：

   1. 將 fetch 中的 method 改為 'GET'。
   2. 在 GAS 中新增一個 doGet(e) 函式，內容與 doPost 一樣。
   3. 直接在瀏覽器網址列輸入你的 scriptURL。
   * 若成功：會看到 {"result":"success"}。
      * 若失敗：會看到 Google 的登入畫面或報錯畫面，這代表是 權限/部署 的問題。
   
如果要我幫你檢查前端代碼與試算表標題的對應關係，請告訴我：

* 你的試算表第一列標題內容是什麼？
* 你的 HTML <input> 的 name 屬性分別是什麼？


## 新增外部GET 來查詢 google sheet 的資料

在 Google Apps Script (GAS) 中，要處理外部的 GET 請求（例如查詢資料），我們需要實作 doGet(e) 函式。這可以讓您的 GitHub Pages 靜態網頁或 Python 腳本透過 URL 帶入參數，直接從 Google Sheets 搜尋並回傳 JSON 資料。
以下是完整的實作與範例：
## 步驟 1：GAS 程式碼 (實作 doGet)
請將這段程式碼加入到您原本的 GAS 專案中。此範例支援「查詢全部資料」或「透過 Email 查詢特定人員資料」。
```
function doGet(e) {
  try {
    var ss = SpreadsheetApp.openById("您的試算表ID");
    var sheet = ss.getSheets()[0]; // 取得第一個工作表
    
    // 取得所有資料列
    var data = sheet.getDataRange().getValues();
    var headers = data[0]; // 第一列是標題 (例如：Timestamp, name, email, message)
    
    // 取得外部傳入的查詢參數 (例如 ?email=test@test.com)
    var searchEmail = e.parameter.email;
    
    var resultRows = [];
    
    // 從第二列開始跑迴圈讀取資料
    for (var i = 1; i < data.length; i++) {
      var row = data[i];
      var rowObject = {};
      
      // 將每一列資料組合為鍵值對 (Key-Value Object)
      for (var j = 0; j < headers.length; j++) {
        rowObject[headers[j]] = row[j];
      }
      
      // 條件篩選：若有帶 email 參數就比對，沒帶就全部抓取
      if (!searchEmail || rowObject["email"] == searchEmail) {
        resultRows.push(rowObject);
      }
    }
    
    // 成功回傳 JSON 資料，並解決 CORS 問題
    return ContentService.createTextOutput(JSON.stringify({ "status": "success", "data": resultRows }))
                         .setMimeType(ContentService.MimeType.JSON);
                         
  } catch (error) {
    // 失敗回傳錯誤訊息
    return ContentService.createTextOutput(JSON.stringify({ "status": "error", "message": error.toString() }))
                         .setMimeType(ContentService.MimeType.JSON);
  }
}
```
💡 重要提醒：修改完 GAS 後，請務必點選「部署」>「管理部署作業」>「編輯」>「選擇新版本」並發布，否則外部會連到舊的程式碼。
------------------------------
## 步驟 2：前端 JavaScript 查詢範例 (GitHub Pages)
在您的網頁中，直接使用 fetch 發送 GET 請求。
```
const scriptURL = '您的_GAS_Web_應用程式_URL';
// 範例 1：查詢全部資料function fetchAllData() {
  fetch(scriptURL)
    .then(res => res.json())
    .then(json => {
      if (json.status === "success") {
        console.log("全部資料：", json.data);
      }
    });
}
// 範例 2：查詢特定 Email 的資料function searchByEmail(email) {
  fetch(`${scriptURL}?email=${encodeURIComponent(email)}`)
    .then(res => res.json())
    .then(json => {
      if (json.status === "success") {
        console.log("搜尋結果：", json.data);
        // 這裡可以將資料渲染到 HTML 畫面中
      }
    });
}
```
------------------------------
## 步驟 3：Python 查詢範例
若使用 Python 進行資料讀取，程式碼如下：
```
import requests
GAS_URL = "您的_GAS_Web_應用程式_URL"
# 1. 查詢全部response = requests.get(GAS_URL)
print("全部資料：", response.json())
# 2. 帶參數條件查詢params = {"email": "bot@example.com"}response_filtered = requests.get(GAS_URL, params=params)
print("特定查詢結果：", response_filtered.json())
```
------------------------------
## ⚠️ 資安考量與效能限制

   1. 資料外洩風險：由於 doGet 必須設定為「所有人 (Anyone)」均可存取，這代表任何人只要拿到你的 GAS 網址，就能看光你試算表內的資料。若資料具敏感性，建議在 URL 參數中加入自訂的私鑰檢查（例如：?key=mySecretPassword123），並在 GAS 內先驗證 key 是否正確才給資料。
   2. 資料量限制：若試算表資料高達數萬行，每次 GET 都用迴圈掃描會導致 GAS 逾時崩潰（執行上限 6 分鐘）。大型資料建議改用 Firebase 或搭配 sheet.getRange() 進行分頁查詢。

------------------------------



