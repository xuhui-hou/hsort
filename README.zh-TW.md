# HSORT

**[English（README.md）](README.md)** | **[日本語（README.ja.md）](README.ja.md)** | **[简体中文（README.zh.md）](README.zh.md)** — 本檔案為 **台灣繁體中文** 說明（用語以台灣為準）。

高效能排序／合併 **Python C 延伸模組**，底層為 C 引擎。透過內部記憶體池快速排序定長、變長與 CSV 檔案，並支援將已排序輸入合併為單一輸出流。

- **排序** — 按文字或數值鍵升序/降序排列記錄  
- **合併** — 將多個已排序檔案合併為一條流  

## 產品要點

- 透過 **pip** 從 PyPI 安裝  
- **`hsort`** 命令列與 **`import hsort`** Python API  
- 支援 **Windows、Linux、macOS**  
- Unix 風格 **`-` / `--`** 選項  

## 排序模式

**記憶體排序** — 當可用記憶體足以容納資料集時，排序過程可不使用暫存檔案。

**外排序** — 當記憶體不足時，資料被分塊排序，並藉助暫存檔案完成合併。

若省略 **`-W`**，引擎會根據輸入大小估算記憶體；若分配失敗，將退回外排序。

## 功能概要

- 可設定記憶體上限（`-W`）；省略時自動估算  
- 定長、變長（文字/二進位）及 **CSV**  
- 標準輸入/輸出；日誌輸出到標準錯誤  
- 多鍵排序；穩定排序（`-S`）；唯一輸出（`-U`）  
- 無效資料列可略過或寫入錯誤檔案（`-E`）  
- ASCII 與數值鍵模式；支援大檔案  
- 跨平台  

## 安裝

```bash
pip install hsort
```

Test PyPI（如適用）：

```bash
pip install --index-url https://test.pypi.org/simple/ hsort
```

驗證安裝：

```bash
hsort --version
hsort --help
```

## 快速上手

**CSV（含標題列，依第 1 欄排序）**

```bash
hsort -C -H -K1 -O output.csv input.csv
```

**定長（64 位元組一筆，鍵為位元組 0–19）**

```bash
hsort -L64 -K0,20 -O output.dat input.dat
```

**變長**

```bash
hsort -K0,10 -O output.dat input.dat
```

## 命令列用法

```text
hsort [options] [input files...]
```

選項區分大小寫（`-C` ≠ `-c`）。短選項與長選項等價（`-C` / `--csv`）。

權威參數說明請執行：

```bash
hsort --help
```

### 參數一覽

| 短選項 | 長選項 | 說明 |
| --- | --- | --- |
| `-h` | `--help` | 顯示說明 |
| `-V` | `--version` | 顯示版本與授權資訊 |
| `-C` | `--csv` | 輸入為 CSV |
| `-H` | `--header` | CSV 第一列作為標題列 |
| `-S` | `--stable` | 穩定排序 |
| `-U` | `--unique` | 唯一輸出（每個鍵保留首條記錄） |
| `-L BYTES` | `--record-length BYTES` | 定長記錄；每筆記錄長度 **1～640KB** |
| `-K KEY_SPEC` | `--key` | 排序鍵（可重複）。**定長／變長：** `起始[,長度][n][a\|d]` — 起始為從 0 起的位元組位移；長度可省略；`n` 為數值排序（預設 ASCII）；`a` 升序（預設）、`d` 降序。**CSV：** `欄位編號[n][a\|d]` — 欄位編號從 1 起算 |
| `-A` | `--all-asc` | 整行升序。與 `-K` 同時時以 `-K` 為準；與 `-R` 同時時以後寫的選項為準 |
| `-R` | `--all-desc` | 整行降序。與 `-K` 同時時以 `-K` 為準；與 `-A` 同時時以後寫的選項為準 |
| `-P CODE` | `--newline CODE` | 變長/CSV 換行：`\n`、`\r`、`\r\n`（預設 `\n`） |
| `-D CHAR` | `--delimiter CHAR` | CSV 分隔符號；Tab（定位字元）請用 `\t`（預設為逗號） |
| `-W SIZE` | `--memory SIZE` | 最大記憶體，如 `64MB`、`1024KB`；省略時由引擎按輸入估算；下限 **16MB**。**不是** 未啟用授權時 **輸入檔案總大小** 上限（見下文「免費檔：輸入檔案總大小」一節） |
| `-T DIR` | `--temp-dir DIR` | 外排序臨時目錄（預設系統臨時）；臨時檔名形如 `hsort_<pid>.tmp`，排序後刪除 |
| `-O FILE` | `--output FILE` | 輸出路徑（預設標準輸出）；可與某輸入路徑相同 |
| `-E FILE` | `--error-file FILE` | 不合法資料列寫入此檔案；省略則略過異常列且不顯示訊息 |
| `-M` | `--merge` | 合併已排序檔案 |
| *（路徑）* | — | 輸入檔案：可多選且須放在**最後**；省略則從標準輸入讀 |

**說明**

- 短選項與長選項等價（如 `-C` / `--csv`）。
- 選項區分大小寫（`-C` ≠ `-c`）。

**與授權相關的 CLI**

```bash
hsort --license YOUR_KEY      # 啟用
hsort --check-license         # 查看狀態
```

### 使用範例

#### I. CSV 排序

**① 逗號分隔、最大記憶體 100MB、整行升序、標準輸入→標準輸出**

```bash
hsort -C -W100M < in.csv
```

**② LF 換行、Tab 分隔、多鍵（第 1 欄升序、第 3 欄降序）、輸出與輸入檔案**

```bash
hsort -C -P'\n' -D'\t' -K1a -K3d -O out.csv in1.csv in2.csv
```

*未指定 `-E`：欄數少於 3 的資料列不會寫入錯誤檔案。*

**③ 唯一輸出、整行降序、臨時目錄、輸出、錯誤檔案、輸入**

```bash
hsort -C -U -R -T /tmp -O out.csv -E err.csv in.csv
```

*異常列寫入 `err.csv`。*

**④ 穩定排序；第 1 欄 ASCII 升序、第 3 欄數值降序**

```bash
hsort -C -S -K1 -K3nd -O out.csv in.csv
```

#### II. 定長排序

**① 記錄長 64 位元組、最大記憶體 100MB、整行升序、標準輸入→標準輸出**

```bash
hsort -L64 -W100M < in.dat
```

**② 記錄長 64、多鍵（升序 + 數值降序）、輸出與輸入**

```bash
hsort -L64 -K0,5a -K3,8nd -O out.dat in1.dat in2.dat
```

*未指定 `-E`：鍵長度不足的資料列不會寫入錯誤檔案。*

**③ 記錄長 64、唯一輸出、整行降序、臨時目錄、輸出、錯誤檔案、輸入**

```bash
hsort -L64 -U -R -T /tmp -O out.dat -E err.dat in.dat
```

**④ 記錄長 64、穩定排序、多鍵（降序+升序）、輸出與輸入**

```bash
hsort -L64 -S -K1,3d -K6,9a -O out.dat in.dat
```

#### III. 變長排序

**① CRLF 換行、最大記憶體 100MB、整行升序、標準輸入→標準輸出**

```bash
hsort -P'\r\n' -W100M < in.dat
```

**② 多鍵（升序 + 數值降序）、輸出與輸入**

```bash
hsort -K5,8a -K0,3nd -O out.dat in1.dat in2.dat
```

**③ 唯一輸出、整行降序、臨時目錄、輸出、錯誤檔案、輸入**

```bash
hsort -U -R -T /tmp -O out.dat -E err.dat in.dat
```

**④ 穩定排序、多鍵（升序+降序）、輸出與輸入**

```bash
hsort -S -K1,3 -K5,9d -O out.dat in.dat
```

## 注意事項

1. 未指定格式時按**變長**處理。
2. 選項區分大小寫（如 `-C` ≠ `-c`）。
3. 短選項與長選項含義相同（如 `-C` / `--csv`）。
4. 變長/CSV 預設換行為 `\n`（LF）。
5. 輸入檔案路徑必須放在**最後**；其餘參數順序任意。
6. `pip install hsort` 後可使用 `hsort` 指令。
7. 在程式中：`import hsort` 並呼叫 `hsort.hsort(args)`。

## 介面語言

CLI 預設 **英文**。日文介面依下列順序偵測：

1. **`HSORT_LANG`**（顯式指定）
2. **Windows**：系統介面語言
3. **Unix/Linux**：`locale.getdefaultlocale()`
4. **`LANG` / `LANGUAGE` / `LC_ALL` / `LC_MESSAGES`**

**說明：** CLI 目前僅支援**英文與日文**介面，尚無**簡體或繁體中文**介面；無法辨識為日文時使用英文。

### 強制語言（`HSORT_LANG`）

**Windows PowerShell**

```powershell
$env:HSORT_LANG="en"
hsort --help

$env:HSORT_LANG="ja"
hsort --help

# 為使用者永久設定：
[System.Environment]::SetEnvironmentVariable("HSORT_LANG", "en", "User")
```

**Linux / macOS（Bash）**

```bash
export HSORT_LANG=en
hsort --help

export HSORT_LANG=ja
hsort --help
```

**Windows CMD**

```cmd
set HSORT_LANG=en
hsort --help

set HSORT_LANG=ja
hsort --help
```

可用取值：`ja` / `japanese` / `jp`（日文），`en` / `english`（英文）。PowerShell 請用 `$env:HSORT_LANG`，不要用 `set`。

### 偵錯：語系偵測

```powershell
$env:HSORT_DEBUG_LANG="1"
hsort --help
```

```bash
export HSORT_DEBUG_LANG=1
hsort --help
```

偵錯資訊會輸出至標準錯誤。

## Python API

可在程式中以函式庫方式呼叫。

### 匯入

```python
import hsort
```

### 函數

`hsort.hsort(args)` 接收與 CLI 相同的 argv 形式參數列。

```python
hsort.hsort(args: List[str]) -> int
```

- **args**：參數列表（與 CLI 相同）
- **回傳值**：結束代碼（`0` 表示成功）

### 使用範例

**範例 1 — 依第 1 欄排序 CSV**

```python
import hsort

ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-O", "output.csv",
    "input.csv",
])

if ret == 0:
    print("排序成功")
else:
    print(f"出錯，結束代碼: {ret}")
```

**範例 2 — CSV 多鍵（第 1 欄升序、第 3 欄降序）**

```python
import hsort

ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-K3d",
    "-O", "output.csv",
    "input.csv",
])
```

**範例 3 — 第 1 欄依數值排序**

```python
import hsort

ret = hsort.hsort([
    "-C",
    "-H",
    "-K1n",
    "-O", "output.csv",
    "input.csv",
])
```

**範例 4 — 定長（每筆 64 位元組）**

```python
import hsort

ret = hsort.hsort([
    "-L64",
    "-K0,20",
    "-O", "output.dat",
    "input.dat",
])
```

**範例 5 — 變長**

```python
import hsort

ret = hsort.hsort([
    "-K0,10",
    "-O", "output.dat",
    "input.dat",
])
```

**範例 6 — 唯一輸出**

```python
import hsort

ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-U",
    "-O", "output.csv",
    "input.csv",
])
```

**範例 7 — 穩定排序**

```python
import hsort

ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-S",
    "-O", "output.csv",
    "input.csv",
])
```

**範例 8 — 錯誤檔案**

```python
import hsort
import os

input_file = "input.csv"
output_file = "output.csv"
error_file = "errors.csv"

ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-K2",
    "-E", error_file,
    "-O", output_file,
    input_file,
])

if ret == 0:
    if os.path.exists(output_file):
        print(f"成功: {output_file}")
    if os.path.exists(error_file) and os.path.getsize(error_file) > 0:
        print(f"部分資料列見: {error_file}")
else:
    print(f"失敗，結束代碼: {ret}")
```

**範例 9 — 記憶體上限**

```python
import hsort

ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-W64MB",
    "-O", "output.csv",
    "input.csv",
])
```

**範例 10 — 批次處理目錄內 CSV**

```python
import hsort
import os
from pathlib import Path

def sort_csv_files(input_dir, output_dir):
    input_path = Path(input_dir)
    output_path = Path(output_dir)
    output_path.mkdir(exist_ok=True)

    for csv_file in input_path.glob("*.csv"):
        output_file = output_path / f"sorted_{csv_file.name}"
        ret = hsort.hsort([
            "-C",
            "-H",
            "-K1",
            "-O", str(output_file),
            str(csv_file),
        ])
        if ret == 0:
            print(f"完成: {csv_file.name}")
        else:
            print(f"失敗: {csv_file.name}（結束代碼 {ret}）")

sort_csv_files("data/input", "data/output")
```

### API 參考

**`hsort.hsort(args)`** — 執行排序/合併，規則與 CLI 相同。

- **args**（`List[str]`）：argv 形式列表。
- **回傳值**（`int`）：結束代碼。
- **例外**
  - **`ImportError`**：延伸模組未正確安裝
  - **`RuntimeError`**：未啟用授權且超過輸入檔案總大小上限（與 CLI 相同條件）

參數格式須與命令列一致。

## 執行環境

- Python **3.9+**  
- Windows、Linux 或 macOS  
- 建議 **≥ 16 MB** 可用記憶體（可用 `-W` 調節）  

## 定價與授權

HSORT 提供免費版與付費授權：

### 🟢 免費版
- 輸入檔案合計最大 **100MB**
- 無功能限制（僅總大小上限）

### 🔵 付費授權
- 輸入大小無上限
- 完整效能
- 商業使用

👉 啟用授權：

```bash
hsort --license YOUR_KEY
```

## 免費檔：輸入檔案總大小

未啟用**有效授權**時，命令列或 `hsort.hsort([...])` 中傳入的**一般輸入檔案合計大小**不得超過 **100 MiB**（**100 × 1024 × 1024** 位元組）。**CLI 與 API** 共用同一套檢查。

- 啟用**付費授權**後解除該總輸入大小限制（以授權合約為準）。  

## 授權（法律說明）

Copyright (c) 2015–2026 株式会社GPO

本專案**不是開源軟體**。軟體為**專有授權**；完整條款見 **`LICENSE`** 檔案。

- **非開源** — 除法律或書面協議允許外，不授予原始碼、再分發或修改的一般權利。  
- **免費檔** — 可能包含限制（例如上文總輸入大小）。不授予完整商業權利。  
- **付費授權** — 依與發行方的合約解鎖全部功能與約定範圍內的使用。  

👉 **購買授權（授權金鑰即時交付）：**  
https://github.com/xuhui-hou/hsort/blob/main/Payment.md

授權諮詢：**soft@gpo-i.com**

## 連結

- 首頁：https://github.com/xuhui-hou/hsort  
- 儲存庫：https://github.com/xuhui-hou/hsort  
- 議題：https://github.com/xuhui-hou/hsort/issues  
