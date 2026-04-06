## Q1 : thread control中的reap和kill差在哪 ?
## A1 : 
> **reap** : to cut and collect a grain crop
### 1. Kill：發送訊號 (The `kill()` System Call)
在底層，`kil` 並不一定代表「殺死」，它的==本質是**發送訊號（Signal）**==。
- **系統調用：** `int kill(pid_t pid, int sig);`
- **運作原理：**
    - 當你執行 `kill(pid, SIGKILL)`，核心（Kernel）會強行停止該 PID 的執行。
    - 當你執行 `kill(pid, SIGTERM)`，則是禮貌性地請進程結束，給它機會清理資源。
- **結果：** 進程停止運行了，但它的 **PCB (Process Control Block)** 仍然留在核心的進程表（Process Table）中。此時，進程進入 **Zombie (Z) 狀態**。
### 2. Reap：回收狀態 (The `wait()` System Call)
「Reaping」並不是一個單一的指令，而是==一個**回收動作**==，通常透過 `wait` 系列的 System Call 完成。
- **系統調用：** `pid_t wait(int *wstatus);` 或 `pid_t waitpid(pid_t pid, int *wstatus, int options);`
- **運作原理：**
    - 父進程調用 `wait()` 時，核心會把子進程的結束狀態（Exit Status）交給父進程。        
    - 一旦狀態被讀取，核心就會把該子進程從進程表中徹底刪除（這就是「收屍」）。        
- **結果：** PID 被釋放，系統資源完全回收。    
### 3. 進程生命週期圖解
理解這兩者差異的最佳方式是看進程的狀態轉移：
1. **Running $\rightarrow$ Zombie:** 透過 `exit()` 或是收到 `SIGKILL`（被 Kill）。    
2. **Zombie $\rightarrow$ Gone:** 透過父進程執行 `wait()`（被 Reap）。
---
## Q2 : detach的是在哪被回收 ?
## A2 : 
### 執行緒的 Detach (`pthread_detach`)
在==預設情況下，執行緒是 **Joinable** 的==。這意味著它死後會變成「殭屍執行緒」，必須有人呼叫 `pthread_join()` 來 Reap 它。
但如果你呼叫了 `pthread_detach()`：
- **誰來回收？**：**作業系統的核心 (Kernel)** 或 **執行緒函式庫 (Threading Library)**。    
- **何時回收？**：當該執行緒執行完畢（回傳或呼叫 `pthread_exit()`）的**那一瞬間**。    
- **回收了什麼？**：系統會立即釋放它的堆疊（Stack）、執行緒控制塊（TCB）以及所有相關的系統資源。   
> **重點：** 一旦 Detach，你就再也不能對它使用 `pthread_join()`，因為它在死掉的當下就已經被系統徹底抹除了，沒有「屍體」可以讓你讀取狀態。
--- 
## Q3 : 為何下面的程式要pthread_attr_destory()，這邊為local變數你不destory他自己會消失阿 ? 
```c
#include <pthread.h>
#include "apue.h"  

int makethread(void *(*fn)(void *), void *arg){
	int err;
	pthread_t tid;
	pthread_attr_t attr;  

	err = pthread_attr_init(&attr);
	if(err != 0)
		return(err);
	err = pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
	if(err == 0)
		err = pthread_create(&tid, &attr, fn, arg);
	pthread_attr_destory(&attr);
	return(err);
}
```
## A3 : 
### 1. 隱藏的動態記憶體分配 (Internal Memory Allocation)
`pthread_attr_t` 是一個結構體，但它的內部實作是隱藏的（Implementation-defined）。
- 當你呼叫 `pthread_attr_init(&attr)` 時，函式庫可能會在**堆積 (Heap)** 上分配額外的空間來儲存執行緒的屬性（例如 Stack Size, Guard Size 等）。    
- 雖然 `attr` 這個變數是在 **Stack** 上，但它內部的指標可能指向了 **Heap** 上的記憶體。    
- 如果你不呼叫 `pthread_attr_destroy`，這塊內部的 Heap 記憶體就會發生 **Memory Leak（記憶體洩漏）**。    
### 2. 資源回收的對稱性 (API Symmetry)
在 Unix System Call 的設計哲學中，==幾乎所有的 `init` 或 `alloc` 動作都必須配對一個 `destroy` 或 `free`==：
- `malloc()` $\rightarrow$ `free()`    
- `open()` $\rightarrow$ `close()`    
- `pthread_mutex_init()` $\rightarrow$ `pthread_mutex_destroy()`    
- **`pthread_attr_init()` $\rightarrow$ `pthread_attr_destroy()`**
即使目前的 Linux 實作（如 NPTL）中 `pthread_attr_destroy` 可能只是把某個欄位設為 0，但在其他的 Unix 系統（如 Solaris 或舊版 AIX）中，它可能涉及複雜的資源釋放。為了**可移植性 (Portability)**，你必須呼叫它。
---
## Q4 : 怎樣去理解thread cancellation中的cancellation point ? 
## A4 : 
理解 **Thread Cancellation Point（取消點）**，最直覺的方式是把它想像成程式碼中的**檢查哨**。
在多執行緒環境下，如果你叫一個執行緒「立刻自殺」（透過 `pthread_cancel`），這其實是非常危險的。如果執行緒正好寫到檔案一半、或正鎖著一個 Mutex，直接死掉會導致檔案毀損或系統死鎖。因此，Unix 設計了 **Deferred Cancellation（延遲取消）** 機制：執行緒收到取消請求後不會立刻死掉，而是繼續跑，直到它跑到了某個「安全的地點」，才會停下來檢查並結束。這個地點就是 **Cancellation Point**。
### 哪些地方是 Cancellation Point？
在 Unix-like 系統中，並非所有程式碼都是取消點。取消點主要分為兩類：
#### A. 隱性取消點 (Implicit)
大多數會導致==執行緒**阻塞（Blocking）或等待**的 System Call==，都被定義為取消點。因為執行緒在等人的時候最適合「被收掉」。 常見的包括：
- `read()`, `write()` (對檔案或 Socket 的操作)    
- `sleep()`, `nanosleep()`    
- `wait()`, `waitpid()`    
- `pthread_cond_wait()`
- `select()`, `poll()`
#### B. 顯性取消點 (Explicit)
如果你的程式碼是一個==純計算的迴圈（例如 `while(1) { i++; }`），裡面沒有任何 System Call，這個執行緒可能**永遠不會死**==，因為它遇不到取消點。 這時你必須手動加入檢查哨：
```c
while (1) {
    // 進行大量計算...
    
    // 手動設置檢查哨
    pthread_testcancel(); 
}
```
`pthread_testcancel()` 的唯一作用就是：檢查現在有沒有人要我取消？有的話，就在這裡結束。

---
## Q5 : 什麼是spurious wakeup ? 
## A5 : 
> **Spurious Wakeup** 是指一個執行緒在沒有被正式通知（Signal 或 Broadcast）的情況下，或是它所等待的條件尚未滿足時，就從「等待（Wait）」狀態中醒來的現象。
## 為什麼會發生虛假喚醒？
這聽起來像是系統的 Bug，但實際上是基於作業系統效能與實作複雜度的折衷設計。主要原因包括：
1. **效能考量 (Performance)**： 在多核心系統上，如果要保證每一次喚醒都百分之百「精準且條件成立」，會對作業系統的核心（Kernel）造成巨大的同步負擔。為了提升執行效率，作業系統允許這種極少數的「誤報」。    
2. **競爭條件 (Race Conditions)**： 假設執行緒 A 被喚醒了，但在 A 重新獲得鎖（Mutex）並開始執行之前，執行緒 B 突然介入並修改了條件（例如把唯一的資源拿走了）。這對 A 來說，醒來時條件又不滿足了，這也算是一種虛假喚醒。    
3. **訊號中斷 (Interrupted System Calls)**： 在 Linux 等系統中，底層的系統調用（如 `futex`）可能會因為收到作業系統訊號（Signal）而被迫中斷並返回，導致執行緒醒來。
## 核心解決法：永遠使用 `while` 而非 `if`
這是處理虛假喚醒的「黃金法則」。你==不能假設醒來就代表條件一定成立==。

---
## Q6 : 在處理superious wakeup中，`while`和`if`差在哪 ?
## A6 :
### `while`
```c
while (apple_count == 0) {         // <--- 1. 檢查條件
    pthread_cond_wait(&cond, &lock); // <--- 2. 停在這裡！(等別人 signal)
                                     // <--- 3. 醒來並搶到鎖後，從這裡「返回」
}                                    // <--- 4. 碰到大括號，自動跳回「1」
// -------------------------------------------------------------------
do_something();                    // <--- 5. 只有條件不成立(有蘋果)才會到這
```
#### `if`
```c
if (apple_count == 0) {
    pthread_cond_wait(&cond, &lock); // <--- 2. 停在這裡
                                     // <--- 3. 醒來後返回
}                                    // <--- 4. 離開 if 區塊
do_something();                      // <--- 5. 直接執行！(不管有沒有蘋果)
```