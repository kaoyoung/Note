# Q1 : 以下shell程式在幹嘛 ? 
```sh
#!/usr/bin/env bash
set -euo pipefail

K_VALUES="512 1024 2048"

UK_NET="./examples/uk-2005_paper.netl"
SK_NET="./examples/sk-2005_paper.netl"

BIN_CON="./deploy/freight_con"
BIN_CUT="./deploy/freight_cut"

# Pre-create output files (optional, not required)
for prefix in uk-2005 sk-2005; do
    for op in con cut; do
        for k in $K_VALUES; do
            touch "${prefix}_${op}_${k}"
        done
    done
done

run_one_set () {
    local bin="$1"
    local input="$2"
    local prefix="$3"
    local op="$4"

    for k in $K_VALUES; do
        echo "Running $op on $prefix with k=$k..."
        "$bin" "$input" --k="$k" > "${prefix}_${op}_${k}"
    done
}

# uk con / sk con / uk cut / sk cut
run_one_set "$BIN_CON" "$UK_NET" "uk-2005" "con"
run_one_set "$BIN_CON" "$SK_NET" "sk-2005" "con"

run_one_set "$BIN_CUT" "$UK_NET" "uk-2005" "cut"
run_one_set "$BIN_CUT" "$SK_NET" "sk-2005" "cut"
```
### `#!/usr/bin/env bash`這在幹嘛 ? 
因為我們要使用bash，所以告訴系統去$PATH找bash檔案，然後用它來執行腳本。Bash script 不需要編譯，它是 **直譯式（interpreted）**。
>為何不用`#!/bin/bash` ?
>不同系統的bash不一定放在`/bin/bash`。
### `set -euo pipefail` 這在幹嘛?
參考自[Bash 隨筆記記 — set 指令](https://medium.com/@40243105s/bash-%E9%9A%A8%E7%AD%86%E8%A8%98%E8%A8%98-set-%E6%8C%87%E4%BB%A4-50b2a2e9e099)
透過設定 set 參數，使用者可以決定 bash 在執行時的行為。
#### **設定參數 :**
>Using + rather than - causes these flags to be turned off. The flags can also be used upon invocation of the shell. The current  set of flags may be found in $-.

開啟要用`-`，但關閉要用`+`，找現在的flag用`$-`。

#### **常用參數 :**
- -u
> Treat unset variables as an error when substituting.
- -e
>Exit immediately if a command exits with a non-zero status.
- -o pipefail
>the return value of a pipeline is the status of the last command to exit with a non-zero status, or zero if no command exited with a non-zero status.

### `local bin="$1"` 、 `local input="$2"` 、 `local prefix="$3"`、 `local op="$4"`在幹嘛 ? 
local為shell中函數內的變數。`$0`為檔案名，`$1`為檔名後面的第一個參數，`$2`為檔名後面的第二個參數。
