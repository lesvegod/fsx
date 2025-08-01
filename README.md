# FTP Script eXecutor (fsx)

一個用於管理 FTP 處理作業的命令列工具，可以預建立的指令腳本檔進行管理。  
讓你方便批次執行 FTP 指令，以達自動上傳、更新、刪除檔案或建立、刪除目錄等管理動作。
亦可利用整批上傳模式，一次性將本地目錄下的所有檔案上傳至遠端 FTP 伺服器指定目錄，或是打開 mirror 模式，使遠端目錄與本地目錄進行單向同步。

如果你的作業環境為 Windows，同時因為任何原因無法或不願使用其他 FTP 工具(如 FileZilla、WinSCP、lftp 等)，即可多多利用本工具進行如排程、 CI / CD、檔案管理等日常作業。

本工具無任何使用上的限制，可自由傳播及使用。

---

## 功能特色

- 支援 FTP / FTPS (Explicit / Implicit) 連線
- 支援多種 FTP 腳本指令 (mput、mdele、mkdir 等)
- 支援指令腳本執行模式
- 支援萬用字元（Wildcard * ?）批次操作
- 支援模擬執行，避免誤操作
- 支援單指令整批上傳，亦可進行單向同步
- 內建簡易保護機制，降低誤刪誤傳風險
- 支援 Log 記錄，便於掌握執行歷程

---
## 運行需求

- .NET 8 SDK / Runtime 或以上版本
- 基礎 FTP / DOS 指令概念(需撰寫腳本)
- 具備 FTP 伺服器相關資訊與存取權

---
## 使用方式   

安裝完成後，即可於命令列中輸入 `fsx` 來啟動工具，可先輸入
```
fsx --help, -h
```
取得指令說明與使用方式，或輸入
```
fsx version, --version, -v
```
確認當前使用版本。

---

## 指令說明
1. `fsx` : 腳本執行模式
1. `fsx push` : 整批上傳模式，將執行指令所在本地目錄上傳至遠端指定目錄 (新增新檔、覆蓋已存在檔案)

## 參數介紹
### 通用參數
1. `--passive, -p` :    啟用被動模式 (PASV)，適用於防火牆或 NAT 環境。
1. `--ssl, -s` :    啟用 FTPS 模式，支援 FTP Secure (FTPS) 連線，需確保 FTP 伺服器支援 FTPS。
1. `--ssl-implicit` : 使用 Implicit 加密協商模式(需配合 --ssl, -s 使用)，未開啟預設使用 Explicit 模式。
1. `--quiet, -q` :   啟用靜默模式，關閉所有運行訊息。
1. `--log, -l` : 可將執行過程記錄於指定檔案中。

### `fsx` 可用參數
1. `--file, -f` : 指定要執行的腳本檔案路徑與名稱，若未指定則預設使用當前目錄下的 `run.script`。
1. `--mock, -m` : 啟用模擬模式，在此模式下，若腳本指令會影響檔案內容或架構，將不會實際執行，比如：
	- `put`
	- `mput`
	- `dele`
	- `mdele`
	- `mkdir`
	- `rmdir` 
	
### `fsx push` 可用參數
`push` 指令
1. `--mirror` : 啟用同步模式，需與 `fsx push` 配合使用，可以同步模式將本地目錄同步至遠端指定目錄 (新增新檔、覆蓋已存在檔案、刪除額外檔案，使用時請確認風險！)
1. `--port <port number>` : 設置 FTP 連接埠，未設置預設使用 21 Port。
1. `--user <account>` : 設置 FTP 帳號，未設置預設使用 "anonymous"。
1. `--pass <password>` : 設置 FTP 密碼，未設置預設使用 "anonymous@example.com"

---
### 腳本介紹
本工具採用腳本運行，故須先撰寫指令腳本，若僅執行
```
fsx
```
工具將於執行所在目錄下，找尋預設腳本 `run.script`，若不存在則會顯示錯誤訊息。也可以輸入
```
fsx --file, -f <your_script_path&name>
```
來指定自訂腳本檔案。
腳本執行過程中，請掌握本地或遠端工作目錄的切換，因為所有檔案操作指令都會依據當前工作目錄進行。
本地端起始為執行所在目錄，遠端端起始為 FTP 伺服器的預設根目錄。
腳本檔案為一般文字檔案，僅須注意檔案儲存格式必須是UTF-8，否則若檔案內有非 ASCII 文字可能造成顯示錯誤。
而在腳本內，空行與註解行（以 `#` 開頭）將被忽略，其他指令則會依照上下順序執行。

### __特別說明__
本工具腳本支援指令基本參照一般 FTP 指令，但有些指令會有額外的限制或特性，且部分指令屬特殊功能。
在此特別說明 `guard` 指令。
`guard` 指令用於設定安全目錄，當設定後，所有影響檔案的指令（如 `put`, `mput`, `dele`, `mdele`, `mkdir`, `rmdir` 等）都會檢查當前工作目錄是否與安全目錄相符。
若不相符，則會顯示錯誤訊息並中斷指令執行。這樣可以避免在錯誤的目錄下進行檔案操作，降低誤操作風險。
假設: 
- 執行需求為將 test.cs 檔案上傳至 `/folder` 目錄下。
- 腳本內設定了 `guard /folder`。
- 同時設定指令 `cd /other`，將會把遠端當前工作目錄切換至 other 目錄。
- 此時若設置指令為 `put test.cs`，由於已經設定了遠端安全目錄，因此會比對遠端安全目錄與當前遠端工作目錄是否相符，若不符則直接中斷腳本運行。
- 設置了遠端安全目錄後，若需要變更遠端安全目錄，需再次使用 `guard` 指令來覆蓋原有設定。
- `unguard` 指令可用來取消遠端安全目錄設定。

同樣亦提供 `lguard`、`lunguard` 指令用於設定／取消本地安全目錄設置。

以下為腳本範例
``` 
# 設定 FTP 伺服器資訊
server 192.168.1.1
# 設定 FTP 連接埠資訊
port 9527
# 設定 FTP 帳號資訊
user test
# 設定 FTP 密碼資訊
pass myPassword
# 啟動 FTP 連接
open

# 設定遠端安全目錄為 /folder
guard /folder
# 切換遠端工作目錄至 /folder
cd /folder
# 顯示 /folder 目錄下的檔案與目錄
ls

# 設定本地安全目錄為 C:\project\output\
lguard C:\project\output
# 切換本地工作目錄至 C:\project\output
lcd C:\project\output
# 顯示本地工作目錄內容
lls

# 將 C:\project\output\test.cs 上傳至遠端 /folder 目錄下
put test.cs

# 結束腳本執行
bye
```
---

## 腳本支援指令
|指令|必需|說明|範例|支援版本|
|:-:|:-:|:--|:--|:-:|
|server|✔|設定 FTP 伺服器資訊|`server` test.example.com<br>`server` 192.168.20.12|V1.0.1|
|port|-|設定 FTP 連接埠資訊，若未設定預設使用 21| `port` 55688 |V1.0.1|
|user|-|設定 FTP 帳號資訊，若未設定預設使用 anonymous |`user` test|V1.0.1|
|pass|-|設定 FTP 密碼資訊，若未設定預設使用 anonymous@example.com|`pass` mypass|V1.0.1|
|open|✔|依 FTP 設定資訊開始進行連接|`open`|V1.0.1|
|guard|-|設定遠端安全目錄，若已設定遠端安全目錄，則後續影響檔案的指令，都會檢查遠端當前工作目錄是否與遠端安全目錄設定相符，若需變更遠端安全目錄可直接設定覆蓋|`guard` /folder/test|V1.0.1|
|unguard|-|取消遠端安全目錄設定|`unguard` /folder/test|V1.0.2|
|ls|-|列出 FTP 工作目錄內容|`ls`|V1.0.1|
|cd|-|切換 FTP 工作目錄|`cd` folder<br>由當前目錄切換至其內 folder 目錄<br>`cd` ..<br>由當前目錄切換至上層目錄<br>`cd` /folder/test2<br>由當前目錄切換至指定層級目錄 |V1.0.1|
|put|-|由本地工作目錄上傳指定單一檔案至 FTP 工作目錄，不支援 WildCard|`put` test.dll |V1.0.1|
|mput|-|由本地工作目錄上傳檔案至 FTP 工作目錄，支援 * ? WildCard，或多筆指定V1.0.1|
|dele|-|於 FTP 工作目錄移除指定單一檔案|`dele` test.dll|V1.0.1|
|mdele|-|於 FTP 工作目錄移除檔案，支援 * ? WildCard，或多筆指定V1.0.1|
|mkdir|-|新增 FTP 目錄，僅可在目前工作目錄下建立目錄|`mkdir` output|V1.0.1|
|rmdir|-|移除 FTP 目錄，僅可在目前工作目錄下移除目錄，且支援遞迴，將移除指定目錄中所有檔案與目錄。|`rmdir` output|V1.0.1|
|lls|-|列出本地工作目錄內容|`lls`|V1.0.1|
|lcd|-|切換本地工作目錄|`lcd` output<br>由當前目錄切換至其內 output 目錄<br>`cd` ..<br>由當前目錄切換至上層目錄<br>`cd` E:\Stage\output<br>由當前目錄切換至指定層級目錄|V1.0.1|
|lguard|-|設定本地安全目錄，若已設定安全目錄，則後續影響檔案的指令，都會檢查本地當前工作目錄是否與本地安全目錄設定相符，若需變更本地安全目錄可直接設定覆蓋|`guard` /folder/test|V1.0.2|
|lunguard|-|取消本地安全目錄設定|`unguard` /folder/test|V1.0.2|
|bye|-|結束腳本操作|`bye`|V1.0.1|

---
## CHANGELOG
### V1.0.1
  - 初始版本
  - 支援基本 FTP 指令
  - 支援 WildCard * ? 批次操作
  - 支援遠端安全目錄設定
  - 支援模擬執行模式
### V1.0.2
  - 調整 `protect` 腳本指令 為 `guard` 腳本指令
  - 新增 `unguard`、`lguard`、`lguard` 腳本指令，支援本地設置安全目錄與取消
  - 新增 Log 機制，`fsx --log` 可將執行過程記錄於指定檔案中
### V1.0.3
  - 新增 FTPS 支援
