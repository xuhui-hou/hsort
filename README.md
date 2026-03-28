# HSORT

## 🌐 Languages

- 🇺🇸 English
- 🇯🇵 [日本語](https://github.com/xuhui-hou/hsort/blob/main/README.ja.md)
- 🇨🇳 [简体中文](https://github.com/xuhui-hou/hsort/blob/main/README.zh.md)
- 🇹🇼 [繁體中文（台灣）](https://github.com/xuhui-hou/hsort/blob/main/README.zh-zw.md)

High-performance sort/merge **Python extension** backed by a C engine. Uses an internal memory pool for fast sorting of fixed-length, variable-length, and CSV files, plus merge of sorted inputs.

- **Sort** — Order records by text or numeric keys, ascending or descending  
- **Merge** — Combine multiple sorted files into one stream  

## Highlights

- Install with **pip** from PyPI  
- **`hsort`** CLI and **`import hsort`** Python API  
- **Windows, Linux, macOS**  
- Unix-style **`-` / `--`** options  

## Sort modes

**In-memory sort** — When enough memory is available for the dataset, sorting avoids temp files.

**External sort** — When memory is insufficient, data is split, sorted in chunks, and merged using temporary files.

If **`-W`** is omitted, the engine estimates memory from input size; if allocation fails, it falls back to external sort.

## Features (summary)

- Configurable memory budget (`-W`) with automatic sizing when omitted  
- Fixed-length, variable-length (text/binary), and **CSV**  
- Stdin/stdout; logs on stderr  
- Multi-key sort; stable sort (`-S`); unique output (`-U`)  
- Invalid records can be skipped or written to an error file (`-E`)  
- ASCII vs numeric key modes; large files supported  
- Cross-platform  

## Installation

```bash
pip install hsort
```

Test PyPI (when applicable):

```bash
pip install --index-url https://test.pypi.org/simple/ hsort
```

Check install:

```bash
hsort --version
hsort --help
```

## Quick start

**CSV (header row, sort column 1)**

```bash
hsort -C -H -K1 -O output.csv input.csv
```

**Fixed-length (64-byte records, key bytes 0–19)**

```bash
hsort -L64 -K0,20 -O output.dat input.dat
```

**Variable-length**

```bash
hsort -K0,10 -O output.dat input.dat
```

## CLI usage

```text
hsort [options] [input files...]
```

Options are **case-sensitive** (`-C` ≠ `-c`). Short and long forms are equivalent (`-C` / `--csv`).

For the authoritative option list, run:

```bash
hsort --help
```

### Option reference

| Short | Long | Description |
| --- | --- | --- |
| `-h` | `--help` | Show help |
| `-V` | `--version` | Show version and license info |
| `-C` | `--csv` | Input is CSV |
| `-H` | `--header` | Treat first CSV row as header |
| `-S` | `--stable` | Stable sort |
| `-U` | `--unique` | Unique output (first record per key) |
| `-L BYTES` | `--record-length BYTES` | Fixed-length records; length **1–640KB** per file layout |
| `-K KEY_SPEC` | `--key` | Sort key (repeatable). **Fixed / variable-length:** `start[,len][n][a\|d]` — start: 0-based byte offset; len: optional key length; `n`: numeric sort (default ASCII); `a`: ascending (default), `d`: descending. **CSV:** `col[n][a\|d]` — column number from 1 |
| `-A` | `--all-asc` | Sort whole record ascending. With `-K`, `-K` wins; with `-R`, the later flag wins |
| `-R` | `--all-desc` | Sort whole record descending. With `-K`, `-K` wins; with `-A`, the later flag wins |
| `-P CODE` | `--newline CODE` | Newline for variable-length / CSV: `\n`, `\r`, `\r\n` (default `\n`) |
| `-D CHAR` | `--delimiter CHAR` | CSV delimiter; use `\t` for tab (default comma) |
| `-W SIZE` | `--memory SIZE` | Max memory, e.g. `64MB`, `1024KB`; if omitted, engine sizes from input; minimum **16MB** enforced. **Not** the free-tier **total input file size** cap (see [Free tier](#free-tier-total-input-file-size) below) |
| `-T DIR` | `--temp-dir DIR` | Temp directory for external sort (default: system temp); files named like `hsort_<pid>.tmp`, removed after sort |
| `-O FILE` | `--output FILE` | Output path (default stdout); may match an input path |
| `-E FILE` | `--error-file FILE` | Invalid records → this file; if omitted, bad records are skipped silently |
| `-M` | `--merge` | Merge already-sorted files |
| *(paths)* | — | Input files: multiple allowed, **must be last**; if omitted, read stdin |

**Notes**

- Short and long options are equivalent (e.g. `-C` / `--csv`).
- Options are case-sensitive (`-C` ≠ `-c`).

**License-related CLI**

```bash
hsort --license YOUR_KEY      # activate
hsort --check-license         # status
```

### Examples

#### I. CSV sorting

**① Comma delimiter, max memory 100MB, whole-record ascending, stdin → stdout**

```bash
hsort -C -W100M < in.csv
```

**② LF newline, tab delimiter, multi-key (col1 asc, col3 desc), output + inputs**

```bash
hsort -C -P'\n' -D'\t' -K1a -K3d -O out.csv in1.csv in2.csv
```

*No `-E`: rows with fewer than 3 columns are not written to an error file.*

**③ Unique, whole-record descending, temp dir, output, error file, input**

```bash
hsort -C -U -R -T /tmp -O out.csv -E err.csv in.csv
```

*Bad records go to `err.csv`.*

**④ Stable sort; column 1 ASCII ascending, column 3 numeric descending**

```bash
hsort -C -S -K1 -K3nd -O out.csv in.csv
```

#### II. Fixed-length sorting

**① Record length 64, max memory 100MB, whole-record ascending, stdin → stdout**

```bash
hsort -L64 -W100M < in.dat
```

**② Record length 64, multi-key (asc + numeric desc), output + inputs**

```bash
hsort -L64 -K0,5a -K3,8nd -O out.dat in1.dat in2.dat
```

*No `-E`: rows shorter than required keys are not written to an error file.*

**③ Record length 64, unique, whole-record descending, temp dir, output, error file, input**

```bash
hsort -L64 -U -R -T /tmp -O out.dat -E err.dat in.dat
```

**④ Record length 64, stable, multi-key (desc + asc), output + input**

```bash
hsort -L64 -S -K1,3d -K6,9a -O out.dat in.dat
```

#### III. Variable-length sorting

**① CRLF newline, max memory 100MB, whole-record ascending, stdin → stdout**

```bash
hsort -P'\r\n' -W100M < in.dat
```

**② Multi-key (asc + numeric desc), output + inputs**

```bash
hsort -K5,8a -K0,3nd -O out.dat in1.dat in2.dat
```

**③ Unique, whole-record descending, temp dir, output, error file, input**

```bash
hsort -U -R -T /tmp -O out.dat -E err.dat in.dat
```

**④ Stable, multi-key (asc + desc), output + input**

```bash
hsort -S -K1,3 -K5,9d -O out.dat in.dat
```

## Notes

1. If format is unspecified, input is treated as variable-length.
2. Options are case-sensitive (e.g. `-C` ≠ `-c`).
3. Short and long options mean the same (e.g. `-C` / `--csv`).
4. Default newline for variable-length / CSV is `\n` (LF).
5. Input file paths must appear **last**; other options can be in any order.
6. After `pip install hsort`, the `hsort` command is available.
7. In code: `import hsort` and call `hsort.hsort(args)`.

## Language (UI)

CLI messages default to **English**. Japanese is used when detected from, in order:

1. **`HSORT_LANG`** (explicit)
2. **Windows**: system UI language
3. **Unix/Linux**: `locale.getdefaultlocale()`
4. **`LANG`**, **`LANGUAGE`**, **`LC_ALL`**, **`LC_MESSAGES`**

### Forcing the language (`HSORT_LANG`)

**Windows PowerShell**

```powershell
$env:HSORT_LANG="en"
hsort --help

$env:HSORT_LANG="ja"
hsort --help

# Persist for user account:
[System.Environment]::SetEnvironmentVariable("HSORT_LANG", "en", "User")
```

**Linux / macOS (Bash)**

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

Accepted values: `ja` / `japanese` / `jp` (Japanese), `en` / `english` (English). On PowerShell, use `$env:HSORT_LANG`, not `set`.

### Debug locale detection

```powershell
$env:HSORT_DEBUG_LANG="1"
hsort --help
```

```bash
export HSORT_DEBUG_LANG=1
hsort --help
```

Details print to stderr.

## Python API

HSORT can be used as a library.

### Import

```python
import hsort
```

### Function

`hsort.hsort(args)` takes the same argv-style list as the CLI.

```python
hsort.hsort(args: List[str]) -> int
```

- **args**: argument list (same as CLI)
- **Return**: exit code (`0` = success)

### Examples

**Example 1 — Sort CSV by column 1**

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
    print("Sort succeeded")
else:
    print(f"Error, exit code: {ret}")
```

**Example 2 — CSV multi-key (col1 asc, col3 desc)**

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

**Example 3 — CSV numeric sort on column 1**

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

**Example 4 — Fixed-length (64-byte records)**

```python
import hsort

ret = hsort.hsort([
    "-L64",
    "-K0,20",
    "-O", "output.dat",
    "input.dat",
])
```

**Example 5 — Variable-length**

```python
import hsort

ret = hsort.hsort([
    "-K0,10",
    "-O", "output.dat",
    "input.dat",
])
```

**Example 6 — Unique output**

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

**Example 7 — Stable sort**

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

**Example 8 — Error file**

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
        print(f"OK: {output_file}")
    if os.path.exists(error_file) and os.path.getsize(error_file) > 0:
        print(f"Some rows in: {error_file}")
else:
    print(f"Failed, exit code: {ret}")
```

**Example 9 — Memory limit**

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

**Example 10 — Batch CSV files**

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
            print(f"OK: {csv_file.name}")
        else:
            print(f"Fail: {csv_file.name} (exit {ret})")

sort_csv_files("data/input", "data/output")
```

### API reference

**`hsort.hsort(args)`** — run a sort/merge; same rules as the CLI.

- **args** (`List[str]`): argv-style list.
- **Returns** (`int`): exit code.
- **Raises**
  - **`ImportError`**: extension not installed
  - **`RuntimeError`**: free-tier total input size exceeded (same condition as CLI)

`args` must follow the same rules as the command line.

## Requirements

- Python **3.6+**  
- Windows, Linux, or macOS  
- Suggest **≥ 16 MB** available RAM (tune with `-W`)  

## Pricing & License

HSORT offers a free tier and paid licenses:

### 🟢 Free version
- Up to **100MB total input size**
- No feature restrictions (size limit only)

### 🔵 Paid license
- Unlimited input size
- Full performance
- Commercial use

👉 Activate license:

```bash
hsort --license YOUR_KEY
```

## Free tier: total input file size

When **no valid license** is activated, the **combined size of regular input files** passed on the command line or in `hsort.hsort([...])` must not exceed **100 MiB** (**100 × 1024 × 1024** bytes). The check is shared by **CLI and API**.

- Defined in source as **`FREE_VERSION_MAX_TOTAL_INPUT_BYTES`** in **`hsort/api.py`**.  
- This is **independent** of **`-W` / `--memory`** (e.g. `-W100M` limits sort memory, not this license input cap).  
- Activating a **paid license** removes this total input-size limit (subject to your license agreement).  

## License

Copyright (c) 2015–2026 株式会社GPO

This project is **not open source**. The software is **proprietary**; see the **`LICENSE`** file for full terms.

- **Not open source** — No general right to source, redistribution, or modification except as allowed by law or a written agreement.  
- **Free tier** — May include limits (e.g. total input size as above). Does not grant full commercial rights.  
- **Paid license** — Unlocks full features per your agreement with the publisher.  

👉 **Buy License (Instant Key Delivery):**  
https://your-website.com/hsort-license

Licensing contact: **soft@gpo-i.com**

## Links

- Homepage: https://github.com/xuhui-hou/hsort  
- Repository: https://github.com/xuhui-hou/hsort  
- Issues: https://github.com/xuhui-hou/hsort/issues  
