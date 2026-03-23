## Q1 : Buffered I/O 和 Unbuffered I/O 差在哪，各有啥優缺 ? 
## A1 :
- **Buffered I/O :** 標準 I/O 函式庫（如 `printf`, `fwrite`, `fread`）。
- **Unbuffered I/O :** 系統呼叫（System Calls，如 `write`, `read`）。
### 機制比較 : 
- **Buffered I/O :**  應用程式 $\rightarrow$ User Buffer $\rightarrow$ Kernel $\rightarrow$ 硬體
- **Unbuffered I/O :** 應用程式 $\rightarrow$ Kernel $\rightarrow$ 硬體
從這邊就可以直觀看出差異，buffered I/O多一層user buffer，使其可以暫時將資料屯在buffer中，不急著寫回硬體，如此節省跟硬體(disk)I/O的時間，但弊端也在這，buffer在RAM中，一斷電你就炸了，資料完全消失。
### 優缺比較
#### buffered I/O :
##### 優點 (Pros)
- **效能較高（吞吐量大）:** 這是最大的優點。因為系統呼叫（從使用者模式切換到核心模式）的開銷很大。將多次小的讀寫合併成一次大的讀寫，能大幅降低 CPU 的負擔。    
- **程式設計簡便:** 程式設計師不需要擔心最佳的區塊大小（Block size），標準函式庫會自動處理細節。    
- **硬體友善:** 減少對磁碟的頻繁零碎存取。    
##### 缺點 (Cons)
- **資料一致性風險:** 如果程式崩潰（Crash）或斷電，停留在 Buffer 中尚未刷新到核心的資料將會遺失。    
- **即時性較差:** 資料寫入後不會立刻出現在目標檔案或裝置上，會有延遲。    
- **記憶體佔用:** 需要額外的記憶體空間來維護 Buffer。
#### Unbuffered I/O :
##### 優點 (Pros)
- **即時性高:** 資料一旦寫入，就立即進入作業系統的管理範圍，適合需要精確時間控制的場景。
- **資料安全性較高:** 在程式崩潰時，因為資料沒有滯留在使用者空間的 Buffer，遺失資料的風險較小（尤其是 Log 紀錄）。    
- **精確控制:** 允許程式設計師確切知道何時發生 I/O，適合資料庫交易（Transaction）或即時互動裝置。   
#### 缺點 (Cons)
- **效能較低（對於頻繁小量傳輸）:** 如果你寫一個迴圈，每次只寫 1 byte，使用 Unbuffered I/O 會導致成千上萬次系統呼叫，導致 CPU 資源浪費嚴重，速度極慢。    
- **程式複雜度:** 開發者需要自己管理讀寫的區塊大小以優化效能。

### 注意 : kernel內也有buffer
不管在buffer i/o或是unbuffer i/o都需要經過kernel，而kernel也有buffer也就是有斷電的風險。想完全規避需要使用 `write()` (Unbuffered I/O) 寫入資料後，**緊接著** 呼叫 `fsync(fd)`，或是在開啟檔案時使用特殊旗標 `open(..., O_DIRECT)`。

---
## Q2 : 你存資料需要存甚麼資訊
## A2 :
- 資料的存取方式 $\rightarrow$ file metadate(file permission、position、...)
- 資料的內容 $\rightarrow$ file content

---
## Q3 : 怎樣去理解file descriptor
## A3 :
基本設置 : Unix associates the numbers 0, 1, and 2 with standard input, standard output, and standard error, respectively
![[Pasted image 20251216164233.png]]
每個process都有各自的open file descriptor table，而kernel維持一個open file table。

---
## Q4 : 為何`umask()` 不是unmask?
## A4 : 
要用user mask去想`umask()` ，而不是unmask。
![[Pasted image 20251216173802.png]]

---
## Q5 : 為什麼說`creat()` is mostly obsolete ? 
## A5 : 
### 第一 : 可以用`open(path, O_WRONLY | O_CREAT | OTRUNC, mode)`替代
### 第二 : 只能「唯寫 (Write-Only)」
### 第三 : 太過霸道，沒用`O_EXCL`
- 如果檔案不存在 $\rightarrow$ 建立。    
- 如果檔案存在 $\rightarrow$ **直接清空內容 (Truncate)** 並覆寫。
O_TRUNC惹的禍
![[Pasted image 20251216181535.png]]

---
## Q6 : File I/O 相關資訊是如何被儲存的
## A6 : 
![[Pasted image 20251217003537.png]]
The Unix OS kernel uses three data structures to represent an open file
- Open file descriptor table (per process)
- Open file table (shared for all open files in the system) 
- V-node table (shared for all open files in the system)
### Open file descriptor table; one entry per file descriptor; each entry contains: 
- The **file descriptor flag** (I will discuss it later)
- A **pointe**r to a system open file table entry
### Open file table; each entry contains: 
-  The **file status flag** for the file (readable/writable/append/sync/nonblocking) 
-  The current **file offset** (I will discuss it later) 
-  A **pointer** to the v-node table entry for the file 
### V-node (i-node) table; each entry contains a V-node data structure that contains: 
- The **pointer** to the i-node structure of the respective file 
-  **V-node information**
### V-node is an in-memory structure for each open file: 
-  Invented to support multiple file system types on a single computer system 
-  V-node information: the type of file and pointers to functions that operate on the file 
### I-node is both stored physically on the storage device and in memory: 
- Contains the **metadata about the file**: file owner, file size, residing device, protection information, and locations of the data blocks comprising a file, .. (you see some of them from “ls”) 
-  The OS kernel reads the I-node from the disk to memory when the associated file is opened; 
![[Pasted image 20251217004149.png]]









