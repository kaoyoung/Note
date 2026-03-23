# Q1 : 為何"ln -s test1 test2"後"cat test2"不是顯示test1的位置
先回顧`ln -s`指令，它是創造一個symbolic link，語法是

```md
ln -s <existing source> <new/destination>
```

>注意`ln cp mv`語法格式都一樣為 : 
>`ln/cp/mv <existing source> <new/destination>`

這個問題是天真地以為`cat test2`會去看test2這檔案的內容，但linux會知道它是軟連結而自動跳到test1，再進去看內容。如果想看test2的內容可以`ls -l`或`readlink` 。

---
# Q2 : `>`跟`>>`有什麼差，`<`跟`<<`有什麼差
回顧資料流重導向的過程
![[data stream redirection.png]]
數字代表意思
- 0 : 標準輸入(stdin)
- 1 : 標準輸出(stdout)
- 2 : 標準錯誤輸出(stderr)
符號意思
- `>` : **輸出**重導向，會**覆蓋**舊檔案，寫入新內容。
- `>>` : **輸出**重導向，會**保留**舊檔案，寫入新內容道原本檔案的後面。
- `<` : **輸入**重導向，會**覆蓋**舊檔案，寫入新內容。
- `<<` : **輸入**重導向，會**保留**舊檔案，寫入新內容道原本檔案的後面。
**例子** : 
-  `<` 的例子 : 
```bash
# 一般用法： cat 讀取檔案並顯示
cat file.txt

# 重導向用法： 把 file.txt 的內容「餵」給 cat
cat < file.txt
# 結果跟上面一樣，但在某些程式（如 mysql）這個語法很重要
# 例如： mysql -u root -p < backup.sql (把 SQL 檔餵給資料庫執行)
```
- `<<` 的例子 : 
```bash
# 告訴 cat：我要開始打字了，直到我輸入 "EOF" 這三個字你才準停 
cat << EOF 
這是第一行 
這是第二行 
EOF 

# 執行後，螢幕會顯示： 
# 這是第一行 
# 這是第二行
```

---
# Q3 : 為何`./sample.sh`可以執行但`sample.sh`不行?
因為`sample.sh`會去環境變數(可由`echo $PATH`去看)找`sample.sh`找這指令，但`sample.sh`在當前資料夾，而當前資料夾不再環境變數中，所以無法辨識。
### 為何不把當前目錄加入環境變數?
不把當前目錄加入環境變數，是為了安全的緣故，如果隨便當前目錄加入環境變數，可能造成有人故意在當前目錄寫個`ls`來稿別人。

---