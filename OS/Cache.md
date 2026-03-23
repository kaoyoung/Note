### Ref : [Difference between fflush and fsync](https://stackoverflow.com/questions/2340610/difference-between-fflush-and-fsync)、[cache design](https://cseweb.ucsd.edu/classes/fa14/cse240A-a/pdf/08/CSE240A-MBT-L15-Cache.ppt.pdf)、[细说Cache-L1/L2/L3/TLB](https://zhuanlan.zhihu.com/p/31875174)、[Cache Memory in Computer Organization](https://www.geeksforgeeks.org/computer-organization-architecture/cache-memory-in-computer-organization/)、[Cache replacement policy](https://www.geeksforgeeks.org/computer-organization-architecture/cache-replacement-policies/)
# 動機

![[cpu_memory gap.png]]
source: [细说Cache-L1/L2/L3/TLB](https://zhuanlan.zhihu.com/p/31875174)
從上面這張圖可以看到 cpu 和 memory 之間的差距很大，為了彌補這個差距，我們使用了 cache 來盡量讓兩者差距不要那麼大。
>[!question] 為啥 cpu 和 memory 差很多不好
>運算時我們需要有運算元和運算子，才能算東西。 cpu 決定運算的速度， memory 提供所需得運算元和運算子， 因此 cpu 和 memory 之間要彼此合作才能運算。 cpu 太快會處在算完但沒東西可算的情況；memory 太快會處在有東西可算但算不完的情況。

---
# Cache Block
![[physical_addr2cache.png]]
source : [Cache Design](https://cseweb.ucsd.edu/classes/fa14/cse240A-a/pdf/08/CSE240A-MBT-L15-Cache.ppt.pdf)
上圖是我們把一個 memory address 分成多欄以供我們在 cache 中查找，我們從右到左來看每一欄在幹嘛
- block offset : 在 cache 中我們是以 block 為單位儲存，為了對齊這個單位，我們用 address 最後幾個 bit 來表達這大小，其位元數量等於 $\lg(\text{block size})$ 。
- block address : 前面的 block offset 已經表達完在 cache 中單位的大小，接下來我們要表達其在哪個單位和驗證是否 hit 。
	- index : 對於 direct mapping 跟 set-associative mapping，我們將 cache 想成一個陣列，其陣列的索引值可以用 index 表達讓我們快速尋找。
	- tag : 檢查該資料是否安全，有了 tag 才能表達出所有 address 所有的位元。

>[!question] 為啥 block address 中要驗證 ?
>這問題等於是在問說 tag 存在的必要性。這問題可以從兩方面切入思考。第一方面，cache 的大小一定小於 memory 的大小，或是更精確地說是所有在電腦上運行程序所需的儲存空間，因此有可能的儲存空間到 cache 之間的映射為多對一，這時僅僅只依靠 index 做區分會產生歧異。另一個角度是，假設在一 64 位元的系統中，你想得到一個唯一的 memory address 需要 64 bit，如果少了 tag 你所能用的 bit 會小於 64 這會帶來歧異。

>[!question] 為啥 cache 要以 block 為單位不以 memory address size 為單位 ?
>可以從三個角度去做思考。第一個是 cache 本身的直覺，我們相信資料是不會隨機亂跳，而是有一定的規律，這規律體現在 spatial locality 和 temporary locality 上，所以如果我們以 block 為單位來設計 cache 可以更好的使用 space locality。第二是從硬體成本的角度來看，想一下我們在 cache 中是用 index 跟 tag 來檢索和確認一個 block ，而 block 才是我們存資料的地方，如果 block 越大，那 index + tag 的面積就可以縮小，所以每個資訊所需的附加儲存成本較小。第三個角度是從 DRAM 本身的特性來看， DRAM 的啟動延遲相較於連續傳輸慢，那為何不配合 spatial locality 一次多傳一點。

---
# Cache Mapping
Cache mapping 決定了 main memory 如何映射到 cache 中，主要有以下三個 mapping 的方式
- Direct Mapping
- Fully Associative Mapping
- Set-Associative Mapping
### Direct Mapping

![[direct_mapping.png]]
source : [Cache Design](https://cseweb.ucsd.edu/classes/fa14/cse240A-a/pdf/08/CSE240A-MBT-L15-Cache.ppt.pdf)
邏輯閘的介紹可以看 [[基礎邏輯閘介紹]] 。
將 memory address 的一個 memory block 映射到一個 cache line 中，如果該 cache line 已經有人直接覆蓋掉。一個顯然觀察是有多個 virtual address 映射到一個 physical address ，這觀察可以用兩角度來看，一個是 cache 大小一定小於 physical address 大小，另一個是 index 加上 cache line size 的位元數小於32。

>[!question] 為啥 cache 中 data 那欄要用 256 bit 而不要用 32 bit ?
>理由同前面  **「為啥 cache 要以 block 為單位不以 memory address size 為單位 ?」** 的回答。


>[!question] cache 的 valid bit、dirty bit 機制跟 [[segmentation & pagging]] 中 pagging 的權限控制 bit 有無異同 ?
>cache 的 valid bit 、 dirty bit 跟 pagging 的權限控制 bit  ，主要都是表達這筆資料的狀態，只是 pagging 的權限控制位元較多，可以管理更多狀態。cache 的 valid bit 表示這筆資料是否有效 (即目前這資料是不是可即接取用) ， dirty bit 表示這筆資料被當前 core 寫過在 write back 機制中，當該 cache line 被 evict 後要寫回 memory。 cache 的這些 bit 需要寫在 tag 、index 、 offset 之外是附加上去的，而 pagging 的權限控制位元是利用 PTE 中那些因為 page size 為 4KB 導致沒有用處的底部 12 位和實體位址並未用滿 64 bit 所遺留的高位元（如 NX bit），詳情可以參考 [[segmentation & pagging]] 中 pagging 權限那一部份。

### Fully Associative Mapping

![[fully_associative_mapping.png]]
source : [Cache Replacement Policies](https://ece752.ece.wisc.edu/lect11-cache-replacement.pdf)
每個 memory block 的值都可以直接放在任一一個 cache line 中，這解決 direct mapping 的 collision 問題，但它有一個致命缺點，要找一筆資料要比完 cache 中所有的 tag 因為 tag 可以亂放，這讓其製作成本提高。
### Set-Associative Mapping

![[set_associative_mapping.png]]
source : [Cache Design](https://cseweb.ucsd.edu/classes/fa14/cse240A-a/pdf/08/CSE240A-MBT-L15-Cache.ppt.pdf)
set-associative mapping 試著在 direct mapping 跟 fully associative mapping 之間找一個平衡點，讓找 tag 有一定規律 (保留 index 結構)，減緩 collision (讓同一個 index 可以放多筆 data)。

>[!question] 細說 set-associative cache 為了測試 set 有無命中所需增加的硬體複雜度 ?
>假設是 N-way set-associative  ，那我們需要 N 個 tag comparators 和 N 個 valid bit 的 AND 邏輯閘，最後的結果要匯入一個 N 對 1 的 multiplexer 做比較。

---
# 專有名詞
- cache line size (cache block size) : The amount of data that gets transferred on a cache miss.
- instruction cache : Cache that only holds instructions.
- data cache : Cache that only caches data.
- unified cache : Cache that holds both.
![[instruction_data_cache.png]]
source : [Cache Design](https://cseweb.ucsd.edu/classes/fa14/cse240A-a/pdf/08/CSE240A-MBT-L15-Cache.ppt.pdf)

- logical cache : cache stores data using virtual address.
- physical cache : cache stores data using physical address.
logical cache 跟 physical cache 關鍵點在於 cache 是出現在 MMU 之前還是之後
![[logical_addr_and_physical_addr.png]]
source: [细说Cache-L1/L2/L3/TLB](https://zhuanlan.zhihu.com/p/31875174)

---
# Cache Miss Rates
Cache Miss 的主要來源有三個，分別是
- Compulsory miss (Cold miss) : 我要先去 cache 找 block of memory ，但對於新的資料來說，不可能在 cache 中找到，因為你根本就沒見過。
- Capacity miss : Cache 的儲存容量有限，所以在 cache 容量滿且有新的資料要加進來的時候，必須要把一部分的 block of cache 清開，這時如果把以後會用到的資料從 cache 清走，那在以後必然造成 cache miss。
- Conflict miss : 對於不是 fully-associative 的 cache，在 cache 中同一個 set 可能有來自不同 block of memory 的映射，而在該 cache 中的 set 如果滿時有一個新的 block of memory 映射進來，必須要把一部分的 block of cache 清開，這時如果把以後會用到的資料從 cache 清走，那在以後必然造成 cache miss。
>可以把 conflict miss 想成 set 的 capacity miss

>[!question] 如何測量這三個 miss ?
>- Compulsory miss : number of misses in an infinite cache model.
>- Capacity miss : additional misses in a fully-associative cache
>- Conflict miss : additional misses in cache of interest

>[!question] 如何減少 miss rate ?
>- larger block size : 增加 spatial locality 。
>- higer associativity : 減小同個 index 時 collision 的影響。
>- software prefetching data
>- compiler optimization : reorder, loop interchange, loop fusion, blocking

---
# Replacement Policy
對於 replacement policy 有多個策略，例如 : FIFO、LRU、LRU、Pseudo-random 等。
Replacement policy 的基線是 optimal (belady's algorithm)，該算法是在說，把最長時間不會在用的 cache block 給替換出去。說一下 LRU 的實作，有一個精確的實作叫 counter-based true LRU 用 counter 來記錄每隔 way 離上次存取過了多久，但這太麻煩。現在的實作是用 Tree-based Pseudo-LRU (PLRU) ，它把每個 way 當成樹的葉節點，利用樹狀結構的分支點來記錄上一次走哪邊，每個 bit 代表一個十字路口，指標永遠指向上一次沒有被存取的那一邊，每次你存取某個 Way，一路上經過的指標就會全部翻轉 (指向另外一邊)。踢除時順著目前指標的箭頭一路走到底，找到的那個 Way 就是即將被踢除的替換目標。

>找到 index 再去找哪個 way 要被替換。

---
# Cache coherency protocol
寫操作才會改變值，所以我們看寫操作的策略
#### Cache hit 的操作
- write back : 數據只寫到 cache 中，代這 cache 被替換出去或是有其他核要存取該數據，才把該數據寫回 memory 中。
- write through : 數據同時寫到 cache 和 memory 中。
#### Cache miss 的操作
- write allocate : 去 memory 把包含該筆資料的 memory block 讀進 cache line 中，之後在 cache 中做修改。
- no-write allocate : 直接在 memory 中做修改。

>[!question] cache hit 和 cahce miss 的操作如何搭配
>- write back + write allocate : 一致的思想:「盡量在 cache 中做操作，減少 memory 的存取」，write allocate 有用到 spatial locality 的想法。
>- write through + no-write allocate : cache 和 memory 有很強的一致性，適合簡單的實作，或是寫入很少的情況。

前面講的 cache 跟 memory 都是 volatile memory ，斷電時如何處理。

>[!question] cache 和 memory 突然斷電的保命措施
>硬體層面 :
>- Uninterruptible Power Supply (UPS) : server 配大電池，在市電斷電時接手
>- BBWC / NVRAM :  在企業級的 RAID Contoller 的 cache 上加上一顆小電池。
>
>軟體層面:
>- `flush()`/`fsync()` : 強迫 cache 的資料寫回**硬碟**中 。

>`flush()`/`fsync()` 的說明 : 
>參考自 [Difference between fflush and fsync](https://stackoverflow.com/questions/2340610/difference-between-fflush-and-fsync) 。
>>`fflush()` works on `FILE*`, it just flushes the internal buffers in the `FILE*` of your application out to the OS.
>>`fsync` works on a lower level, it tells the OS to flush its buffers to the physical media.
>所以要想把當前應用的值真的寫回 buffer 需要先 `fflush()` 再 `fsync()`

對於不同 core 之間 cache 的一致性問題可以到 [[MESI and MOESI protocal]] 做理解。

---
# Cache 跟 page table 的比較

#### 儲存位置
- cache : 在 cpu 內
- page table : 在 memory 內
#### 功能
- cache : 讓 cpu 的運算可以更快找到資料
- page table : 讓 MMU 將 virtual address 轉成 physical address (在 RAM 中的位置) 。
#### 儲存東西
- cache : 資料本身和一些 status bit (valid bit, dirty bit)。
- page table : virtual address 和 physical address 轉換表，出來的值是寫在 **PTE** 的數值表示轉換的 physical address 還有 **Protection/Control Bits**（例如：Valid/Present bit 確認是否在 RAM 中、Read/Write 權限、Dirty bit 等）。
#### 注意細節
page table 找到的 physical address 會對應到一個 frame ，而你要的資料要靠 page offset 找出來，找到該 memory block 之後會寫到 cache line 中再到 register 給計算單元處理。

