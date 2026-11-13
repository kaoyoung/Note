![[Pasted image 20261111213207.png]]
參考: [Embedded Linux system development training](https://bootlin.com/doc/training/embedded-linux/embedded-linux-slides.pdf)

>[!question]
>圖中的 host 跟 target 如何理解?

![[Pasted image 20261111231850.png]]
參考: [Embedded Linux system development training](https://bootlin.com/doc/training/embedded-linux/embedded-linux-slides.pdf)

- Host: 開發的機器，在上面寫程式、編譯產生映像檔 (Image file: 將作業系統、核心、開機引導程序 (bootloader) 等所有必要程式碼跟資料，打包成一組可直接寫入裝置儲存媒體的二進制檔案)。 嵌入式映像檔常見組成: bootloader 、kernel 、device tree (描述硬體配置與周邊設備資訊的檔案，`.dtb` 檔) 、root filesystem (包含系統運作所需的函式庫、設定檔、指令工具與使用者應用程式)。
- Target: 最後執行這些程式的嵌入式系統。

Target 受限於記憶體大小、儲存空間和 CPU 性能等硬體因素，不能或是不便在 Target 上編譯，或是不想在 target 上裝開發工具 (編譯器 (compiler)、組議器 (assembler)、連結器 (linker)、C 函式庫 (glibc/ musl)、gdb 等)，這可能是怕麻煩或為了安全。不在 target 上做邊譯還有一個雞生蛋的問題，最一開始 target 沒有 kernel 跟 root filesystem (一整個目錄樹的內容: `/bin`、`/lib`、`/etc` 等，掛載在 `/`) 根本沒有環境來跑編譯器。

>[!note] OS vs. Kernel vs. File system
>OS 提供操作環境給使用者和應用程式，包括: 核心、系統函式庫、系統工具與 shell等；kernel 是 OS 最底層、權限最高的部分，可以直接跟硬體互動，應用程式必須透過系統呼叫讓 kernel 來幫自己跟硬體互動，kernel 負責
>- process management
>- memory management
>- device driver
>- file system
>- network
>
>File system 代表資料在儲存裝置上的組織規則，它決定檔案怎樣存、資料夾如何分層、權限如何設；檔案放在儲存裝置的哪個區塊，例如: ext4、FAT、UBIFS 等，由 kernel 裡的檔案系統驅動實做。


>[!question]
>Development PC (Host) 中的 Tool 如何理解?

這邊的 tool 是指 toolchain 代表軟體開發的工具集，包括
- compiler (編譯器): 將原始碼轉成 assembly language
- assembler (組譯器): 將 assembly language 轉成目的檔 (object file, `.o`) ，包括 machine code、symbol table 和 relocation
- Library (函式庫): 提供程式許多有用的功能，同時提供程式和 kernel 互動的函式。例如: glibc、musl c。
- linker (連結器): 將 machine code 跟 library 的函數連結起來。精確來說是，把多個目的檔和函式庫合併，其中要解析符號（找出每個函式和變數定義在哪裡），再做重定位（決定最後的位址並修正引用），最後產出 ELF 執行檔。連結方式包括
	- 靜態連結: 其檔案常為 `.a` file，會將有用的 `.o` 打包進最後的執行檔。
	- 動態連結: 執行檔會記錄需要那些 `.so` file，以及需要那些符號名稱。程式執行時，由動態連結器把 `.so` 載入記憶體並解析地址。
- debugger (除錯器): 用於測試、追蹤與排查程式執行時的錯誤，例如 gdb。
- kernel header:  C library 需要跟 kernel 互動，所以需要 kernel headers 來編譯，而使用者程式有時要直接使用 kernel 定義的介面，因為 C library 沒有涵蓋所有 kernel 介面（例如驅動程式專用的 `ioctl` 命令碼和結構），這時就要直接引入 kernel headers。可參考以下圖片。

![[Pasted image 20261111231316.png]]
參考: [Embedded Linux system development training](https://bootlin.com/doc/training/embedded-linux/embedded-linux-slides.pdf)

注意：toolchain 裡 kernel headers 的版本，必須小於或等於 target 上執行的 kernel 版本。kernel 對 user space 的介面是向後相容的。

>[!question]
>Native toolchain 跟 cross-compile toolchain 差在哪?

Native toolchain 是在使用那一套 toolchain 翻譯成執行檔後可不可以直接在編譯的那台機器上運行，可以的話便是 native toolchain。
Cross-compiling toolchain 在 host 上執行，但產生的執行檔是給另一個平台（target）跑的。使用 cross-compiling toolchain 可能是因為架構不同、C Library 不同、kernel headers 版本不同或是避免誤連到 host 自己的函式庫。

>[!question]
>cross compile 要注意那些細節? 請以建立 toolchain 過程進行探討

建立 toolchain 時要注重三個點: ABI 、floating point support 跟 CPU optimization flags
ABI (application binary interface): 編成 machine code 後的約定，包括
- 呼叫函式時，參數放在哪些 register ，register 不夠後放在 stack 上。
- 回傳值放在哪個 register。
- 系統呼叫如何發出 ( syscall 編號放在哪，用哪個指令進入 kernel)
- 結構體、各型別多大
- endianness
- padding
程式、函式庫跟核心會共用一個 ABI 以支持相互呼叫。
Floating point support: 有些架構沒有 FPU (Floating Point Unit)，所以我們需要在編譯的時候決定要用 hard float code 還是 solft float code
- hard float code: 編譯器直接產生 FPU 的浮點指令。在有 FPU 的 CPU 上，這是最快、也最正常的選擇。如果沒有 FPU 則由 kernel 模擬。
- soft float code: 由 user space library 來做運算 (可以由加、減跟位元運算來模擬)。
CPU optimization flag: 在編譯時可以為特定的 target architecture 做編譯，如此可以依照該架構特性進行優化，例如:
- `-march`: 選定架構家族**內**的版本和擴充，決定可以用哪些指令
- `-mtune`: 只針對某顆 CPU 調整指令排程，不改變可用的指令

>[!question]
>在沒有 FPU 的情況下，Hard float code 跟 soft float code 哪個較慢?

在沒有 FPU 的 CPU 上，兩者最後都要用整數運算來模擬浮點，差別在於模擬發生在哪裡：hard float code 是在執行時，每條浮點指令觸發例外，再由 kernel 模擬；soft float code 是在編譯時，就把浮點運算換成呼叫函式庫libgcc）的函式。。Hard float code 的步驟如下:
1. CPU 不認得這條指令，觸發例外。
2. 保存目前得 PC 跟狀態，跳到核心的例外處理入口
3. 保存 user space 使用者程式的暫存器到記憶體
4. 找出是哪條指令觸發例外
5. 找出這條指令要幹嘛
6. 做計算
7. 寫回結果
8. 恢復  user space 使用者程式的狀態，在切回使用者模式，並把 PC 一到下一條指令。

soft float code 不會觸發例外，也不需要解碼（編譯時就決定好要呼叫哪個函式），所以 1 到 5 和 8 都不用做，只剩 6、7。此外，它也避開了例外帶來的額外成本：
- 例外會導致 pipeline flush
- 切進 kernel 要執行另一段程式碼，user space 的程式失去 locality 的優勢，kernel 的程式碼和資料會擠掉 user space 的 cache line 跟 TLB 項目。


>[!question]
>Embedded system (target) 中的 Bootloader 如何理解?

bootloader (啟動程式)，開機後作業系統載入前執行的一段程式碼，用來
- 初始化硬體: 初始化基本硬體設備。設定 CPU 的初始狀態 (設定時脈、CPU 模式等)，初始化 RAM (要先設定記憶體控制器)，並初始化開機需要的周邊（如 UART、Flash）。
- 建立執行環境
	- 設定 stack pointer
	- 清除 `.bss` (block started by symbol，存放未初始化的 global variables 和 static variables，該段的記憶體內容會被清為 `0` 或 `NULL`)
- 載入作業系統:  把 kernel image 、device tree（`.dtb`），以及選用的 initramfs，從儲存裝置或網路讀進 RAM。

最後傳遞開機參數，然後把控制權交給 OS kernel。