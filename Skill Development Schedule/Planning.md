# 寒假結束前

## 計算機網路
- https://csdiy.wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/topdown_ustc/
- https://csdiy.wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/CS144/#_1
	- 看一下他們怎麼處理多線呈伺服器
## system programming
- [How to Make a Multiprocessor Computer That Correctly Executes Multiprocess Programs](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=1675439)
- Rewrite the sp homework
- linux內核開發那本書(Linux系統編程(第二版))
- trainning
	 ### Level 1: 指標與記憶體的絕對掌控 (The Memory Master)
	**目標**：消除對指標運算的恐懼，理解 C 語言字串的本質（Null-terminated string），以及 Stack vs Heap 的生命週期。

	#### 任務 1.1：重造 `string.h`

不要使用標準函式庫，請實作以下函式。這能訓練你對「邊界條件」的敏感度。

- `size_t my_strlen(const char *s);`
    
    - **挑戰**：用 `while` 迴圈掃描直到 `\0`。嘗試優化它（例如一次讀 4 bytes，雖然初學者不用這麼深，但可以思考）。
        
- `char *my_strcpy(char *dest, const char *src);`
    
    - **挑戰**：回傳值應該是什麼？為什麼要回傳 `dest`？
        
- `void *my_memcpy(void *dest, const void *src, size_t n);`
    
    - **挑戰**：這是最關鍵的一題。如果 `dest` 和 `src` 的記憶體重疊（Overlap）了怎麼辦？（這其實是 `memmove` 的功能，但請試著理解為什麼單純的 `memcpy` 在重疊時會壞掉）。
        
    - **練習重點**：使用 `void *` 轉型成 `char *` 進行 byte-level copy。
        

	#### 任務 1.2：動態陣列 (Dynamic Array / Vector)

實作一個可以自動擴展大小的整數陣列。

- **API**：
    
    - `init(size_t initial_capacity)`
        
    - `push_back(int value)` -> 當容量不夠時，用 `realloc` 擴大成原本的 2 倍。
        
    - `get(size_t index)`
        
    - `free_array()`
        
- **關鍵訓練**：理解 `malloc`, `realloc`, `free` 的運作，以及如何處理 Memory Leak（使用 Valgrind 檢查）。
    

---

	### Level 2: 與作業系統對話 (The System Interface)

**目標**：理解 System Calls (系統呼叫)。這是 User Space 進入 Kernel Space 的入口。你需要習慣閱讀 `man 2` (System calls) 文件。

#### 任務 2.1：最小化 Shell (Mini Shell)

寫一個無窮迴圈程式，印出 `myshell>` ，讀取使用者輸入並執行。

- **階段一：執行簡單指令**
    
    - 使用 `fgets` 讀取輸入。
        
    - 使用 `strtok` 切割字串（解析 `ls`, `-l`, `/tmp`）。
        
    - **核心機制**：`fork()` 產生子行程 -> 子行程呼叫 `execvp()` 變身為目標程式 -> 父行程呼叫 `waitpid()` 等待子行程結束。
        
    - **圖解概念**：你需要腦中浮現 Process Tree 的分岔與合併。
        
- **階段二：IO Redirection (輸出重導向)**
    
    - 支援指令：`ls -l > out.txt`
        
    - **核心機制**：在 `execvp` 之前，使用 `open()` 開啟檔案，並用 `dup2()` 將 File Descriptor 1 (STDOUT) 替換成該檔案的 FD。
        
- **階段三：Pipes (管線) —— 這是大魔王關卡**
    
    - 支援指令：`ls | grep .c`
        
    - **核心機制**：`pipe()` 建立管線，連通兩個子行程的 STDIN 和 STDOUT。這需要精準的 FD 操作，非常考驗邏輯。
        

---

	### Level 3: 平行處理與同步 (The Concurrency Expert)
	**目標**：處理 Race Condition (競爭危害) 和 Deadlock (死結)。這是現代後端與系統開發最值錢的技能。
	#### 任務 3.1：安全佇列 (Thread-safe Queue)

寫一個 Queue，讓多個 Thread 同時塞資料（Producer）和拿資料（Consumer）也不會壞掉。

- **需求**：
    
    - 使用 `pthread_mutex_t` 保護 Queue 的內部結構。
        
    - 使用 `pthread_cond_t` (Condition Variable) 實作等待機制：當 Queue 空了，Consumer 要睡覺等待；當 Queue 滿了，Producer 要睡覺等待。
        
- **測試**：開 10 個 Producer thread 和 10 個 Consumer thread 同時狂跑，確保最後資料數量正確，沒有 Segfault。
    

	#### 任務 3.2：執行緒池 (Thread Pool)

這是系統程式的期末考。你將建立一組固定的 Workers 隨時待命。

- **架構**：
    
    1. **Task Queue**：存放待執行的任務（函式指標 + 參數）。
        
    2. **Worker Threads**：啟動 N 個 Thread，無窮迴圈從 Queue 拿任務執行。
        
    3. **Submit Function**：主程式呼叫 `pool_submit(function, arg)` 將任務丟進 Queue。
        
- **為什麼難**：你需要處理「優雅關閉」(Graceful Shutdown)。當我呼叫 `pool_destroy()` 時，如何通知所有正在睡覺等待任務的 Thread 起床並安全結束？
    

---

### 執行建議

1. **環境設定**：請在 Linux 環境下開發（VirtualBox 裝 Ubuntu 或使用 WSL2）。不要用 IDE 的 Run 按鈕，請學會寫 `Makefile`，並在 Terminal 用 `gcc` 編譯。
    
2. **遇到不懂的函式**：在 Terminal 輸入 `man 3 函式名` (Standard Library) 或 `man 2 函式名` (System Call)。例如 `man 2 fork`。這才是系統工程師的標準查法。
    
3. **檢驗標準**：
    
    - 程式不能 Crash。
        
    - 用 `valgrind --leak-check=full ./your_program` 執行，必須看到 "0 errors from 0 contexts"。
## 操作系統 
- https://csdiy.wiki/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/MIT6.S081/#_1

## leetcode (記得去看靈神的解說與分類)
- 滑動窗口 
- 雙指針  
- 枚舉 
- 前綴和
- 差分
- 鏈表(用c寫)
- 二叉樹(用c寫) 

# programming language 
- assembly
	- 王爽assembly前十章  
- c
	- 你所不知道的c語言(jserv)
		- [為什麼要深入學習 C 語言？](https://hackmd.io/@sysprog/c-standards)  
		- [指標篇](https://hackmd.io/@sysprog/c-pointer)  
		- [函式呼叫篇](https://hackmd.io/@sysprog/c-function) 
		- [遞迴呼叫篇](https://hackmd.io/@sysprog/c-recursion)  
		- [記憶體管理、對齊及硬體特性](https://hackmd.io/@sysprog/c-memory)  
		- [物件導向程式設計篇](https://hackmd.io/@sysprog/c-oop)
		- [前置處理器應用篇](https://hackmd.io/@sysprog/c-preprocessor)
		- [動態連結器](https://hackmd.io/@sysprog/c-dynamic-linkage)
		- [goto 和流程控制](https://hackmd.io/@sysprog/c-control-flow)
		- [linked list 和非連續記憶體操作](https://hackmd.io/@sysprog/c-linked-list)
		- [技巧篇](https://hackmd.io/@sysprog/c-trick)
	- https://csdiy.wiki/%E7%BC%96%E7%A8%8B%E5%85%A5%E9%97%A8/C/Duke-Coursera-Intro-C/
- c++
	- c++ primer (stl函數和容器、迭代器、智能指針、class)  
	- https://google.github.io/styleguide/cppguide.html#Classes
		- classes
		- functions
		- naming
		- comments
	- https://csdiy.wiki/%E7%BC%96%E7%A8%8B%E5%85%A5%E9%97%A8/cpp/AUT1400/#_1 
	- https://csdiy.wiki/%E7%BC%96%E7%A8%8B%E5%85%A5%E9%97%A8/cpp/CS106B_CS106X/
	- [Yui Huang 演算法學習筆記](https://yuihuang.com/) 
		- Unit 8 , done
			- [a130: 12015 – Google is Feeling Lucky](https://zerojudge.tw/ShowProblem?problemid=a130), done
			- [b428: 凱薩加密](https://zerojudge.tw/ShowProblem?problemid=b428) ,done
			- [a065: 提款卡密碼](https://zerojudge.tw/ShowProblem?problemid=a065)  ,done
			- [d235: 10929 – You can say 11](https://zerojudge.tw/ShowProblem?problemid=d235) , done
			- [d275: 11586 – Train Tracks](https://zerojudge.tw/ShowProblem?problemid=d275) , done
			- [c459: 2. 自戀數](https://zerojudge.tw/ShowProblem?problemid=c459) , done
			- [d267: 11577 – Letter Frequency](https://zerojudge.tw/ShowProblem?problemid=d267) , done
			- [d671: 11716 – Digital Fortress](https://zerojudge.tw/ShowProblem?problemid=d671) , done
			- [c015: 10018 – Reverse and Add](https://zerojudge.tw/ShowProblem?problemid=c015) , done
		- Unit 11
			- [e529: 00482 – Permutation Arrays](https://zerojudge.tw/ShowProblem?problemid=e529)
			- [e155: 10935 – Throwing cards away I](https://zerojudge.tw/ShowProblem?problemid=e155)
			- [e564: 00540 – Team Queue](https://zerojudge.tw/ShowProblem?problemid=e564)
			- [b838: 104北二2.括號問題](http://b838:%C2%A0104%E5%8C%97%E4%BA%8C2.%E6%8B%AC%E8%99%9F%E5%95%8F%E9%A1%8C/)
			- [c123: 00514 – Rails](https://zerojudge.tw/ShowProblem?problemid=c123)
			- [f640: 函數運算式求值](https://zerojudge.tw/ShowProblem?problemid=f640)
			- [f607: 3. 切割費用](https://zerojudge.tw/ShowProblem?problemid=f607)
			- [d123: 11063 – B2-Sequence](https://zerojudge.tw/ShowProblem?problemid=d123)
			- [d442: 10591 – Happy Number](https://zerojudge.tw/ShowProblem?problemid=d442)
			- [a135: 12250 – Language Detection](https://zerojudge.tw/ShowProblem?problemid=a135)
			- [e641: 10260 – Soundex](https://zerojudge.tw/ShowProblem?problemid=e641)
			- [d267: 11577 – Letter Frequency](https://zerojudge.tw/ShowProblem?problemid=d267)
			- [d221: 10954 – Add All](https://zerojudge.tw/ShowProblem?problemid=d221)
			- [c875: 107北二2.裝置藝術](https://zerojudge.tw/ShowProblem?problemid=c875)
			- [b231: TOI2009 第三題：書](https://zerojudge.tw/ShowProblem?problemid=b231)
			- [d980: 11479 – Is this the easiest problem?](https://zerojudge.tw/ShowProblem?problemid=d980)
			- [e446: 排列生成](https://zerojudge.tw/ShowProblem?problemid=e446)
# some skill
- vim (done)
	- https://csdiy.wiki/%E5%BF%85%E5%AD%A6%E5%B7%A5%E5%85%B7/Vim/#vim_1
- git
	- https://csdiy.wiki/%E5%BF%85%E5%AD%A6%E5%B7%A5%E5%85%B7/Git/
	- https://www.lintcode.com/problem/?typeId=14
- gdb
	- https://jasonblog.github.io/note/gdb/index.html
- cmake
	- https://csdiy.wiki/%E5%BF%85%E5%AD%A6%E5%B7%A5%E5%85%B7/CMake/
- 提問的智慧 
	- https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/main/README-zh_CN.md
- shell  
	- https://www.shellscript.sh/
	- https://www.hackerrank.com/domains/shell
- 英國奧運開幕
- 活著
- 區公所問兵役
- 細想自指的bug
--- 
# 碩二下
## 架設server
- [服器架設篇目錄 - RockyLinux 9](https://linux.vbird.org/linux_server/rocky9/) 
# 并行与分布式系统
- https://csdiy.wiki/%E5%B9%B6%E8%A1%8C%E4%B8%8E%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F/CS149/
## 计算机系统基础
- https://csdiy.wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%B3%BB%E7%BB%9F%E5%9F%BA%E7%A1%80/CSAPP/
## 计算机体系结构
- https://csdiy.wiki/%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84/CS61C/
## Linux核心設計
- [Linux 核心設計 jserv](https://www.youtube.com/watch?v=7Rij1IZqazM&list=PL6S9AqLQkFpongEA75M15_BlQBC9rTdd8&index=1)




- 交大IOC5226[https://oscapstone.github.io/labs/overview.html]
- gun make
	- https://csdiy.wiki/%E5%BF%85%E5%AD%A6%E5%B7%A5%E5%85%B7/GNU_Make/

# 長期目標
- 東南大學 李逸 基礎分析
- 線性代數
- 矩陣論
- 隨機過程 張顥 清華大學
- 數字信號處理 張顥 清華大學
- 基礎代數
- baby rudin
- 機率論

