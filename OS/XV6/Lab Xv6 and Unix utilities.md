# 配置
```txt
riscv64-linux-gnu-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o user/sh.o user/sh.c
user/sh.c: In function 'runcmd':
user/sh.c:58:1: error: infinite recursion detected [-Werror=infinite-recursion]
   58 | runcmd(struct cmd *cmd)
      | ^~~~~~
user/sh.c:89:5: note: recursive call
   89 |     runcmd(rcmd->cmd);
      |     ^~~~~~~~~~~~~~~~~
user/sh.c:109:7: note: recursive call
  109 |       runcmd(pcmd->left);
      |       ^~~~~~~~~~~~~~~~~~
user/sh.c:116:7: note: recursive call
  116 |       runcmd(pcmd->right);
      |       ^~~~~~~~~~~~~~~~~~~
user/sh.c:95:7: note: recursive call
   95 |       runcmd(lcmd->left);
      |       ^~~~~~~~~~~~~~~~~~
user/sh.c:97:5: note: recursive call
   97 |     runcmd(lcmd->right);
      |     ^~~~~~~~~~~~~~~~~~~
user/sh.c:127:7: note: recursive call
  127 |       runcmd(bcmd->cmd);
      |       ^~~~~~~~~~~~~~~~~
cc1: all warnings being treated as errors
make: *** [<builtin>: user/sh.o] Error 1
```
- 問題是 infinite recursion
解法是用
```c
__attribute__((noreturn))
```
這是 GCC/Clang 的一個**函數屬性（Function Attribute）**，用來告訴編譯器：**這個函數永遠不會返回（return）到呼叫者**。

>[!question] 為啥要雙下底線 `__` 跟雙括號 `(())`?
>- `__`: 所有以雙下底線 (`__`) 開頭，或是單下底線加上大寫字母開頭的識別字，都是『保留給編譯器與標準函式庫』使用的。
>- `(())`: 為了騙過早期的編譯器解析器，把多個複雜的屬性「打包」成一個單一區塊來處理。

>[!question] `attribute` 是啥意思?
>提供給編譯器有關變數、函數和類型的更多訊息。
# Sleep
```c
int main(int argc, char *argv[])
```
- `int argc`: argc (argument count) 代表參數的數量，這數量至少為一，代表函式的名子。
- `char *argv[]`: 存儲參數的名稱
	- `argv[0]`: 代表函式的名稱 (或函式的路徑)
	- `argv[1]`: 代表輸入的第一個參數

>[!question] 如果寫 `int main(int a, int b, int c)` 會怎樣?
>不會報錯但這是 undefined behaviour。在 C/C++ 對於 `main` 只允許兩個合法的函式簽名
>```c
>int main(void) 
>int main(int argc, char *argv[])
>```

```c
int atoi(const char *s)
{
int n;
n = 0;
while('0' <= *s && *s <= '9')
	n = n*10 + *s++ - '0';
return n;
}
```
這邊可以看到
 - `! ~ ++ -- + - * & (type) sizeof`: 一元運算（邏輯非、位元取反、自增減、取正負、取地址、指標、型別轉換、大小）是右至左
 另外這程式是有以下問題的
 - 遇到前面的空白沒跳過
 - 負號沒處理
 - 在讀到數字後的非數字後沒停止
 
>[!question] 如何從 user mode 切到 kernel mode?
>![[Pasted image 20260511163200.png]]
>### Step 1: User process 準備 ecall
>呼叫 `write(fd, buf, n)` 這類 libc 函式時，實際上會跳到 `usys.S` 自動生成的組語
>```asm
>write:
>	li a7, SYS_write   # syscall number 存入 a7
>	ecall              # 觸發 trap
>	ret
>```
>而參數依照 RISC-V calling convention 放在 `a0~a5`
>### Step 2: RISC-V 硬體自動
>執行 `ecall` 後，CPU 在**一個 cycle 內**完成:
>- `sepc` (supervisor exception program counter): 當 trap 進入 S-mode 時，硬體會將 U-mode 當下的 pc 存在 `sepc` 等到 kernel 處理完執行 `sret`(supervisor return) 後跳回這儲存的 PC
>- `scause` (supervisor cause register): 紀錄 trap 發生 (可能是 interrupt 或是 exception) 原因的暫存器。
>- `sstatus` (supervisor status register): 這是一個追蹤和控制 CPU 核心狀態的綜合暫存器。
>	- `sstatus.SPP` (supervisor previous privilege): 紀錄發生 Trap **之前**的特權層級。在 trap 時你可能在 S-mode 或是 U-mode (在 S-mode 有可遇到 exception，exception 是硬體執行的致命或是必要事件，無法被屏蔽)。
>	- `sstatus.SIE` (supervisor interrupt enalbe): S-mode 的全域中斷啟用開關 (1: 勇許中斷，0: 遮蔽中斷)。這防止 S-mode 處理中斷時，被新的中斷打斷，使得 `sepc` 被覆蓋。
>- PC 跳至 `stvec` (supervisor trap vector bass address register)，`stvec` 是在開機初始化，或是準備切換到 user mode 之前由 OS 設定好的。這邊的 `stvec` 設定為 trampoline 的 `uservec`。
>### Step 3: 切換 user page table 到 kernel page table
>XV6 把 `trampoline.S` 這一頁同時映射在 user 和 kernel 的虛擬位置最高處 (`TRAMPOLINE = 0x3fffff000`) ，而 `uservec` 做
>```asm
># 先用 sscratch 暫存 a0，騰出一個暫存器
>csrrw a0, sscratch, a0
># a0 現在指向 trapframe（也是固定映射在 user 頁表）
sd ra,  0(a0)
sd sp,  8(a0)
...                   # 把 32 個暫存器全存入 trapframe
># 切換頁表
ld t1, 56(a0)         # trapframe 裡存的 kernel_satp
csrw satp, t1
sfence.vma zero, zero # 清 TLB
.# 跳入 kernel C code
ld t0, 16(a0)         # usertrap 的函式指標
jr t0
>```
>注意這邊是透過 `satp` (Supervisor Address Translation and Protection) 做 S-mode 下的 page table 轉換和維持。 另外在轉換到 kernel 前要把 user process 的32個 register (ra: return address, sp: user stack pointer, t1/t2: 臨時值) 存下來。
>### 後面的 Steps 看圖

>[!question] 為啥 RISC-V 不要把 `ecall` 跟 `csrw satp, t1` 綁在一起，不是要從 user 切到 kernel 了?
>RISC-V 的設計哲學:「硬體只做必要的事」，所以 `ecall` 只做: 存 PC、存切換原因、切 mode、跳 `strvec`。連 stack 跟 page table 都不換。我們可以自由的決定如果是 user 切到 kernel 換 sp (這時 kerenl stack 是空的，所以直接切到頂沒差)，而 kernel 內發生 exception 的 sp 不切 (kernel 裡的 exception 不能切 SP，因為 kernel stack 是非空，直接切會讓 kernel stack 變空的，使得之前在 stack 中的值被覆蓋)。如果不切 page table (即不換 `satp`)，如果把 OS 的 kernel 或是例外處理都跟 user 用同一張頁表，syscall 可以省掉兩次 `satp` 切換 (進出 kernel)。

>[!note] Interrupt 跟 Exception 差別
>#### Interrupt
>由外部硬體 (網卡、鍵盤、Timer 等) 來觸發，可在任何時候發生，無關乎當前指令，而且可遮蔽，例子有: Timer interrupt、externel interrupt (Crtl+C) 等。主要是用來處理 I/O 事件、多工作業系統。
>- Crtl+C: 是一個普通的鍵盤外部中斷，但它的本質是 linux 終端子系統 (TTY) 攔截特定按鍵後，生成並傳遞給前景程式的軟體訊號 (`SIGINT`)。
>#### Exception
>由內部 CPU (當下正在執行的指令) 來觸發，必定在執行某個特定指令時觸發，**不可遮蔽**，例子有: page fault、illegal instruction、divide by zero 等。主要是用來處理軟體錯誤、記憶體管理、系統呼叫等。
>#### `signal_handler`
>`signal_handler` 是在 OS 的軟體層面模擬出來的軟體中斷 (software interrupt)，觸發可能來自於 exception、interrupt 或是軟體 (system call) 的 Signal，`kill` 指令是軟體的 signal 的例子。OS 把 exception、interrupt 或是軟體的 Signal ，包裝成標準的 POSIX Signal ，再轉發給 user-mode 的應用程式。
# Pingpong
## pipe
讓 kernel 配置環形緩衝區，並給呼叫形成兩個 fd
- `fd[0]`: 讀端；`fd[1]`: 寫端 (`write()` 寫入小於 `PIPE_BUF`（4096 bytes）時是原子的)
pipe 預設為  blocking 的模式，但可以透過以下方式設定為 `O_NONBLOCK`
```C
// 將讀取端設為非阻塞
int flags = fcntl(fd[0], F_GETFL, 0);
fcntl(fd[0], F_SETFL, flags | O_NONBLOCK);
```
預設的**阻塞 (blocking)** 讓緩衝區
- pipe 為空時讀取: 該進程會被掛起休眠，等到有人寫入才被喚醒
- pipe 滿時寫入: 該進程會被掛起休眠，等到有人讀取騰出空間後才喚醒
端點關閉
- 所有寫端關閉後，`read()` 回傳 `0`，表示 EOF
- 所有讀端關閉後，`write()` 觸發 `SIGPIPE`；若訊號被攔截則回傳 `-1`，`errno` 為 `EPIPE`
**非阻塞 (Non-blocking)**
- pipe 為空時讀取: `read()` 會回傳 -1，並接錯誤碼 (`errno`) 設為 `EAGAIN` 或是 `EWOULDBLOCK` ，進程可以去做其他事。
- pipe 滿時寫入: `write()` 會回傳 -1，並接錯誤碼 (`errno`) 設為 `EAGAIN`，進程可以去做其他事。
端點關閉: **行為同 blocking**。
>[!important] 讀跟寫想法
>這邊的讀或寫是 partial 的，無法保證一次全寫完，所以常用策略
>```C
>ssize_t total = 0;
>while (total < expected) {
>    ssize_t n = read(fd[0], buf + total, expected - total);
>    if (n == 0) break;       // EOF，寫端全關
>    if (n < 0) {
>        if (errno == EAGAIN) continue;  // non-blocking 暫時沒資料
>        // 其他錯誤處理
>        break;
>    }
>    total += n;
>}
>```
>確定讀或寫完，才去幹下一件事，這裡讀或寫端全關閉只會影響 `read` 跟 `write` 的回傳值。
>如果真的要異步去做
>-  `select` / `poll` / `epoll`: 同時監聽多個 fd，有資料才去讀，中間可以處理其他事件。
>- `io_uring`: Linux 真正的 async I/O，submit 請求後立刻去做別的，完成後再收結果。
>- 多執行緒讓另一條 thread 負責阻塞讀，主 thread 繼續跑。

>[!note] blocking 想法
>blocking 想描述兩件事。**緩衝區狀態**（空/滿時如何反應）以及**端點狀態**（對端關閉時如何反應）。須注意 blocking 跟 non-blocking 端點關閉行為相同，**差別只在緩衝區**。
# Primes



# Find

>[!question] 在 C 中比較兩字串用 `==` 會怎樣?
>

>[!question] 為啥以下程式要這樣寫
```c=
void* memmove(void *vdst, const void *vsrc, int n)
{
	char *dst;
	const char *src; 
	dst = vdst;
	src = vsrc;

	if (src > dst) {
		while(n-- > 0)
			*dst++ = *src++;
	} else {
		dst += n;
		src += n;

		while(n-- > 0)
			*--dst = *--src;
	}
	return vdst;
}
```
這樣寫得關鍵觀察是:「我們無法在一個操作下把 `vsrc` 搬到 `vdst`

>[!question] 為啥以下程式可以這樣寫
```C
char* strcpy(char *s, const char *t)
{
	char *os;
	os = s;
	while((*s++ = *t++) != 0)
		;

	return os;
}
```

# Xargs





