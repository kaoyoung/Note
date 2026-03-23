# Q1 : unimitialize的global不是在賦值時才給記憶體，那時heap位置不會跑掉嗎
你搞錯順序了。在硬碟上時為了節省空間，編譯器不會在執行檔裡真的塞一堆 0，只會在檔頭（Header）記錄，但一但載到記憶體開始執行時，記憶體會立刻分配記憶體給它，並初始為0。

---
# Q2 : 設定`longjmp(jump_buf emv, int val)`中的val為0會怎樣 ?
我們去看manual page [longjump(3)](https://linux.die.net/man/3/longjmp) ，他說
>If **longjmp**() is invoked with a second argument of 0, 1 will be returned instead.

系統會強制return 1，所以我們可以知
>longjump() returns a nonzero value if returning from a call to longjump()

或是反過來想，我們需要知道longjump()從哪裡回來，所以強制call回來非0。

---
# Q3 : 如果有多個sigsetjmp那siglongjmp後跳回哪一個
`siglongjmp` 會跳回哪一個 `sigsetjmp`，完全取決於你呼叫 `siglongjmp` 時傳入的那個 `env` 參數（`sigjmp_buf` 變數）。它並不是依照「時間順序」或「堆疊深淺」來自動判斷，而是**依照你指定的「存檔點」**。

---
