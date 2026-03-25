### ref : [深入理解缓存一致性协议MESI和MOESI](https://zhuanlan.zhihu.com/p/721128435)、[MESI 协议学习笔记](https://lotabout.me/2022/MESI-Protocol-Introduction/)、[Cache Coherency & I/O ordering](https://hackmd.io/@qwe661234/r1BDYhVHo)、[MESI and MOESI protocols](https://developer.arm.com/documentation/den0013/0400/Multi-core-processors/Cache-coherency/MESI-and-MOESI-protocols)
# 動機
對於多核 CPU的結構來看，每個 core 都有自己私有的 cache ，而這時如果我們採用 write back 或是 write allocate 的技術，那不同 core 對於同一個物理地址的資料可能不同步，那我們如何確保 core 之間的訊息是一致的呢。

>注意 : cache 最小單位是 cache line ，所以 cache 一致性協議的顆粒度是 cache line 。

---
# MESI
#### 動機
對一個 cache line 有影響的操作有以下幾種可能
- local read : local CPU do read.
- local write : local CPU do write.
- remote read : remote CPU do read.
- remote write : remote CPU do write.

從這些狀態出發，我們可以很自然地推出 cache line 有以下幾種狀態
- local read : 沒啥影響。
- local write : cache line 的資料被改變，跟 memory 中的不一致。
- remote read : 別人也也這份資料，所以 cache line 的狀態有「自己獨享」跟「和別人共享」兩狀態。
- remote write : 自己的該資料變無效，因為別的 core 的資料是最新的。

總結一下，我們將 cache line 的狀態定為四種
- Modified (M) : The data in this cache line is modified.
- Exclusive (E) : The data in this cache line is exclusive.
- Shared (S) : The data in this cache line is shared.
- Invalid (I) : The data in this cache line is invalid.

依據這些狀態說明有兩個顯然觀察
- modifed 跟 exclusive 的資料是只有該 core 有，其他人沒有。
- shared 的資料有多個 core 有。

#### cache line 狀態的狀態機
cache line 的狀態有可能因為自己本地操作而修改，或是別的 core 的操作而修改 (透過 shared bus 或是 interconnect 溝通)。
自己本地 core 操作的 cache line 狀態機
![[private_cache_line_FSM.png]]
source : [OpenMP, Cache Coherence](https://inst.eecs.berkeley.edu/~cs61c/su20/pdfs/lectures/lec23.pdf)

>說明 : 
>- Invalid 會依靠 read/write miss 變成 valid 的情況 (shared、exclusive、modified) 
>- read hit 沒用。
>- 只要 write 該 cache line 就會變成 modified。

其他 core 操作對自己 cache line 的狀態機
![[other_cache_line_FSM.png]]
source : [OpenMP, Cache Coherence](https://inst.eecs.berkeley.edu/~cs61c/su20/pdfs/lectures/lec23.pdf)

>說明 : 
>- **這邊的 read/write hit 都是 probe 出來的，也就是要別的 core 有對一樣的 cache line 有操作，經 shared bus 跟 interconnect 並藉由 bus snooping 讓自己得知才算。**
>- 別的 core 有 write 一樣的 cache line 操作 (probe write hit)，自己的 cache line 都會變成 invalid，因為別人的資料才是最新的
>- 別的 core 有 read 一樣的 cache line 操作 (probe read hit)，自己的 cache line 如果是 modifed 要寫回記憶體，別人才能讀到最新的值。

>[!question] 為何要分 shared 跟 exclusive ，他們的資料都是 clean 且跟 memory 同步
>一個 cache line 是 exclusive 表示這資料只有自己有，所以 write 時可以自己偷偷改。另一方面 cache line 是 shared 表示這資料別人也有，所以 write 時可以要到匯流排廣播說我要改了，把該 shared 的 cache line 變 invalid ，當然自己的 cache line 狀態是 modified。

---
# MOESI

>[!motivation] 動機
>針對以下情況做優化，在 MESI 中，如果本地的 cache line 是 modified，當別的 core 要讀時，需要把數據寫回記憶體中，別的 core 再去 memory 中讀，這太慢了，我們可不可以直接用匯流排傳該數據。

相比 MESI 多定義一個狀態
- Owned (O) : A cache line that is modified and in possibley more than one cache.
	- **Only one** core can hold the data in the **owned state**. **The other** cores can hold the same data in **shared state**.
Owned 狀態允許這 cache linie 在 cache 之間傳遞並保持 shared 的狀態。而不需要立刻寫回 memory，等到該 cache line 被 evict 才寫回 memory。
