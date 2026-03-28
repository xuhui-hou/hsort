# HSORT

**[English（README.md）](README.md)** | **[日本語（README.ja.md）](README.ja.md)** | **[繁體中文／台灣（README.zh-zw.md）](README.zh-zw.md)**

高性能排序/合并 **Python 扩展**，底层为 C 引擎。通过内部内存池快速排序定长、变长与 CSV 文件，并支持将已排序输入合并为单一输出流。

- **排序** — 按文本或数值键升序/降序排列记录  
- **合并** — 将多个已排序文件合并为一条流  

## 产品要点

- 通过 **pip** 从 PyPI 安装  
- **`hsort`** 命令行与 **`import hsort`** Python API  
- 支持 **Windows、Linux、macOS**  
- Unix 风格 **`-` / `--`** 选项  

## 排序模式

**内存排序** — 当可用内存足以容纳数据集时，排序过程可不使用临时文件。

**外排序** — 当内存不足时，数据被分块排序，并借助临时文件完成合并。

若省略 **`-W`**，引擎会根据输入大小估算内存；若分配失败，将回退到外排序。

## 功能概要

- 可配置内存上限（`-W`）；省略时自动估算  
- 定长、变长（文本/二进制）及 **CSV**  
- 标准输入/输出；日志输出到标准错误  
- 多键排序；稳定排序（`-S`）；唯一输出（`-U`）  
- 无效记录可跳过或写入错误文件（`-E`）  
- ASCII 与数值键模式；支持大文件  
- 跨平台  

## 安装

```bash
pip install hsort
```

Test PyPI（如适用）：

```bash
pip install --index-url https://test.pypi.org/simple/ hsort
```

验证安装：

```bash
hsort --version
hsort --help
```

## 快速上手

**CSV（含表头，按第 1 列排序）**

```bash
hsort -C -H -K1 -O output.csv input.csv
```

**定长（64 字节记录，键为字节 0–19）**

```bash
hsort -L64 -K0,20 -O output.dat input.dat
```

**变长**

```bash
hsort -K0,10 -O output.dat input.dat
```

## 命令行用法

```text
hsort [options] [input files...]
```

选项区分大小写（`-C` ≠ `-c`）。短选项与长选项等价（`-C` / `--csv`）。

权威参数说明请运行：

```bash
hsort --help
```

### 参数一览

| 短选项 | 长选项 | 说明 |
| --- | --- | --- |
| `-h` | `--help` | 显示帮助 |
| `-V` | `--version` | 显示版本与许可信息 |
| `-C` | `--csv` | 输入为 CSV |
| `-H` | `--header` | CSV 首行作为表头 |
| `-S` | `--stable` | 稳定排序 |
| `-U` | `--unique` | 唯一输出（每个键保留首条记录） |
| `-L BYTES` | `--record-length BYTES` | 定长记录；每条记录长度 **1～640KB** |
| `-K KEY_SPEC` | `--key` | 排序键（可重复）。**定长/变长：** `起始[,长度][n][a\|d]` — 起始为从 0 起的字节偏移；长度可省略；`n` 为数值排序（默认 ASCII）；`a` 升序（默认）、`d` 降序。**CSV：** `列号[n][a\|d]` — 列号从 1 开始 |
| `-A` | `--all-asc` | 整行升序。与 `-K` 同时时以 `-K` 为准；与 `-R` 同时时以后写的选项为准 |
| `-R` | `--all-desc` | 整行降序。与 `-K` 同时时以 `-K` 为准；与 `-A` 同时时以后写的选项为准 |
| `-P CODE` | `--newline CODE` | 变长/CSV 换行：`\n`、`\r`、`\r\n`（默认 `\n`） |
| `-D CHAR` | `--delimiter CHAR` | CSV 分隔符；制表符用 `\t`（默认逗号） |
| `-W SIZE` | `--memory SIZE` | 最大内存，如 `64MB`、`1024KB`；省略时由引擎按输入估算；下限 **16MB**。**不是** 无许可时 **输入文件总大小** 上限（见下文「免费档：输入文件总大小」一节） |
| `-T DIR` | `--temp-dir DIR` | 外排序临时目录（默认系统临时）；临时文件名形如 `hsort_<pid>.tmp`，排序后删除 |
| `-O FILE` | `--output FILE` | 输出路径（默认标准输出）；可与某输入路径相同 |
| `-E FILE` | `--error-file FILE` | 非法记录写入此文件；省略则静默跳过坏行 |
| `-M` | `--merge` | 合并已排序文件 |
| *（路径）* | — | 输入文件：可多选且须放在**最后**；省略则从标准输入读 |

**说明**

- 短选项与长选项等价（如 `-C` / `--csv`）。
- 选项区分大小写（`-C` ≠ `-c`）。

**与许可证相关的 CLI**

```bash
hsort --license YOUR_KEY      # 激活
hsort --check-license         # 查看状态
```

### 使用示例

#### I. CSV 排序

**① 逗号分隔、最大内存 100MB、整行升序、标准输入→标准输出**

```bash
hsort -C -W100M < in.csv
```

**② LF 换行、Tab 分隔、多键（第 1 列升序、第 3 列降序）、输出与输入文件**

```bash
hsort -C -P'\n' -D'\t' -K1a -K3d -O out.csv in1.csv in2.csv
```

*未指定 `-E`：列数不足 3 列的行不会写入错误文件。*

**③ 唯一输出、整行降序、临时目录、输出、错误文件、输入**

```bash
hsort -C -U -R -T /tmp -O out.csv -E err.csv in.csv
```

*错误行写入 `err.csv`。*

**④ 稳定排序；第 1 列 ASCII 升序、第 3 列数值降序**

```bash
hsort -C -S -K1 -K3nd -O out.csv in.csv
```

#### II. 定长排序

**① 记录长 64 字节、最大内存 100MB、整行升序、标准输入→标准输出**

```bash
hsort -L64 -W100M < in.dat
```

**② 记录长 64、多键（升序 + 数值降序）、输出与输入**

```bash
hsort -L64 -K0,5a -K3,8nd -O out.dat in1.dat in2.dat
```

*未指定 `-E`：键长度不足的行不会写入错误文件。*

**③ 记录长 64、唯一输出、整行降序、临时目录、输出、错误文件、输入**

```bash
hsort -L64 -U -R -T /tmp -O out.dat -E err.dat in.dat
```

**④ 记录长 64、稳定排序、多键（降序+升序）、输出与输入**

```bash
hsort -L64 -S -K1,3d -K6,9a -O out.dat in.dat
```

#### III. 变长排序

**① CRLF 换行、最大内存 100MB、整行升序、标准输入→标准输出**

```bash
hsort -P'\r\n' -W100M < in.dat
```

**② 多键（升序 + 数值降序）、输出与输入**

```bash
hsort -K5,8a -K0,3nd -O out.dat in1.dat in2.dat
```

**③ 唯一输出、整行降序、临时目录、输出、错误文件、输入**

```bash
hsort -U -R -T /tmp -O out.dat -E err.dat in.dat
```

**④ 稳定排序、多键（升序+降序）、输出与输入**

```bash
hsort -S -K1,3 -K5,9d -O out.dat in.dat
```

## 注意事项

1. 未指定格式时按**变长**处理。
2. 选项区分大小写（如 `-C` ≠ `-c`）。
3. 短选项与长选项含义相同（如 `-C` / `--csv`）。
4. 变长/CSV 默认换行为 `\n`（LF）。
5. 输入文件路径必须放在**最后**；其余参数顺序任意。
6. `pip install hsort` 后可使用 `hsort` 命令。
7. 在程序中：`import hsort` 并调用 `hsort.hsort(args)`。

## 界面语言

CLI 默认 **英文**。日文界面按以下顺序检测：

1. **`HSORT_LANG`**（显式指定）
2. **Windows**：系统界面语言
3. **Unix/Linux**：`locale.getdefaultlocale()`
4. **`LANG` / `LANGUAGE` / `LC_ALL` / `LC_MESSAGES`**

**说明：** 当前**无简体中文界面**；未识别为日文时使用英文。

### 强制语言（`HSORT_LANG`）

**Windows PowerShell**

```powershell
$env:HSORT_LANG="en"
hsort --help

$env:HSORT_LANG="ja"
hsort --help

# 为用户永久设置：
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

可用取值：`ja` / `japanese` / `jp`（日文），`en` / `english`（英文）。PowerShell 请用 `$env:HSORT_LANG`，不要用 `set`。

### 调试区域检测

```powershell
$env:HSORT_DEBUG_LANG="1"
hsort --help
```

```bash
export HSORT_DEBUG_LANG=1
hsort --help
```

调试信息输出到标准错误。

## Python API

可在程序中以库方式调用。

### 导入

```python
import hsort
```

### 函数

`hsort.hsort(args)` 接收与 CLI 相同的 argv 风格参数列表。

```python
hsort.hsort(args: List[str]) -> int
```

- **args**：参数列表（与 CLI 相同）
- **返回值**：退出码（`0` 表示成功）

### 使用示例

**示例 1 — 按第 1 列排序 CSV**

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
    print(f"出错，退出码: {ret}")
```

**示例 2 — CSV 多键（第 1 列升序、第 3 列降序）**

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

**示例 3 — 第 1 列按数值排序**

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

**示例 4 — 定长（64 字节记录）**

```python
import hsort

ret = hsort.hsort([
    "-L64",
    "-K0,20",
    "-O", "output.dat",
    "input.dat",
])
```

**示例 5 — 变长**

```python
import hsort

ret = hsort.hsort([
    "-K0,10",
    "-O", "output.dat",
    "input.dat",
])
```

**示例 6 — 唯一输出**

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

**示例 7 — 稳定排序**

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

**示例 8 — 错误文件**

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
        print(f"部分行见: {error_file}")
else:
    print(f"失败，退出码: {ret}")
```

**示例 9 — 内存上限**

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

**示例 10 — 批量处理目录内 CSV**

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
            print(f"失败: {csv_file.name}（退出码 {ret}）")

sort_csv_files("data/input", "data/output")
```

### API 参考

**`hsort.hsort(args)`** — 执行排序/合并，规则与 CLI 相同。

- **args**（`List[str]`）：argv 风格列表。
- **返回值**（`int`）：退出码。
- **异常**
  - **`ImportError`**：扩展未正确安装
  - **`RuntimeError`**：无许可时超过输入文件总大小上限（与 CLI 相同条件）

参数格式须与命令行一致。

## 运行环境

- Python **3.6+**  
- Windows、Linux 或 macOS  
- 建议 **≥ 16 MB** 可用内存（可用 `-W` 调节）  

## 定价与许可

HSORT 提供无许可免费使用与付费许可：

### 🟢 免费版
- 输入文件合计最大 **100MB**
- 无功能阉割（仅总大小限制）

### 🔵 付费许可
- 输入大小无上限
- 完整性能
- 商业使用

👉 激活许可：

```bash
hsort --license YOUR_KEY
```

## 免费档：输入文件总大小

未激活**有效许可**时，命令行或 `hsort.hsort([...])` 中传入的**普通输入文件合计大小**不得超过 **100 MiB**（**100 × 1024 × 1024** 字节）。**CLI 与 API** 共用同一检查。

- 激活**付费许可**后解除该总输入大小限制（以许可协议为准）。  

## 许可（法律说明）

Copyright (c) 2015–2026 株式会社GPO

本项目**不是开源软件**。软件为**专有许可**；完整条款见 **`LICENSE`** 文件。

- **非开源** — 除法律或书面协议允许外，不授予源代码、再分发或修改的一般权利。  
- **免费档** — 可能包含限制（例如上文总输入大小）。不授予完整商业权利。  
- **付费许可** — 按与发行方的协议解锁全部功能与约定范围内的使用。  

👉 购买许可：
https://softwave56.gumroad.com/l/python-hsort

许可咨询：**soft@gpo-i.com**

## 链接

- 主页：https://github.com/xuhui-hou/hsort  
- 仓库：https://github.com/xuhui-hou/hsort  
- 问题：https://github.com/xuhui-hou/hsort/issues  
