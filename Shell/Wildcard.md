# Q1 :  How you would copy all the files from `/tmp/a` into `/tmp/b`. All the .txt files? All the .html files?

# A1 : 
```shell
$ cp /tmp/a/* /tmp/b/
$ cp /tmp/a/*.txt /tmp/b/
$ cp /tmp/a/*.html /tmp/b/
```
---
## Q2 : How would you list the files in `/tmp/a/` without using `ls /tmp/a/`?
## A2 : 
```shell 
$ echo /tmp/a/*
```
當你在終端機輸入這個指令時，並不是 `echo` 指令去尋找檔案，而是**殼層 (Shell，例如 Bash 或 Zsh)** 在執行指令前先動了手腳：
1. **展開 (Expansion)**：Shell 看到 `*`，會先去查看 `/tmp` 底下有哪些檔案（假設有 `a.txt` 和 `b.log`）。    
2. **替換**：Shell 會把 `/tmp/*` 替換成 `/tmp/a.txt /tmp/b.log`。    
3. **執行**：最後真正執行的指令其實是 `echo /tmp/a.txt /tmp/b.log`。
---
## Q3 : How could you rename all .txt files to .bak? Note that ?
## A3 : 
```shell=
$ mv *.txt *.bak
```
這個指令**通常不會如你預期地**把所有 `.txt` 變成 `.bak`。在==大多數情況下，它會**報錯**，或者產生讓你意想不到的結果==。
### 為什麼它不能運作？
當你輸入 `mv *.txt *.bak` 時，Shell 會在執行 `mv` 之前先展開萬用字元。
假設你的資料夾裡有：`a.txt`、`b.txt`、`c.txt`。
1. **展開階段**：Shell 會把 `*.txt` 變成 `a.txt b.txt c.txt`。    
2. **處理目標**：如果目前沒有任何 `.bak` 檔案，`*.bak` 會被當作**純字串**（在某些設定下）。
3. **最終執行**：指令變成了 `mv a.txt b.txt c.txt *.bak`。
---
