
---
title: "iAeris 系統 API使用手冊"
date: 2026-06-23
weight: 2
---

 **iAeris 物聯網系統 — Web API 使用手冊**。

這份手冊對參數型別、時間格式、回傳欄位進行了規範化梳理，方便後續與前端網頁（如 Chart.js 趨勢圖）或第三方系統進行對接與維護。

---

# iAeris 物聯網系統 — Web API 使用手冊

## 📌 通用規範

* **基底 URL**：`http://<server_ip>:<port>`
* **資料互動格式**：要求與回傳均採用 `application/json; charset=utf-8`
* **時間字串格式**：統一使用標準補零格式 `YYYY-MM-DD HH:mm:ss`（例如：`2026-06-23 13:15:00`）

---

## 🛠️ API 接口清單

### 1. 單設備歷史趨勢資料

用於 `chart.html` 等單一設備的數據曲線繪製。

* **路徑**：`/api/history`
* **方法**：`GET`
* **請求參數 (Query Params)**：
| 參數名 | 型別 | 必填 | 說明 | 範例 |
| --- | --- | --- | --- | --- |
| `sno` | String | 是 | 設備序號 | `000020E500002018` |
| `index` | String | 是 | 查詢的感測器指標（如 co2, temperature） | `co2` |
| `start` | String | 否 | 起始時間 | `2026-06-23 00:00:00` |
| `end` | String | 否 | 結束時間 | `2026-06-23 23:59:59` |
| `limit` | Integer | 否 | 資料筆數限制（預設/最大 1000） | `1000` |


* **回傳範例 (JSON Array)**：
```json
[
  {
    "x": "2026-06-23 13:15:00",
    "y": 52.0
  },
  {
    "x": "2026-06-23 13:12:00",
    "y": 52.0
  },
  ...
]

```



---

### 2. 獲取所有設備列表

取得系統資料庫中註冊的所有設備序號。

* **路徑**：`/api/devices`
* **方法**：`GET`
* **回傳範例 (JSON Array)**：
```json
[
  "000062C200000288",
  "0000AD7CF81D0045",
  "AE8065C000000140",
  "000020E500002018"
]

```



---

### 3. 獲取在線設備實時狀態

設備連線狀態監控與查詢。

* **路徑**：`/api/onlinedevices`
* **方法**：`GET`
* **回傳範例 (JSON Array)**：
```json
[
  {
    "sno": "000020E500002018",
    "pid": "iAeris12",
    "ip": "192.168.2.60",
    "fwver": "2.17.84",
    "is_online": true,
    "idle_time": -9.291299,
    "msw": 78
  },
  {...},
  ...
]

```



---

### 4. 設備原始日誌查詢

用於 `log.html` 數據表格的分頁或範圍查詢。

* **路徑**：`/api/logs`
* **方法**：`GET`
* **請求參數 (Query Params)**：
| 參數名 | 型別 | 必填 | 說明 | 範例 |
| --- | --- | --- | --- | --- |
| `sno` | String | 是 | 設備序號 | `000020E500002018` |
| `start` | String | 否 | 起始時間 | `2026-06-23 00:00:00` |
| `end` | String | 否 | 結束時間 | `2026-06-23 23:59:59` |
| `limit` | Integer | 否 | 回傳上限（預設 1000） | `1000` |


* **回傳範例 (JSON Object)**：
```json
{
  "columns": [
    "dataid",
    "log_date_time",
    "create_date_time",
    "temperature",
    "rh",
    "co",
    "co2",
    "pm2d5",
    "pm10",
    "hcho",
    "tvoc",
    "o3",
    "noise",
    "maxnoise",
    "sno",
    "room",
    "local_ip",
    "pid",
    "firmware_ver",
    "dbg0",
    "dbg1",
    "dbg2",
    "dbg3",
    "no2",
    "so2",
    "ws",
    "wd",
    "sun_lux",
    "air_pressure",
    "rain",
    "uv",
    "nh3",
    "h2s",
    "heat_index",
    "heat_level",
    "aqi",
    "o2",
    "ch4"
  ],
  "data": [
    {
      "air_pressure": null,
      "aqi": null,
      "ch4": null,
      "co": 0.0,
      "co2": 585.0,
      "create_date_time": "2026-06-23 13:15:00",
      "dataid": 2985457,
      "dbg0": 0,
      "dbg1": 0,
      "dbg2": 0,
      "dbg3": -62,
      "firmware_ver": "2.05.024",
      "h2s": 10.0,
      "hcho": 0.12,
      "heat_index": null,
      "heat_level": null,
      "local_ip": "192.168.2.252",
      "log_date_time": "2026-06-23 13:15:07",
      "maxnoise": null,
      "nh3": 100.0,
      "no2": 20.0,
      "noise": null,
      "o2": null,
      "o3": 5.0,
      "pid": "iAeris57",
      "pm10": 12.0,
      "pm2d5": 5.0,
      "rain": null,
      "rh": 52.0,
      "room": "iAeris54_0140",
      "sno": "AE8065C000000140",
      "so2": 20.0,
      "sun_lux": null,
      "temperature": 26.4,
      "tvoc": 0.15,
      "uv": null,
      "wd": null,
      "ws": null
    },
    {
      "air_pressure": null,
      "aqi": null,
      "ch4": null,
      "co": 0.0,
      "co2": 572.0,
      "create_date_time": "2026-06-23 13:12:00",
      "dataid": 2985454,
      "dbg0": 0,
      "dbg1": 0,
      "dbg2": 0,
      "dbg3": -63,
      "firmware_ver": "2.05.024",
      "h2s": 10.0,
      "hcho": 0.12,
      "heat_index": null,
      "heat_level": null,
      "local_ip": "192.168.2.252",
      "log_date_time": "2026-06-23 13:14:28",
      "maxnoise": null,
      "nh3": 100.0,
      "no2": 20.0,
      "noise": null,
      "o2": null,
      "o3": 5.0,
      "pid": "iAeris57",
      "pm10": 13.0,
      "pm2d5": 6.0,
      "rain": null,
      "rh": 52.0,
      "room": "iAeris54_0140",
      "sno": "AE8065C000000140",
      "so2": 20.0,
      "sun_lux": null,
      "temperature": 26.2,
      "tvoc": 0.15,
      "uv": null,
      "wd": null,
      "ws": null
    },
    ....
  ]
}
```



---

### 5. 多設備歷史趨勢對比

用於 `charts.html` 多機疊加曲線圖。

* **路徑**：`/api/history_multi`
* **方法**：`GET`
* **請求參數 (Query Params)**：
| 參數名 | 型別 | 必填 | 說明 | 範例 |
| --- | --- | --- | --- | --- |
| `snos` | String | 是 | 多個序號，以英文逗號 `,` 分隔 | `sno1,sno2,sno3` |
| `index` | String | 是 | 感測器指標項目 | `temperature` |
| `start` | String | 否 | 起始時間 | `2026-06-23 00:00:00` |
| `end` | String | 否 | 結束時間 | `2026-06-23 23:59:59` |
| `limit` | Integer | 否 | 每台設備的資料上限 | `1000` |


* **回傳範例 (JSON Object)**：
```json
{
  "AE8065C000000140": [
    { "x": "2026-06-23 13:15:00", "y": 52.0 },
    { "x": "2026-06-23 13:12:00", "y": 52.0 }
  ],
  "000020E500002018": [
    { "x": "2026-06-23 13:15:00", "y": 48.5 }
  ]
}

```



---

### 6. 統計區間極值資料 (小時/日/月)

用於分析統計圖表。自動按指定時間區間以統計平均、最大與最小值。

* **路徑**：`/api/extremes`
* **方法**：`GET`
* **請求參數 (Query Params)**：
| 參數名 | 型別 | 必填 | 說明 | 可選值 |
| --- | --- | --- | --- | --- |
| `sno` | String | 是 | 設備序號 |  |
| `index` | String | 是 | 感測器指標項目 |  |
| `interval` | String | 是 | 統計聚合時間區間 | `hour`, `day`, `month` |
| `start` | String | 否 | 起始時間 |  |
| `end` | String | 否 | 結束時間 |  |


* **回傳範例 (JSON Object)**：
```json
{
  "labels": [
    "2026-06-23 12:00:00",
    "2026-06-23 11:00:00",
    ....
  ],
  "avg": [26.383333, 26.715000, ...],
  "max": [27.0, 27.1, ...],
  "min": [25.3, 26.4, ...],
}

```



---

### 7. 所有設備最新資料

用於實時全局監控看板 `dashboard.html` 的一鍵刷新與地圖數值渲染。

* **路徑**：`/api/latest_all`
* **方法**：`GET`
* **回傳範例 (JSON Array)**：
```json
[
  {
    "sno": "000020E500002018",
    "create_date_time": "2026-06-23 11:18:33",
    "idle_seconds": 65.714942,
    "co2": 731.0,
    "pm2d5": 16.0,
    "temperature": 25.1,
    "rh": 59.0
  },
  {
    "sno": "AG302625H0000583",
    "create_date_time": "2026-06-23 11:18:00",
    "idle_seconds": 98.714942,
    "pm2d5": 14.0,
    "pm10": 25.0,
    "tvoc": 0.05
  }
]

```



---

## 🚫 錯誤代碼與狀態說明

本系統之 HTTP 狀態碼遵循 RESTful 規範：

* **`200 OK`**：請求成功，不論是否有查詢到資料均回傳。
* **`400 Bad Request`**：遺失必要參數（例如未傳 `sno` 或 `index`）。
* **`500 Internal Server Error`**：伺服器內部出錯或資料庫鎖定，詳細錯誤堆疊可見 `logs/error.log`。