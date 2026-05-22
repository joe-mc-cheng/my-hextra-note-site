# 20260429
Cloudflare 並**不會「經常」**無預警地變更 TLS 1.2 的支援規格，但為了維護網路安全與符合產業合規標準（如 PCI DSS），他們會根據加密技術的發展，**週期性地汰換**過時或具安全性漏洞的加密套件（Cipher Suites）。

以下是關於 Cloudflare 支援 TLS 1.2 的幾個關鍵行為與趨勢：

### 1. 加密套件（Cipher Suites）的汰換
雖然 TLS 1.2 協定本身仍受支援，但並非所有 TLS 1.2 的加密算法都是安全的。Cloudflare 會定期棄用安全性較弱的算法：
* **非轉發安全性（Non-Forward Secrecy）算法：** 例如 RSA 金鑰交換（`TLS_RSA_WITH_...`）。Cloudflare 已排定於 **2025 年下半年**開始分階段淘汰這些 ciphers。
* **CBC 模式加密：** 由於存在如 Padding Oracle 等攻擊風險，這類舊式的 TLS 1.2 加密算法也預計在 **2026 年中**逐步停止支援。

### 2. PCI 合規要求與「最小 TLS 版本」
Cloudflare 預設的最小支援版本通常是 TLS 1.0（為了最大相容性），但對於涉及金流或高安全性需求的網站，他們**強烈建議**並讓用戶自行設定「最小 TLS 版本」為 **1.2** 或 **1.3**。
* 如果你發現連線突然失敗，通常是因為你手動調高了「Minimum TLS Version」，而客戶端設備太舊無法支援。

### 3. TLS 1.3 的推廣
Cloudflare 的策略是「擁抱 TLS 1.3」。他們會持續優化 TLS 1.3 的連線速度（如 0-RTT），並將其設為所有方案的預設開啟選項。雖然 TLS 1.2 仍然是目前網路連線的主力，但 Cloudflare 會讓其維持在「相容模式」，除非該算法被證實有嚴重漏洞（如過去的 3DES），否則不會隨意變更。

---

### 如何確認與調整？
如果你擔心變更會影響到你的應用程式（例如 IoT 設備或舊型伺服器）：
* **查看儀表板：** 在 Cloudflare 控制面板的 `SSL/TLS` -> `Edge Certificates` 中，你可以看到當前的 **Minimum TLS Version** 設定。
* **API 自定義：** 如果你是付費方案（如 Business 或 Enterprise），你可以透過 API 自定義特定的加密套件組合，避免 Cloudflare 自動更新導致舊設備連不上。
* **監測：** 建議觀察 `SSL/TLS` -> `Overview` 下的流量統計，確認目前是否還有大量流量使用舊版的 TLS 1.2 ciphers。

**總結：** Cloudflare 的異動通常會有**數個月的前導預告期**（透過電子郵件或官方 Blog），且主要針對「不安全的加密演算法」而非「整個 TLS 1.2 協定」。對於現代化的軟體開發與 IoT 應用（如你正在開發的 iAeris 系統），只要使用支援 **GCM（Galois/Counter Mode）** 的加密套件（如 `AES-128-GCM-SHA256`），在未來幾年內連線應該都會非常穩定。

## iAeris TLS1.2 支援過程
### 2020 年網站TLS1.2 支援清單
### 1. Protocols
* TLS 1.3 No
* TLS 1.2 Yes
* TLS 1.1 No    `主要關閉 TLS 1.1以下的支援`
* TLS 1.0 No
* SSL 3 No
* SSL 2 No

### 2. Cipher Suites
* TLS_RSA_WITH_3DES_EDE_CBC_SHA (0xa) WEAK 112
* TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (0x16) DH 2048 bits FS WEAK 112
* TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA (0xc012) ECDH secp521r1 (eq. 15360 bits RSA) FS WEAK 112
* TLS_RSA_WITH_AES_128_CBC_SHA (0x2f) WEAK 128
* TLS_DHE_RSA_WITH_AES_128_CBC_SHA (0x33) DH 2048 bits FS WEAK 128
* TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (0x41) WEAK 128
* TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (0x45) DH 2048 bits FS WEAK 128
* TLS_RSA_WITH_SEED_CBC_SHA (0x96) WEAK 128
* TLS_DHE_RSA_WITH_SEED_CBC_SHA (0x9a) DH 2048 bits FS WEAK 128
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xc013) ECDH secp521r1 (eq. 15360 bits RSA) FS WEAK 128
* TLS_RSA_WITH_AES_128_CBC_SHA256 (0x3c) WEAK 128
* TLS_DHE_RSA_WITH_AES_128_CBC_SHA256 (0x67) DH 2048 bits FS WEAK 128
* TLS_RSA_WITH_AES_128_GCM_SHA256 (0x9c) WEAK 128
* TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (0x9e) DH 2048 bits FS 128
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (0xc027) ECDH secp521r1 (eq. 15360 bits RSA) FS WEAK 128
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f) ECDH secp521r1 (eq. 15360 bits RSA) FS 128
* TLS_RSA_WITH_RC4_128_MD5 (0x4) INSECURE 128
* TLS_RSA_WITH_RC4_128_SHA (0x5) INSECURE 128
* TLS_RSA_WITH_IDEA_CBC_SHA (0x7) WEAK 128
* TLS_ECDHE_RSA_WITH_RC4_128_SHA (0xc011) ECDH secp521r1 (eq. 15360 bits RSA) FS INSECURE 128
* TLS_RSA_WITH_AES_256_CBC_SHA (0x35) WEAK
* TLS_DHE_RSA_WITH_AES_256_CBC_SHA (0x39) DH 2048 bits FS WEAK 256
* TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (0x84) WEAK 256
* TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA (0x88) DH 2048 bits FS WEAK 256
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014) ECDH secp521r1 (eq. 15360 bits RSA) FS WEAK 256
* TLS_RSA_WITH_AES_256_CBC_SHA256 (0x3d) WEAK 256
* TLS_DHE_RSA_WITH_AES_256_CBC_SHA256 (0x6b) DH 2048 bits FS WEAK 256
* TLS_RSA_WITH_AES_256_GCM_SHA384 (0x9d) WEAK 256
* TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x9f) DH 2048 bits FS 256    `iAeris 選擇支援`
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (0xc028) ECDH secp521r1 (eq. 15360 bits RSA) FS WEAK 256
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030) ECDH secp521r1 (eq. 15360 bits RSA) FS 256 `iAeris 選擇支援`

----------------------------
```
* Date : 2020/06/10		svn39/--
* Author : Joe
* Modification :

1. Add TLS1.2 support
   - Based on (MTK SDK 4.3.1) & (mbedtls 2.1.0)
   - Add two tls modules. (supported by sites of xsize & sgs.)
     TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x9f)
     TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)
   - change watch dog timer from 6sec too 20sec.
   - change head size from 160k to 190k.
2. Excution time comparison
   - http :   1sec
   - https(tls1.0/1.1): 4sec
   - https(tls1.2-DHE_RSA) : 11sec
   - https(tls1.2-ECDHE_RSA) : 16sec
```
----------------------------

2024-03-14
### SNI Issue
* 發現 xsize.net 無法以https 進行資料傳送 !!
```
server hello
ssl->f_recv(_timeout)() returned 5 (-0xfffffffb)
ssl->f_recv(_timeout)() returned 2 (-0xfffffffe)
mbedtls_ssl_handshake() failed, ret:-0x7780.
```

* 問題解析：
```
https 通訊必須加 SNI (Server Name Indication) 
- xsize & AWS 必須有 SNI 才能工作:  

由於 mqtt_tls ssl_connect()是OK的，兩相比較發現 mqtt_tls_connect 有加 SNI 的程序：
mbedtls_ssl_handshake 前必須加 
mbedtls_ssl_set_hostname(&ssl->ssl_ctx, host);

在 HTTPS 中，先發生 TLS 交握，然後才能開始 HTTP 對話（HTTPS 仍使用 HTTP，它只是對 HTTP 訊息進行加密）。如果沒有SNI，用戶端將無法向伺服器表明想與之通訊的主機名稱。這樣一來，伺服器可能為錯誤的主機名稱產生 SSL 憑證。如果 SSL 憑證上的名稱與用戶端嘗試連線的名稱不匹配，則用戶端瀏覽器將傳回錯誤，且通常會終止連線。

SNI 會將網域名稱新增至 TLS 交握程序，以便 TLS 程序連線正確的網域名稱並接收正確的 SSL 憑證，從而使 TLS 交握的其餘部分能正常進行。

具體來說，SNI 在 Client Hello 訊息或 TLS 交握的第一個步驟中包含主機名稱。
```


2024-03-14 

發現TLS1.2 原新增兩種加密套件，僅僅剩下 0xc030，另外開始支援 TLS1.3。 
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030) ECDH secp521r1 (eq. 15360 bits RSA) FS 256

Date : 2024/08/6 svn209
* iAeris2 WizFi360模組支援 https 通訊
* 測試 
* https://www.xsize.net/dp1/interfaces/air/monitor_data_collect0001.php

2024-08-09 實驗 message
```
043684  16:56:03.005   CIPSTART=4,"SSL","www.xsize.net",443
043685  16:56:05.777     Ret=1 WifiRxBuf_cnt=58
043686  16:56:05.798   <<AT+CIPSTART=4,"SSL","www.xsize.net",443
043687  16:56:05.804   4,CONNECT
043688  16:56:05.808   OK
043689  16:56:05.824   --WIFI NET-OPEN 4 conn=0x10
043690  16:56:05.834   SSL Connect OK!! 1
043691  16:56:05.875   Upload Head(190):POST /dp1/interfaces/air/monitor_data_collect0001.php HTTP/1.1
043692  16:56:05.887   Host: www.xsize.net 
043693  16:56:05.894   Accept: */*
043694  16:56:05.906   Content-Length: 316
043695  16:56:05.931   Content-Type: application/x-www-form-urlencoded
043696  16:56:05.941   Connection: close
043697  16:56:05.956   >>AT+CIPSEND=4,190
043698  16:56:05.968     Ret=1 WifiRxBuf_cnt=26
043699  16:56:05.978   <<AT+CIPSEND=4,190
043700  16:56:05.982   OK
043701  16:56:05.990   > ..Send..
043702  16:56:05.993   <<
043703  16:56:06.000   Recv 190 bytes
043704  16:56:06.007   SEND OK
043705  16:56:06.012     Ret=1
043706  16:56:06.184   Upload Data(316):application_id=&application_check=&serial_no=&hcho=0.000&co=0&co2=752&rh=58.4&temperature=24.6&pm10=0.0&pm2p5=2.0&fungi=0.0&tvoc=0&sno=0000294800001030&blesno=000000000000&firmware_ver=K.25.207&longitude=&latitude=&local_ip=172.24.20.45&create_date_time=2024-5-23 16:55:0&pid=iAeris22&dbg0=0.0&dbg1=0&dbg2=0&dbg3=-37
043707  16:56:06.194   >>AT+CIPSEND=4,316
043708  16:56:06.208     Ret=1 WifiRxBuf_cnt=26
043709  16:56:06.218   <<AT+CIPSEND=4,316
043710  16:56:06.222   OK
043711  16:56:06.229   > ..Send..
043712  16:56:06.232   <<
043713  16:56:06.239   Recv 316 bytes
043714  16:56:06.246   SEND OK
043715  16:56:06.251     Ret=1
043716  16:56:06.260   Upload Data OK
043717  16:56:06.263   <<
043718  16:56:06.286   +IPD,4,718,172.67.170.117,443:HTTP/1.1 200 OK
043719  16:56:06.305   Date: Thu, 23 May 2024 08:56:06 GMT
043720  16:56:06.326   Content-Type: text/html; charset=UTF-8
043721  16:56:06.341   Transfer-Encoding: chunked
043722  16:56:06.351   Connection: close
043723  16:56:06.365   X-Powered-By: PHP/5.4.16
043724  16:56:06.379   CF-Cache-Status: DYNAMIC
043725  16:56:06.509   e7YAl08Ljxx47xqiluCAcc4Q1Vuv2%2F%2FBFM3eO"}],"group":"cf-nel","max_age":604800}
043726  16:56:06.543   NEL: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
043727  16:56:06.554   Server: cloudflare
043728  16:56:06.570   CF-RAY: 8883d4b0e888045d-HKG
043729  16:56:06.586   alt-svc: h3=":443"; ma=86400
043730  16:56:06.590   76
043731  16:56:06.608   You come from 60.250.191.43<br>
043732  16:56:06.620   Record insert OK<br>
043733  16:56:06.624   PM10=
043734  16:56:06.628   PM2.5=
043735  16:56:06.634   PMOffset=0
043736  16:56:06.640   PMGain=0
043737  16:56:06.644   CO=0
043738  16:56:06.648   NO2=0
043739  16:56:06.652   SO2=0
043740  16:56:06.657   O3=0
043741  16:56:06.678   WiFi_ConnExceptionRst Timer=1748 sec
043742  16:56:06.687   get IPD message
043743  16:56:06.694     +IPD NO: 1
043744  16:56:06.725   -->> len:718 from channel[4] ip:[172.67.170.117] port:[443]
043745  16:56:06.733   HTTP/1.1 200 OK
043746  16:56:06.752   Date: Thu, 23 May 2024 08:56:06 GMT
043747  16:56:06.774   Content-Type: text/html; charset=UTF-8
043748  16:56:06.789   Transfer-Encoding: chunked
043749  16:56:06.799   Connection: close
043750  16:56:06.812   X-Powered-By: PHP/5.4.16
043751  16:56:06.826   CF-Cache-Status: DYNAMIC
043752  16:56:06.956   e7YAl08Ljxx47xqiluCAcc4Q1Vuv2%2F%2FBFM3eO"}],"group":"cf-nel","max_age":604800}
043753  16:56:06.991   NEL: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
043754  16:56:07.002   Server: cloudflare
043755  16:56:07.018   CF-RAY: 8883d4b0e888045d-HKG
043756  16:56:07.033   alt-svc: h3=":443"; ma=86400
043757  16:56:07.037   76
043758  16:56:07.055   You come from 60.250.191.43<br>
043759  16:56:07.067   Record insert OK<br>
043760  16:56:07.071   PM10=
043761  16:56:07.076   PM2.5=
043762  16:56:07.082   PMOffset=0
043763  16:56:07.088   PMGain=0
043764  16:56:07.091   CO=0
043765  16:56:07.095   NO2=0
043766  16:56:07.099   SO2=0
043767  16:56:07.103   O3=0
043768  16:56:07.130   Task_WiFi get +IPD_Flag:ID=0;Len=0 status=0
043769  16:56:07.138   HTTP/1.1 200 OK
043770  16:56:07.158   Date: Thu, 23 May 2024 08:56:06 GMT
043771  16:56:07.179   Content-Type: text/html; charset=UTF-8
043772  16:56:07.193   Transfer-Encoding: chunked
043773  16:56:07.204   Connection: close
043774  16:56:07.218   X-Powered-By: PHP/5.4.16
043775  16:56:07.232   CF-Cache-Status: DYNAMIC<<
043776  16:56:07.263   Report-To: {"endpoints":[{"url":"https:\/\/a.nel.cloudfla
043777  16:56:07.365   Oze7YAl0+8Ljxx47xqiluCAcc4Q1Vuv2%2F%2FBFM3eO"}],"group":"cf-nel","maxIP_age":604800}
043778  16:56:07.401   NEL: {"success_fraction":0,"report_to":"cf-D,nel","max_age":604800}
043779  16:56:07.412   Server: cloudflare
043780  16:56:07.428   CF-RAY: 8883d44b0e888045d-HKG
043781  16:56:07.444   alt-svc: h3=":443"; ma=86400
043782  16:56:07.448   76
043783  16:56:07.467   You ,5come from 60.250.191.43<br>
043784  16:56:07.478   Record insert OK<br>
043785  16:56:07.482   PM10=
043786  16:56:07.486   ,
043787  16:56:07.489   PM2.5=
043788  16:56:07.494   PMOffset=0
043789  16:56:07.500   PMGain=0
043790  16:56:07.504   CO=0
043791  16:56:07.508   NO2=0
043792  16:56:07.512   SO2=0
043793  16:56:07.517   O3=170
043794  16:56:07.528   ---------------
043795  16:56:07.539   ipd.status=0 id=0
043796  16:56:07.549   SEND HEADER:[1920]
043797  16:56:07.564   Error: ID#0 already Close!!
043798  16:56:07.572   >>AT+CIPCLOSE=0
043799  16:56:07.579   ]dy Close!!
043800  16:56:07.599   WiFi_ConnExceptionRst Timer=1800 sec
043801  16:56:07.607   get IPD message
043802  16:56:07.614     +IPD NO: 1
043803  16:56:07.645   -->> len:5 from channel[4] ip:[172.67.170.117] port:[443]
043804  16:56:07.647   0
043805  16:56:07.663     Ret=2 WifiRxBuf_cnt=42
043806  16:56:07.668   <<4,CLOSED
043807  16:56:07.677   AT+CIPCLOSE=0
043808  16:56:07.681   UNLINK
043809  16:56:07.686   ERROR
043810  16:56:07.704   --WIFI NET-CLOSED 4 conn=0x00
043811  16:56:07.727   Task_WiFi get +IPD_Flag:ID=0;Len=0 status=0
043812  16:56:07.730   0
043813  16:56:07.740   ---------------
043814  16:56:07.749   ipd.status=0 id=0
043815  16:56:07.760   SEND HEADER:[190]
043816  16:56:07.774   Error: ID#0 already Close!!
043817  16:56:07.783   >>AT+CIPCLOSE=0
043818  16:56:07.796     Ret=2 WifiRxBuf_cnt=32
043819  16:56:07.804   <<AT+CIPCLOSE=0
043820  16:56:07.809   UNLINK
043821  16:56:07.815   ERROR
043822  16:56:11.760   >>AT+CIPCLOSE=4
043823  16:56:16.685     Ret=0 WifiRxBuf_cnt=32
043824  16:56:16.693   <<AT+CIPCLOSE=4
043825  16:56:16.697   UNLINK
043826  16:56:16.704   ERROR
043827  16:56:16.712   >>AT+CIPMUX=1
043828  16:56:16.726     Ret=1 WifiRxBuf_cnt=19
043829  16:56:16.733   <<AT+CIPMUX=1
043830  16:56:16.737   OK
043831  16:56:16.748   >>AT+CIPSERVER=1,80
043832  16:56:16.762     Ret=1 WifiRxBuf_cnt=25
043833  16:56:16.772   <<AT+CIPSERVER=1,80
```

2026-04-27 

發現 www.xsize.net TLS.2 的 0xc030 也不支援了
* TLS 1.3
-----------------------------------------------------------
* TLS_AES_128_GCM_SHA256 (0x1301) ECDH x25519 (eq. 3072 bits RSA) FS                                                  128
* TLS_AES_256_GCM_SHA384 (0x1302) ECDH x25519 (eq. 3072 bits RSA) FS                                                  256
* TLS_CHACHA20_POLY1305_SHA256 (0x1303) ECDH x25519 (eq. 3072 bits RSA) FS                                    256
* TLS1.2
-----------------------------------------------------------
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (0xc02b) ECDH x25519 (eq. 3072 bits RSA) FS             128
* TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca9) ECDH x25519 (eq. 3072 bits RSA) FS  256<sup>P</sup>
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (0xc009) ECDH x25519 (eq. 3072 bits RSA) FS WEAK           128
* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (0xc02c) ECDH x25519 (eq. 3072 bits RSA) FS                 256
* TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (0xc00a) ECDH x25519 (eq. 3072 bits RSA) FS WEAK               256
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 (0xc023) ECDH x25519 (eq. 3072 bits RSA) FS WEAK        128
* TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 (0xc024) ECDH x25519 (eq. 3072 bits RSA) FS WEAK        256
-----------------------------------------------------------




===========================================

這兩組加密演算法，以下是根據 Cloudflare 目前的技術佈告與產業安全趨勢的詳細分析：

### 1. TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)
* **目前的狀態：非常安全。**
* **移除風險：極低。**
* **分析：**
    這是一組現代化的加密套件，具備 **ECDHE（橢圓曲線暫時金鑰交換）**，能提供「完美前向安全性（PFS）」。它是目前 TLS 1.2 連線中的主流推薦配置。Cloudflare 以及大多數瀏覽器在未來幾年內都沒有移除此算法的計畫。

### 2. TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x9f)
* **目前的狀態：支援中，但已趨向邊緣化。**
* **移除風險：中等（長期來看）。**
* **分析：**
    這組算法使用 **DHE（有限域 Diffie-Hellman）**。雖然它也提供前向安全性，但在效能與安全性平衡上，業界普遍偏向使用前述的 ECDHE（橢圓曲線）。
    * **效能問題：** DHE 的運算成本比 ECDHE 高，對於嵌入式設備或大量連線的伺服器來說負擔較重。
    * **安全性考量：** 如果伺服器端使用的 DH 參數（dhparams）小於 2048-bit，會被認為是不安全的（易受 Logjam 攻擊）。
    * **Cloudflare 的態度：** 雖然 Cloudflare 目前仍支援 DHE 演算法以確保相容性，但許多現代瀏覽器（如 Chrome/Firefox）已經開始限縮 DHE 的使用。如果 Cloudflare 未來為了精簡加密清單或提升效能，DHE 類型的算法通常會比 ECDHE 更早被列入移除清單。

---

### 未來變動預測與建議

Cloudflare 近期公佈的 **2025-2026 汰換計畫** 主要針對的是：
1.  **不具備前向安全性（No Forward Secrecy）**的套件（如：`TLS_RSA_WITH_...`）。
2.  使用 **CBC 模式**的套件（如：`AES_256_CBC`）。

由於您提供的這兩組都使用了 **GCM (Galois/Counter Mode)** 且都具備 **前向安全性 (DHE/ECDHE)**，因此它們**不在**第一波強制移除的清單中。

#### 針對系統開發的建議：
* **首選 ECDHE (0xc030)：** 若您的設備支援，請盡可能將 `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384` 設為預設或最高優先權。這在相容性與安全性之間達到了最佳平衡。
* **監測 Cloudflare 公告：** 如果您使用的是 Cloudflare 的免費或專業（Pro）方案，系統會自動套用最新的安全策略。建議定期檢查 Cloudflare Dashboard 中的 `SSL/TLS` 流量分析，確認是否有設備仍在回退（Fallback）到較舊的演算法。
* **準備轉向 TLS 1.3：** 若未來硬體更新，建議考慮支援 TLS 1.3，因為 TLS 1.3 大幅簡化了加密套件，僅保留最安全的組合（如 `TLS_AES_256_GCM_SHA384`），能徹底避免這類演算法被移除的隱憂。

**總結：** 短期內（1-2年）這兩組演算法在 Cloudflare 上都是安全的；長期而言，`0xc030` 比 `0x9f` 更加穩固。



------------------------------

2026-04-29 
DearSoft 回應：

看文件與實際測試 cloudflare api 了，business 也沒有此權限，需 enterprise 或 加購方案，而且也不支援加回 DHE (0x9f)了，只能加回其中 ECDHE(0xc030) 那條