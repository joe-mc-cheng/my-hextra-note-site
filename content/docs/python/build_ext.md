---
title: "python build_ext 筆記"
date: 2026-06-04
weight: 2
---

# 錯誤訊息 Cannot convert Unicode string to 'str' implicitly 
是 Cython 在處理 -> str 型別提示（Type Hint）時最經典的底層衝突Bug。 [1, 2] 

## 🚀 最快的解決方法：把 -> str 改成 -> unicode
最直截了當的修復方式，就是將函式回傳值的型別提示從 str 改為 unicode：

# 修正後：將 -> str 改為 -> unicodedef _dict_to_modbus_line(data: dict) -> unicode:
    return f"RTU,{com},{baud},{mb_id}"

------------------------------
## 🔍 為什麼會發生這個錯誤？
這個問題並非你的程式碼邏輯有錯，而是 Cython 在將 Python 編譯為 C 語言時，對 str 關鍵字的定義存在歷史包袱：

   1. f-string 的本質：在 Python 3 中，所有的 f-string 執行後的底層本質都是 Unicode 字串（即大於 8-bit 的字元編碼）。 [1, 2, 3] 
   2. Cython 對 str 的誤解：為了相容舊時代的 Python 2，Cython 的編譯器核心有時仍會把型別提示中的 str 當作 C 語言的 bytes（傳統由 8-bit 組成的 char 陣列）。 [1, 2, 4] 
   3. 編譯器拒絕自動轉換：當你宣告回傳值是 str（Cython 誤判為 bytes），但實際上 f-string 回傳了 Unicode 時，Cython 為了安全起見，不願意幫你做隱式（implicitly）轉換，因而拋出該錯誤。 [1, 2, 5] 

------------------------------
## 🛠️ 替代解決方案
如果你不希望在 Python 程式碼中看見 unicode 這種舊關鍵字，你也可以選擇以下兩種等價的做法：
## 方法 A：移除型別提示（最省心）
純 Python 的型別提示（Type Hints）主要用於編輯器靜態檢查。如果直接移除 -> str，Cython 在編譯時就不會產生型別衝突的誤判：

def _dict_to_modbus_line(data: dict):
    return f"RTU,{com},{baud},{mb_id}"

## 方法 B：在編譯設定（setup.py）中指定語言等級
如果你使用的是舊版 Cython，強烈建議在負責打包編譯的 setup.py 中，明確指定使用 Python 3 的字串語法（3str）。請在 setup.py 中加入 language_level="3" 或 binding=True 參數： [1, 2] 

from setuptools import setupfrom Cython.Build import cythonize

setup(
    ext_modules = cythonize(
        "your_file.pyx", 
        compiler_directives={'language_level': "3"} # 強制將 str 對齊為 Python 3 的 Unicode
    )
)

[1] [https://github.com](https://github.com/cython/cython/issues/4829)
[2] [https://github.com](https://github.com/cython/cython/issues/4829)
[3] [https://forum.pycom.io](https://forum.pycom.io/topic/1192/typeerror-can-t-convert-str-object-to-str-implicitly)
[4] [https://cython.readthedocs.io](https://cython.readthedocs.io/en/latest/src/tutorial/strings.html)
[5] [https://cython.readthedocs.io](https://cython.readthedocs.io/en/latest/src/tutorial/strings.html)


-----------------
# TypeError: get_sys_logs() takes no keyword arguments
以 python 模組可以執行,但是 build_ext 變成 library 之後會有錯誤訊息 TypeError: get_sys_logs() takes no keyword arguments

這是一個非常經典且進階的 Cython 編譯優化問題。 [1, 2] 
當你使用 build_ext（通常是透過 Cython）將 Python 原始碼打包編譯成 C 語言的二進位擴充模組（.so 或 .pyd 檔）時，核心原因在於：Cython 對於「只有 0 或 1 個參數」的函數啟用了底層 C 語言的優化（METH_O 模式），這會導致該函數在 C 底層「拒絕接收任何關鍵字參數（Keyword Arguments）」。 [1, 2] 
然而，Flask 在解析路由時（例如 <log_type>），必定會以關鍵字參數的方式將 log_type="xxx" 傳入你的 get_sys_logs 函數。這就導致原生 Python 執行沒問題，但編譯成 library 後立刻崩潰。 [2, 3] 
------------------------------
## 🛠️ 解決方案
要解決這個問題，你有以下三種最常用的解決方法：
## 方法一：在 Python 原始碼中加入編譯器指示（最推薦，免改 setup.py）
你可以在包含該 API 的 .py（或 .pyx）檔案的最頂端，加上 Cython 的編譯指令，強制讓函數允許關鍵字參數： [2] 
```
# cython: always_allow_keywords=Truefrom flask import Flask, jsonify
app = Flask(__name__)

@app.route('/api/admin/logs/<log_type>', methods=['GET'])
def get_sys_logs(log_type):
    return jsonify({"status": "success", "type": log_type})
```
註：# cython: always_allow_keywords=True 必須寫在檔案的第一行才有效。
## 方法二：在 setup.py 中全域開啟此設定
如果你有非常多個檔案都要打包，可以直接在你的編譯腳本 setup.py 中，將 always_allow_keywords 設為 True： [2, 3] 
```
from setuptools import setupfrom Cython.Build import cythonize

setup(
    ext_modules = cythonize(
        "your_app_module.py", 
        compiler_directives={'always_allow_keywords': True} # 👈 關鍵是加上這一行
    )
)
```
## 方法三：修改函數特徵以避開 Cython 的單參數優化（不改設定的髒方法）
如果你不想動到編譯設定，可以透過修改函數參數，強迫 Cython 把他當成「多參數函數」處理。例如加上不影響功能的 **kwargs： [2] 
```
@app.route('/api/admin/logs/<log_type>', methods=['GET'])def get_sys_logs(log_type, **kwargs):  # 👈 加上 **kwargs 強制解除 C 底層優化
    return f"正在查詢 {log_type} 日誌"
```
------------------------------
## 🔍 深入原因（為什麼會這樣？）

   1. Python 原生行為：在純 Python 中，def get_sys_logs(log_type): 同時支援位置參數 get_sys_logs('auth') 與關鍵字參數 get_sys_logs(log_type='auth')。 [2] 
   2. Cython 的底層優化：當 Cython 看到你的函數只有一個參數 log_type 時，為了追求極致的效能，它會在 C 底層使用 Python C API 的 METH_O 呼叫約定。這種約定只允許透過位置傳參，不允許關鍵字傳參，藉此省去比對參數名稱的 CPU 時間。 [1, 2] 
   3. Flask 的運作機制：Flask 的路由系統在匹配到網址中的變數後，會固定用類似 get_sys_logs(**{"log_type": "system"}) 的關鍵字字典展開方式來呼叫你的 view function。當這兩個碰撞在一起時，編譯後的 C 函數不認得關鍵字，就拋出了 TypeError。 [1, 2] 

[1] [https://github.com](https://github.com/cython/cython/issues/3090)
[2] [https://github.com](https://github.com/bottlepy/bottle/issues/453)
[3] [https://stackoverflow.com](https://stackoverflow.com/questions/50932527/how-do-you-compile-a-flask-app-with-cython)
