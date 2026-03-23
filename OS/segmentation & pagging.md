### Ref: [Segmentation in Operating System](https://www.geeksforgeeks.org/operating-systems/segmentation-in-operating-system/)、[Paging](https://www.geeksforgeeks.org/operating-systems/paging-in-operating-system/)、[Multilevel Paging in Operating System](https://www.geeksforgeeks.org/operating-systems/multilevel-paging-in-operating-system/)、[Demand Paging in Operating System](https://www.geeksforgeeks.org/operating-systems/what-is-demand-paging-in-operating-system/)

# Segmentation

![[segementation_demonstration.png]]
source: [Segmentation in Operating System](https://www.geeksforgeeks.org/operating-systems/segmentation-in-operating-system/)
觀察上面這個圖可以看到 segmentation 主要在維護的資料結構是 segment table，segment table 主要由兩個元素主成
- base address : 決定了從哪裡開始 d 個 byte 的位移
- limit : limit 決定了從 base address 位移的極限，這確保了資料的安全性，一個程序的資料可以不被其他程序讀到。
這樣的記憶體位置安排可以去除 internal fragmentation 因為要多大就配多大的記憶體，由 limit 處理限制，但無可避免的會有 external fragmentation，雖然記憶體可以按照順序一個個接著排，但有一個程序執行完，其記憶體會被回收，該 hole 在補的時候難免會出現補不齊的情況。 segmentation 在找位置時需要一次大小於判斷和一次加法的判斷，相比 paging 慢。

---
# Paging

![[paging_demonstration.png]]
source: [Paging](https://www.geeksforgeeks.org/operating-systems/paging-in-operating-system/)

paging 的主要思考方式是**想把 logical address 切成幾段，這樣我只要處裡好每一段從 logical address 到 physical address 的映射就好**，如此可以避開 segmentation 要做加法的麻煩。須注意 **page table 是在 RAM** 中處理，所以只要給位置就好。

>page table 跟 segmentation 有一顯然的區別， segmentation 的分割是沒標準化的，也就是起始點是不確定的，那我們對於每個位置的存取只能老老實實地做加法；另一方面 page table 切割的每一段都是 $2^k$ 大小，所以可以直接用位元拼貼的方式求最後位置。

Page 的運作流程如上如所示，可以看到它改進了 segmentation 要去做加法運算的痛點，但看起來失去 segmentation 中 limit 限制對於存取權的掌控。可以看到現在為**加快 paging 轉換到 physical address 的速度引入了 TLB 這個快取的元件**，對於快取的知識可以到 [[Cache]] 觀看。名詞解釋一下
- cpu 給的 logical address 分成兩部份， page number (p) 跟 page offset (d)。
- physical address 分成兩部份， frame number (f) 跟 frame offset (d)。
- frame offset 等於 page offset 。

>[!question] page table 如何做權限控制
>在 32 位元的系統中 page table entry (PTE) 大小通常是 32 bit ，在 64 位元的系統中 page table entry (PTE) 大小通常是 64 bit 。以 page 通常的大小 4kb 做說明，因為 page 大小是 4kb ，所以最後面的 12 bit 沒有用處而部份的權限控制是靠這 12 bit
>- 位 0 (present) : 指示 page 是否在 RAM 中。
>- 位 1 (read/write) : 控制 page 讀寫權限。
>- 位 2 (user/kernel) : 區分 user mode 跟 supervisor mode 。
>- 位 4 (page-level cache disable) : 控制 page 緩存策略。 1 表示禁用 page 的緩存； 0 啟用緩存。
>- 位 5 (accessed) : 紀錄 page 是否被訪問過。
>- 位 6 (dirty) : 紀律 page 是否被修改過。
>
>64 位元很大我們不可能讓 physical address 全部使用，最多就 50 出頭，所以位 50 幾到 63 可以用來做權限控制
>- 位 63 (No-Execute) : 控制 page 執行權限。 1 禁止在該頁面執行程式； 0 允許執行。
>
>從上面的說明可以看到我們是**針對一整個 page 作權限控制而不是針對一個字組 (Word)**。 Word 是 CPU 執行單元的存取單位 (通常 4 或 8 Bytes)。

>[!question] page table 是否會過大 ?
>想一下在 64 位元的作業系統中，我們令 page 大小為 4 KB ，那整個 page table 多大，至少為 $$\text{PTE size} \times \text{number of PTE} = 64 \times \frac{2^{64}}{2^{12}} = 2^{58} \text{ bits} = 2^{55} \text{ bytes}$$
>這計算其實有小瑕疵，現在的系統雖然是 64-bt 系統，但其實虛擬地址不會用完全部的 64 bits ，大概就使用 50 位元附近。從前述說明可以知道用單層分頁表來說不合理，通常會用多層分頁表 (multilevel paging) ，其核心思想是：「我們不可能一次用完所有 virtual address ，那我們可不可以讓 page table 隨著使用的 virtual address 越多而變得越大」。多層分頁表的操作是把 page number 切成好幾段，那每一段所需要的 table 大小會指數級下降，而我們從 virtual address 拼接出整個 physical address，只需要每個分段的 page table 有對應關係就好，因此全部的 page table 大小可以隨著使用的 virtual address 越多而變得越大，流程圖如下
>![[Multilevel_paging.png]]
>source: [Multilevel Paging in Operating System](https://www.geeksforgeeks.org/operating-systems/multilevel-paging-in-operating-system/)


>[!question] 如何減小 RAM 中 page table 的大小
>使用 demand paging 的技術，只在該 page 需要時才把該頁面從 disk 載到 memory 中。下面流程圖小品一下
>![[demand_paging.png]]
>source: [Demand Paging in Operating System](https://www.geeksforgeeks.org/operating-systems/what-is-demand-paging-in-operating-system/)
>這邊需要注意一下以下三點
>- Page Replacement : RAM 的大小是很有限的，所有程序所要的 page 大小有可能大於記憶體大小，所以我們需要置換策略，常見的有 FIFO、LRU、LEU、MRU、Random。
>- Page cleanup : 當 process 結束時需要把該 page 清掉。作業系統會根據這個 process 分配到的實體記憶體位置，在該 process 結束時把它用到的記憶體位置分配回 free frame list ，而 page table 占用的記憶體空間會被釋放掉。
>- valid-invalid bit : 這在 page table 中用來做 process 執行期間的狀態確認和保護
>	- valid : 表示目前 page 合法。表示 page 已放在 RAM 中可直接供 CPU 使用。
>	- Invalid : 有兩個可能。一個是該 page 已放到 disk 裡 (不在 RAM 中)，另一個是該 process 根本沒分配到該 page ，你越界存取了。當 CPU 讀到 Invalid，就會觸發 Page Fault 。

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

