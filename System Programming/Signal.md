# 問題集
## Q1 : 為何不寫`sighandler_t signal(sighandler_t handler(int signum));`要寫`sighandler_t signal(int signum, sighandler_t handler);`

你寫`signal(handler(signum))`會直接執行`handler(signum)`，跟`sighandler_t signal(sighandler_t handler(int signum));`的想法背道而馳，他只是想重設handler而已沒叫你執行阿。
## Q2 : user space做違法的事情時，是如何給SIGNAL ? 
#### 觸發階段
cpu執行指令時，會檢查指令是否合法
- Memory Violation : 試圖存取無權限的記憶體位址(dereference NULL pointer、寫入read-only)，**MMU (Memory Management Unit)** 會攔截此操作並觸發 **Page Fault**。
- Illegal instruction : CPU 讀取到一串無法解析的 Binary Code（亂碼或非該架構指令）。
- Arithmetic error : 例如除以零，ALU 單元會觸發 **Divide Error Exception**。

>注意 : page fault是中性的事件，不一定是錯誤，是否錯誤由OS決定。可能表示
> 1. 該頁還沒載入 → OS 會進行 Demand Paging    
> 2. 該頁是 Copy-on-Write → OS 會配置新頁    
> 3. 該頁不存在或沒有權限 → OS 才會判定為 Memory Violation（segfault）
#### 異常處理
CPU 根據異常類型，去查詢 **IDT (Interrupt Descriptor Table)**，找到對應的 Kernel 處理函式（Exception Handler）。
#### 產生訊號：修改 Process 狀態 (Signal Generation)
Kernel 確定要懲罰該 Process 後，會執行以下步驟：
- **鎖定目標：** 找到該 Process 的 PCB (Process Control Block)。    
- **設定 Flag：** 在該 Process 的「待處理訊號位元遮罩」（Pending Signal Mask）中，將對應的訊號位元設為 `1`。
#### 遞送訊號：返回使用者空間前 (Signal Delivery)
在kernel mode返回user mode時，process才會被處理(是在User space處理)，該處理是依照PCB中flag的設定。
#### 整個流程
CPU發現怪怪的，丟到Kernel去看，Kernel仲裁並將結果寫到PCB中，返回user mode，user mode透過PCB決定行為。

---
## Q3 : 在函式`int sigprocmask(int how, const sigset_t *set, sigset_t *oset)` 中如果設oset為NULL會怎樣 ?
在這函式中oset是用來記錄在呼叫這函式之前的sigset，所以你如果把它設為NULL，就單純不紀錄而已。這也可以從函數的變數名稱看出來，set用const，而oset沒用const。

---
## Q4 : `sigaction()`跟`signal()`差在哪，不都在改變disposition嗎 ?
`signal()`是舊標準而`sigaction()`是新規
>`signal()`的缺點 : 移植性差、 race condition、沒有signal mask、資訊較少。
-  移植性差 : 在不同 Unix 系統上實作不同。
- race condition : 在舊式行為中（System V），因為 Handler 觸發後會變回預設值，你必須在 Handler 函式的第一行重新註冊 `signal()`。
- 沒有signal mask : 你無法簡單地控制「在處理 Signal A 時，我要遮蔽 Signal B」，這可能導致nested function calling。
- 資訊較少 : Handler 只能接收一個參數（Signal ID）。你只知道「發生了什麼事」，不知道「是誰做的」。
---
## Q5 : `sigaction()`中`sa_handler`跟`sa_sigaction` 差在哪?
`sa_handle`可想成`sa_sigaction`得簡易版，在`sa_flags` 中沒有設定 `SA_SIGINFO` 旗標。

| 場景                | 推薦使用           | 原因                 |
| ----------------- | -------------- | ------------------ |
| 簡單 signal 處理      | `sa_handler`   | 代碼簡單，兼容性好          |
| 需要知道 signal 來源    | `sa_sigaction` | 可獲取 si_pid, si_uid |
| 處理 SIGSEGV/SIGBUS | `sa_sigaction` | 可獲取故障地址 si_addr    |
| 處理 SIGCHLD        | `sa_sigaction` | 可獲取子進程狀態 si_status |
| 安全關鍵應用            | `sa_sigaction` | 避免惡意 signal 欺騙     |

---
## Q6 : 怎樣去思考"two signals SIGKILL and SIGSTOP cannot be blocked, ignored, or caught by the program's signal handler" ?
- can't be blocked $\rightarrow$  不能一直pending
- can't be ignored $\rightarrow$ 不能不處理
- can't be caught by the program's signal handler $\rightarrow$ 不能自己處理
---
## Q7 : "after calling `sigprocmask()`, if any unblocked signals are pending, at least one of these signals is delivered to the process before `sigprocmask()` reutrns"為何要這樣設計
### 1. 避免「信號飢餓」與無效的解鎖 (Preventing Signal Starvation)
這是最關鍵的原因。在系統程式設計中，一種常見的模式是「原子操作」保護：
1. 阻塞信號（進入臨界區）。    
2. 執行關鍵代碼。    
3. 解除阻塞（短暫開放窗口讓信號進來）。    
4. 再次阻塞信號。   
如果 `sigprocmask()` 在解除阻塞後**不立即**遞送信號就直接返回，程式可能會執行到下一行指令（即第 4 步「再次阻塞」）。
### 2. 確保邏輯上的「即時回應」(Logical Responsiveness)
當程式呼叫 `sigprocmask()` 解除某個信號的阻塞時，這代表程式在對作業系統宣告：「我現在準備好處理這個信號了」。
既然已經準備好了，且信號已經在排隊（Pending），作業系統的邏輯應該是**立刻**滿足這個請求，而不是拖延。
- **Unblock = "Process it NOW"**：如果在解鎖的當下不處理，程式可能會基於「信號尚未發生」的錯誤假設繼續執行後續代碼，這會導致狀態不一致。   
### 3. 符合核心（Kernel）的實作機制
從作業系統核心的實作角度來看，這也是最自然的設計點。
`sigprocmask` 是一個**系統呼叫（System Call）**。
1. 程式進入核心模式（Kernel Mode）修改信號遮罩。    
2. 修改完成後，準備返回使用者模式（User Mode）。    
3. **核心的標準檢查流程：** 在任何系統呼叫即將返回使用者模式的那一刻（Return from Trap/Syscall），核心都會檢查：「當前 Process 是否有未處理的信號，且該信號未被阻塞？」
4. 如果有，核心會修改堆疊（Stack），讓 CPU 跳轉去執行信號處理函式（Signal Handler），而不是直接返回到 `sigprocmask` 的下一行指令。   
因此，這不僅是規範要求，也是 OS 架構中「從核心返回」這一機制的自然副作用。
>是在kernel設完mask後，在跳回User前做檢查

---
## Q8 : 直觀的說`signal()`、`sigprocmask()`、`sigempty()`、`sigaddset()`之間的關係
- `signal()` : 是用來設signal和signal handler之間的應對關係
- `sigempty()`、`sigaddset()`、... : 用來設定signal的集合
- `sigprocmask()` : 設定signal的集合的狀態`SIG_SETMASK`、`SIG_BLOCK`、`SIG_UNBLOCK`。
---
