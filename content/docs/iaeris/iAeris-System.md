---
title: "本地空品雲端系統技術手冊"
date: 2026-04-15
weight: 2
---

# 🌐 本地空氣品質雲端系統概覽

本系統旨在提供完整的室內空氣品質監測與管理解決方案。

## 📊 系統Dashboard顯示功能

### 1. **設備數據顯示**：支援分群組顯示設備數據，並收集各因子即時數值。
![iAeris 設備看板介面](../../static/images/iaeris-system/dashboard.jpg)

### 2. **設備統計數據顯示**：支援分群組顯示設備統計數據：即時統計設備 在/離線 狀況，以及各因子正常或超規的數量。
<video width="100%" height="auto" autoplay loop muted playsinline>
  <source src="../../static/images/iaeris-system/device_info.mp4" type="video/mp4">
  您的瀏覽器不支援 HTML5 影片播放。
</video>

### 3. **歷史曲線分析**：提供分、時、日、月的均值歷史曲線顯示與下載。
![iAeris 網頁歷史趨勢曲線](../../static/images/iaeris-system/chart.jpg)

### 4. **歷史極值分析**：提供時、日、月的與極值歷史曲線顯示與下載。
![iAeris 網頁歷史極值曲線](../../static/images/iaeris-system/chart2.jpg)
  
### 5. **設備數值比較**：提供多設備量測數值比對。
![iAeris 設備數值比對](../../static/images/iaeris-system/charts.jpg)

### 6. **數據查詢**：支援完整的設備數據查詢與 CSV 下載功能 。
![iAeris 數據查詢下載](../../static/images/iaeris-system/logdata.jpg)

---

## 🔐 帳號權限管理
系統設有嚴謹的權限區分，確保數據安全與設定管理：
### 1. **一般帳號**：系統預設權限，僅可觀看 Dashboard 相關功能(參考上章節說明)。
### 2. **管理帳號**：登入後可切換至管理權限 (網頁右上角有登入/登出功能選單)，可執行以下功能：
![登入](../../static/images/iaeris-system/login.jpg)
#### * 註冊新帳號。
![註冊新帳號](../../static/images/iaeris-system/register.jpg)
#### * 修改密碼。
![密碼修改](../../static/images/iaeris-system/changepass.jpg)
#### * 設定設備因子警示/通知上下限：警示上下限可控制dashboard顯示顏色、通知上下限使用於手機APP 超規/恢復 通知。
![上下限設定](../../static/images/iaeris-system/limit_set.jpg)
#### * 設定設備離線通知。
![離線通知設定](../../static/images/iaeris-system/offline_set.jpg)
#### * 設備群組配置(設備數量多可區分群組，方便管理)。
![設備群組設定](../../static/images/iaeris-system/group_set.jpg)
#### * 設備基礎參數設定。
![設備參數設定](../../static/images/iaeris-system/device_set.jpg)

---

## ⚙️ 硬體與系統需求
為了確保 iAeris 本地雲端系統穩定執行，建議硬體配置如下：

| 項目 | 最低需求 |
| :--- | :--- |
| **CPU** | Intel® Core™ i3 2.3 GHz 或以上 |
| **RAM** | 8 GB 或以上 |
| **系統空間** | 20 GB 根分區 |
| **儲存空間** | 100 GB 資料儲存分區 (用於索引與文檔)  |
| **作業系統** | Ubuntu 18.04+ 或 Windows 10+ |

---

## 🛰️ iAeris 資料傳輸架構說明

系統資料擷取支援以下三種模式：

### 1. HTTP Client 模式 (主動推播)
* **運作機制**：iAeris 設備作為 **Client**，主動透過網路將資料推送到中央系統。
* **技術路徑**：利用 **HTTP POST API** 傳送至伺服器指定連接埠。
* **優點**：架構簡單，適合標準的 Web 伺服器整合。

### 2. MQTT 模式 (發布/訂閱)
* **運作機制**：iAeris 設備擔任 **Publisher (發布者)**，將即時氣體數據發布至主題 (Topics)；中央系統擔任 **Subscriber (訂閱者)** 接收資料。
* **技術路徑**：中間透過 **MQTT Broker** (如 Mosquitto) 進行訊息轉發。
* **優點**：低時延、異步傳輸，適合大規模設備連網。

### 3. Modbus 模式 (主動詢問)
* **運作機制**：中央系統擔任 **Modbus Master (主站)**，主動向設備發出讀取請求；iAeris 設備則為 **Slave (從站)** 響應資料。
* **技術路徑**：支援 **Modbus RTU (RS-485)** 或 **Modbus TCP (Net)** 讀取暫存器資料。
* **優點**：工業級標準，適合與 PLC 或工業控制系統直接連動。
* **注意**：若採用Modbus TCP，設備端必須設置成固定IP。
![設備資料流](../../static/images/iaeris-system/data_flows.jpg)


---

## 🛠 快速設定指南
1. **設定固定 IP**：進入電腦主機的網路設定，手動配置 PC地固定IP模式，設置 IP/Mask/Gateway 與 DNS相關參數。此主機 IP 必須設定到設備端，設備才能將資料回傳到系統。
2. **設備設定連線**：iAeris設備透過 RS485 連線至電腦，使用 `iAerisSW` 軟體，於 `setting` 頁面，將 Cloud 設定為 `Custom cloud(HTTP)`。
![設備設定setp1](../../static/images/iaeris-system/device_set1.jpg)
![設備設定setp2](../../static/images/iaeris-system/device_set2.jpg)
3. **主機導向設定**：使用 `iAerisSW` 軟體，於 `WiFi/TCP` 頁面，先執行 右側 Cloud 中下方的  `Refresh` 按鈕，再將主機固定 IP 填入 Address 欄位，並將 **Port 設定為 1180**。
![設備設定setp3](../../static/images/iaeris-system/device_set3.jpg)  
4. **開啟系統**：瀏覽器輸入 `http://<主機IP>:1180` 即可開啟管理網頁。
![系統圖](../../static/images/iaeris-system/system.jpg)  
