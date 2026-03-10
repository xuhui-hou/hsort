# HSORT

**超高速ソートマージ Python拡張ライブラリ**

HSORTは、C言語で開発された高性能ソートエンジンをPython拡張ライブラリとして提供するパッケージです。自作メモリプールを使用して、超高速で固定長、可変長、CSVファイルのソート、マージ機能を提供します。

- **ソート機能**: ファイル内のレコードをデータ中の文字または数字をキーとして、昇順または降順に並べ替える機能
- **マージ機能**: 複数のファイルのデータを一つのファイルに併合する機能

大規模データファイルの処理に最適化された高性能ソート・マージエンジン。固定長、可変長、CSV形式に対応し、安定したマルチキーソートをサポートします。

## Python拡張ライブラリとしての特徴

- ✅ **pipで簡単インストール**: PyPIからインストール可能
- ✅ **コマンドラインインターフェース**: `hsort` コマンドとして使用可能
- ✅ **Python API**: プログラム内から `import hsort` で使用可能
- ✅ **クロスプラットフォーム**: Windows、Linux、macOSで動作
- ✅ **標準的なCLI形式**: Unixスタイルの `-` オプションと `--` 長オプションをサポート

## ソート方式

ソート機能の処理には、入力ファイルの総サイズ及び指定されたメモリのサイズにより次の二つの方式があります。

### メモリ上ソート
入力データ量に対して十分なメモリサイズを指定した場合、一時ファイルを使用しないでソートする処理のことです。

### 一時ファイルでソート
入力データ量に対して最大メモリサイズが不足した場合、一時ファイルを使用して、入力データを分割しながらソートする処理のことです。

**注意**: 使用可能なメモリサイズが未指定の場合、入力ファイルの総サイズを基づいて「メモリ上ソート」の必要なメモリサイズを自動計算してメモリを割り当ててみます。メモリは確保できなければ、「一時ファイルでソート」します。

## 機能

- ✅ ソフト起動の際にメモリプール用メモリを確保し、高速にメモリを割り当て、メモリ断片化を最小化すること
- ✅ 最大メモリサイズが指定できること、未指定の場合、自動計算すること
- ✅ 固定長、可変長のテキスト、バイナリファイルをサポートすること
- ✅ CSVファイルをサポートすること（文字列の記述には、ダブルクォーテーション（"）で囲むことは可能）
- ✅ 標準入力、標準出力をサポートすること
- ✅ ログ情報を標準エラーに出力すること
- ✅ 複数キー又は全レコードの昇順、降順が指定できること
- ✅ 等しいソートキーの最初レコードのみ出力すること（ユニック出力機能）
- ✅ レコード順序保持機能（安定ソート）
- ✅ 不正レコード（最大キー不足）をスキップ、又はエラーファイルに出力する
- ✅ ASCII順、数字順でソートすること
- ✅ 2G以上ファイルもサポートすること
- ✅ クロスプラットフォーム対応（Windows/Unix/Linux/macOS）

## インストール

### PyPIからインストール

```bash
pip install hsort
```

### TestPyPIからインストール（開発版）

```bash
pip install --index-url https://test.pypi.org/simple/ hsort
```

### インストールの確認

インストール後、以下のコマンドで動作確認できます：

```bash
# バージョン確認
hsort --version

# ヘルプ表示
hsort --help
```

## クイックスタート

### CSVファイルのソート

```bash
# 第1列でソート（ヘッダー付き）
hsort -C -H -K1 -O output.csv input.csv
```

### 固定長ファイルのソート

```bash
# 固定長ファイルのソート（レコード長64バイト）
hsort -L64 -K0,20 -O output.dat input.dat
```

### 可変長ファイルのソート

```bash
# 可変長ファイルのソート
hsort -K0,10 -O output.dat input.dat
```

## 基本的な使用方法

```bash
hsort [オプション] [入力ファイル...]
```

### パラメータ一覧

| 短オプション | 長オプション | 意味 |
| --- | --- | --- |
| `-h`, `--help` | `--help` | ヘルプを表示します |
| `-V` | `--version` | バージョン情報、ライセンス情報を表示します |
| `-C` | `--csv` | 入力データは CSV ファイルです。 |
| `-H` | `--header` | CSVファイルの先頭行をヘッダとして扱います |
| `-S` | `--stable` | 安定ソートフラグを指定します。 |
| `-U` | `--unique` | ユニーク出力フラグを指定します。 |
| `-L BYTES` | `--record-length BYTES` | 固定長ファイルを指定します。<br>入力ファイル編成ごとのレコード長<br>※ファイルのレコード長を「1 ～ 640KB」バイトの範囲で指定します。 |
| `-K KEY_SPEC` | `--key KEY_SPEC` | ソートキーを指定します（複数指定可能）。<br><br>**固定長、可変長の場合**: `開始位置[,長さ][n][a\|d]`<br>- 開始位置：0から始まるバイト位置<br>- 長さ：キーの長さ（省略可）<br>- n：数字順でソート（未指定するとASCII順）<br>- a：昇順（省略可）、d：降順<br><br>**CSVの場合**: `キー位置[n][a\|d]`<br>- キー位置：1から始まる列番号<br>- n：数字順でソート（未指定するとASCII順）<br>- a：昇順（省略可）、d：降順 |
| `-A` | `--all-asc` | 全レコード昇順フラグを指定します。<br>※-K と同時に指定すると、-K オプションを省略します。<br>※-R と同時に指定すると、後に指定されたオプションを省略します。 |
| `-R` | `--all-desc` | 全レコード降順フラグを指定します。<br>※-K と同時に指定すると、-K オプションを省略します。<br>※-A と同時に指定すると、後に指定されたオプションを省略します。 |
| `-P CODE` | `--newline CODE` | 可変長・CSV ファイルの改行コードを指定します。<br>※指定可能な値：`\n`, `\r`, `\r\n`<br>※未指定の場合、デフォルトは `\n` を使用します。 |
| `-D CHAR` | `--delimiter CHAR` | CSV ファイルのデリミタを指定します。<br>※タブ文字には `\t` を使用<br>※未指定の場合、カンマを使用します。 |
| `-W SIZE` | `--memory SIZE` | 使用できる最大メモリサイズを指定します。<br>※形式：数値[MB\|KB]（例：`64MB`, `1024KB`）<br>※未指定の場合、全メモリを使用してソートします<br>※常に最小値 16MB で制限されます。 |
| `-T DIR` | `--temp-dir DIR` | ソート用一時ファイルのディレクトリを指定します。<br>※未指定の場合、システムの一時ディレクトリを使用します。<br>※一時ファイル名は `hsort_プロセスid.tmp` となります。<br>※ソート後、一時ファイルを自動的に削除します。 |
| `-O FILE` | `--output FILE` | 出力先ファイルのパスを指定します。<br>※出力先ファイルを入力ファイルと同じファイルの指定もできます。<br>※未指定の場合、標準出力に出力します |
| `-E FILE` | `--error-file FILE` | エラーファイルのパスを指定します。<br>※未指定の場合、不正レコードがスキップされ、出力されない。 |
| `-M` | `--merge` | ソート済みファイルをマージします |
| `入力ファイル...` | - | 入力ファイルのパスを指定します。<br>※複数指定可、更に全て引数の最後に指定しなければならない。<br>※未指定の場合、標準入力から読み込みます |

**注意**: 
- オプションは大文字、小文字を区別します（例：`-C` と `-c` は異なります）
- 短オプションと長オプションは同じ意味です（例：`-C` と `--csv` は同じ）

### 使用例

#### I、CSVファイルのソート

**① デリミタ（カンマ）、最大メモリサイズ 100MB、全レコード昇順、標準入力、標準出力**
```bash
hsort -C -W100M < in.csv
```

**② 改行コード（Lf）、デリミタ（Tab）、複数キー、昇順、降順、出力ファイル、入力ファイル**
```bash
hsort -C -P'\n' -D'\t' -K1a -K3d -O out.csv in1.csv in2.csv
```
※エラーファイルが未指定のため、列数が3列不足のレコードがエラーレコードとして出力されない

**③ ユニーク出力、全レコード降順、一時ファイル、出力ファイル、エラーファイル、入力ファイル**
```bash
hsort -C -U -R -T /tmp -O out.csv -E err.csv in.csv
```
※エラーレコードがエラーファイル err.csv に出力される

**④ 安定ソート、1列目 ASCII 昇順、3列目数字で降順**
```bash
hsort -C -S -K1 -K3nd -O out.csv in.csv
```

#### II、固定長ファイルのソート

**① レコード長 64、最大メモリサイズ 100MB、全レコード昇順、標準入力、標準出力**
```bash
hsort -L64 -W100M < in.dat
```

**② レコード長 64、複数キー、昇順、数字で降順、出力ファイル、入力ファイル**
```bash
hsort -L64 -K0,5a -K3,8nd -O out.dat in1.dat in2.dat
```
※エラーファイルが未指定のため、列数が3列不足のレコードがエラーレコードとして出力されない

**③ レコード長 64、ユニーク出力、全レコード降順、一時ファイル、出力ファイル、エラーファイル、入力ファイル**
```bash
hsort -L64 -U -R -T /tmp -O out.dat -E err.dat in.dat
```
※エラーレコードがエラーファイル err.dat に出力される

**④ レコード長 64、安定ソート、複数キー、降順、昇順、出力ファイル、入力ファイル**
```bash
hsort -L64 -S -K1,3d -K6,9a -O out.dat in.dat
```

#### III、可変長ファイルのソート

**① 改行コード（CrLf）、最大メモリサイズ 100MB、全レコード昇順、標準入力、標準出力**
```bash
hsort -P'\r\n' -W100M < in.dat
```

**② 複数キー、昇順、数字で降順、出力ファイル、入力ファイル**
```bash
hsort -K5,8a -K0,3nd -O out.dat in1.dat in2.dat
```
※エラーファイルが未指定のため、列数が3列不足のレコードがエラーレコードとして出力されない

**③ ユニーク出力、全レコード降順、一時ファイル、出力ファイル、エラーファイル、入力ファイル**
```bash
hsort -U -R -T /tmp -O out.dat -E err.dat in.dat
```
※エラーレコードがエラーファイル err.dat に出力される

**④ 安定ソート、複数キー、昇順、降順、出力ファイル、入力ファイル**
```bash
hsort -S -K1,3 -K5,9d -O out.dat in.dat
```

## 注意事項

1. 入力ファイルのフォーマットが指定されないと、可変長として処理します。
2. オプションは大文字、小文字を区別します（例：`-C` と `-c` は異なります）。
3. 短オプションと長オプションは同じ意味です（例：`-C` と `--csv` は同じ）。
4. 可変長、CSV ファイルのデフォルトな改行コードは `\n` (LF) となります。
5. 入力ファイルは最後にならなくてはいけませんが、他の各パラメタ順序は任意です。
6. Python拡張ライブラリとして、`pip install hsort` でインストール後、`hsort` コマンドが利用可能になります。
7. プログラム内から使用する場合は `import hsort` して `hsort.hsort(args)` 関数を呼び出します。

## Python API の使用方法

HSORTは、プログラム内でPythonライブラリとしても使用できます。

### 基本的なインポート

```python
import hsort
```

### API関数

メイン関数は `hsort.hsort(args)` で、コマンドラインインターフェースと同様の引数リストを受け取ります。

**関数シグネチャ:**
```python
hsort.hsort(args: List[str]) -> int
```

**パラメータ:**
- `args`: コマンドライン引数のリスト（CLIと同じ形式）

**戻り値:**
- 終了コード: 成功時は `0`、エラー時は非ゼロ

### Python API の使用例

#### 例1: CSVファイルのソート

```python
import hsort

# 第1列でCSVファイルをソート
ret = hsort.hsort([
    "-C",      # CSV形式
    "-H",      # ヘッダー付き
    "-K1",     # 第1列でソート
    "-O", "output.csv",
    "input.csv"
])

if ret == 0:
    print("ソート成功！")
else:
    print(f"エラーが発生しました。終了コード: {ret}")
```

#### 例2: CSVマルチキーソート

```python
import hsort

# 第1列昇順、第3列降順でソート
ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",     # 第1列昇順
    "-K3d",    # 第3列降順
    "-O", "output.csv",
    "input.csv"
])
```

#### 例3: CSV数値ソート

```python
import hsort

# 第1列を数値としてソート
ret = hsort.hsort([
    "-C",
    "-H",
    "-K1n",    # 第1列を数値ソート
    "-O", "output.csv",
    "input.csv"
])
```

#### 例4: 固定長ファイルのソート

```python
import hsort

# 固定長ファイルのソート（レコード長64バイト）
ret = hsort.hsort([
    "-L64",    # レコード長64バイト
    "-K0,20",  # ソートキー：位置0、長さ20
    "-O", "output.dat",
    "input.dat"
])
```

#### 例5: 可変長ファイルのソート

```python
import hsort

# 可変長ファイルのソート
ret = hsort.hsort([
    "-K0,10",  # ソートキー：位置0、長さ10
    "-O", "output.dat",
    "input.dat"
])
```

#### 例6: ユニーク出力

```python
import hsort

# 重複レコードを削除
ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-U",      # ユニーク出力
    "-O", "output.csv",
    "input.csv"
])
```

#### 例7: 安定ソート

```python
import hsort

# 安定ソート（同じキー値のレコードの入力順序を保持）
ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-S",      # 安定ソート
    "-O", "output.csv",
    "input.csv"
])
```

#### 例8: エラーハンドリング

```python
import hsort
import os

input_file = "input.csv"
output_file = "output.csv"
error_file = "errors.csv"

# エラーファイル出力付きでソート
ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-K2",
    "-E", error_file,  # エラーレコードファイル
    "-O", output_file,
    input_file
])

if ret == 0:
    if os.path.exists(output_file):
        print(f"✓ ソート成功！出力: {output_file}")
    if os.path.exists(error_file) and os.path.getsize(error_file) > 0:
        print(f"⚠ 一部のレコードにエラーがありました。確認: {error_file}")
else:
    print(f"✗ ソート失敗。終了コード: {ret}")
```

#### 例9: メモリ管理

```python
import hsort

# メモリ使用量を64MBに制限
ret = hsort.hsort([
    "-C",
    "-H",
    "-K1",
    "-W64MB",  # メモリ制限
    "-O", "output.csv",
    "input.csv"
])
```

#### 例10: プログラムによるファイル処理

```python
import hsort
import os
from pathlib import Path

def sort_csv_files(input_dir, output_dir):
    """ディレクトリ内のすべてのCSVファイルをソート"""
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
            str(csv_file)
        ])
        
        if ret == 0:
            print(f"✓ ソート完了: {csv_file.name}")
        else:
            print(f"✗ 失敗: {csv_file.name} (終了コード: {ret})")

# 使用例
sort_csv_files("data/input", "data/output")
```

### APIリファレンス

**関数:** `hsort.hsort(args)`

hsortソート操作を実行します。

- **パラメータ:**
  - `args` (List[str]): コマンドライン引数のリスト。形式はCLI使用法と同一です。
  
- **戻り値:**
  - `int`: 終了コード（成功時は0、エラー時は非ゼロ）

- **例外:**
  - `ImportError`: hsortモジュールが正しくインストールされていない場合

**注意:** 引数の形式はコマンドラインインターフェースと同じです。すべてのオプションとファイルパスは、リスト内の文字列として提供する必要があります。

## 要件

- Python 3.6 以上
- サポートされているオペレーティングシステム: Windows, Linux, macOS
- メモリ: 推奨は少なくとも16MBの利用可能メモリ（`-W`オプションで調整可能）

## ライセンス

本プロジェクトは Apache 2.0 の下で公開されています。

Copyright (C) 2015 by 株式会社GPO

### MIT License

```
MIT License

Copyright (c) 2026 株式会社GPO

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 連絡方式

不具合を発見したら、下記メール宛に連絡していただいたら、幸いです。

**soft@gpo-i.com**

## リンク

- ホームページ: https://github.com/xuhui-hou/hsort
- リポジトリ: https://github.com/xuhui-hou/hsort
- イシュー: https://github.com/xuhui-hou/hsort/issues
