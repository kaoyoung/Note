## Q1 : relocatable object file跟shared object file差在哪 ?
## A1 :
### 核心差異比較表

|**特性**|**Relocatable Object File (.o)**|**Shared Object File (.so)**|
|---|---|---|
|**主要功能**|作為靜態連結的輸入，用來產生執行檔或 .so。|提供動態連結，可在程式執行時被載入。|
|**生成工具**|由編譯器/組譯器（如 `gcc -c`）生成。|由連結器（Linker，如 `ld` 或 `gcc -shared`）生成。|
|**地址定位**|使用相對節區（Section）的偏移量。|包含程式頭（Program Header），定義虛擬地址。|
|**共享性**|連結後代碼被併入執行檔，無法跨進程共享。|同一個檔案在記憶體中可被多個進程共享。|
|**內部結構**|通常只有 Section Header Table（區段表）。|同時擁有 Section Header 與 Program Header（段表）。|
|**ELF 標記**|`ET_REL`|`ET_DYN`|
#### 1. Relocatable Object File (可重定位檔案)
這是編譯過程中產生的中間產物。當你執行 `gcc -c main.c` 時，產生的 `main.o` 就是這種類型。
- **特性**：它包含了二進位代碼和數據，但裡面的函數調用（如 `printf`）或全域變數的地址尚未確定，標記為「待重定位」。    
- **用途**：連結器（Linker）會把多個 `.o` 檔案「黏」在一起，解析彼此的符號（Symbols），最後產出一個完整的執行檔。   
#### 2. Shared Object File (共享目標檔案)
這就是我們常說的「動態連結庫」（Shared Library）。當你執行 `gcc -shared -fPIC -o libtest.so test.c` 時，產生的就是這類檔案。
- **特性**：它已經過初步連結，==內含「位置無關代碼」（PIC, Position-Independent Code）==。這意味著它可以被載入到記憶體的任何位置而不需要修改代碼本身。    
- **動態連結**：與靜態連結不同，`.so` 的代碼不會被複製到最終的執行檔中。執行檔只會紀錄「我需要這個庫」，直到程式**運行時**（Runtime），動態連結器（ld-linux.so）才會把 `.so` 載入記憶體。  
### 結構上的關鍵差異
如果你使用 `readelf -h` 命令觀察這兩種檔案，你會發現：
1. **Program Header Table**：    
    - **Relocatable (.o)**：通常**沒有** Program Header，因為它不需要直接被作業系統載入（Loader）執行。        
    - **Shared Object (.so)**：**必須有** Program Header，因為載入器需要知道如何將該檔案的區段（Segments）映射到記憶體空間。        
2. **符號解析**：    
    - 在 `.o` 中，符號引用的地址通常是 `0x0`，並附帶一個重定位表（Relocation Table）告訴連結器以後要修正。        
    - 在 `.so` 中，雖然也有重定位資訊，但它主要透過 **GOT（Global Offset Table）** 和 **PLT（Procedure Linkage Table）** 機制來實現在運行時的間接跳轉。
---
## Q2 : `printf()`為何是relocatable object file他不是定義在<stdio.h>中嗎，那應該是preprocessor處理阿
## A2 :
### 1. 釐清誤區：宣告 (Declaration) vs. 定義 (Definition)
要理解這個問題，必須先區分這兩個概念：
- 宣告 (Declaration) —— 在 <stdio.h> 中：    
    這就像是告訴編譯器：「這世界上有一個函數叫 printf，它長這樣：接收一個字串，回傳一個整數。」這只是規格說明，讓編譯器在編譯你的程式時，知道你沒寫錯參數。    
- 定義 (Definition) —— 在 libc 庫中：
    這是 printf 真正的原始碼（執行邏輯）。這份程式碼早在你安裝作業系統或編譯器時，就已經被編譯成二進位的「目標檔案」存放在系統裡了（通常在 /lib/libc.so.6 或 /usr/lib/libc.a）。
### 2. 編譯的四個階段：為何 Preprocessor 搞不定？
Preprocessor（預處理器）的工作非常簡單：**純粹的文字取代**。
1. **Preprocessor (預處理)**：看到 `#include <stdio.h>`，它就把該檔案的內容「複製、貼上」到你的程式碼頂端。這時你的程式碼裡依然==只有 `printf` 的規格==，沒有它的機器碼。    
2. **Compiler (編譯)**：將 C 語言轉成組合語言。它看到你呼叫 `printf`，會檢查規格是否符合標頭檔的宣告，符合就過關。    
3. **Assembler (組譯)**：將組合語言轉成 **Relocatable Object File (.o)**。
    - **關鍵點**：這時你的 `.o` 檔案裡有一行指令說「跳轉到 `printf` 的位址」，但 `.o` 檔案**根本不知道 `printf` 的位址在哪**。        
    - 因此，它會留下一個「填空題」**（Relocation Entry），標記為：「這裡有個 `printf` 符號，請連結器以後幫我填上真正的位址」。
4. **Linker (連結)**：這才是 `printf` 真正現身的時候。連結器會把你的 `.o` 檔案跟系統提供的 `libc` 結合。  
### 3. 為什麼它是 Relocatable（可重定位）？
當你編譯自己的 `main.c` 產生 `main.o` 時：
- 你的 `main.o` 就是一個 **Relocatable Object File**。    
- 它包含了一個「符號表」（Symbol Table），裡面記錄著 `printf` 是個 **Undefined Symbol（未定義符號）**。
- 「可重定位」的意思是：這個檔案裡的程式碼位址都是從 `0` 開始算的「相對位址」。等到連結器（Linker）把它們跟其他的庫（Library）拼裝在一起時，才會根據最終的位置，「重定位」這些符號的真正記憶體位址。    
### 總結對比

|**階段**|**處理對象**|**printf 的狀態**|
|---|---|---|
|**Preprocessor**|`#include <stdio.h>`|僅僅是把 `printf` 的**宣告**（名片）貼進來。|
|**Compiler**|語法檢查|確認你呼叫 `printf` 的方式跟名片上寫的一樣。|
|**Assembler**|產生 `.o` 檔|產生機器碼，但 `printf` 的位址是個**空格**。|
|**Linker**|結合庫檔案|從 `libc` 中找到 `printf` 的定義，把**空格填上正確位址**。|

---
## Q3 : shared library是靠mmap實現嗎 ?
## A3 :
你執行一個程式時，作業系統的核心（Kernel）與動態連結器（Dynamic Linker，通常是 `ld-linux.so`）會合作，利用 `mmap` 將 `.so` 檔案的內容映射到進程的虛擬位址空間中。

以下是它是如何透過 `mmap` 運作的詳細過程：
### 1. 動態連結器的載入流程
當程式啟動時，動態連結器會檢查執行檔需要的 `.so` 列表，然後執行以下步驟：
1. **開啟檔案**：使用 `open()` 開啟 `.so` 檔案。    
2. **讀取 ELF Header**：確認檔案結構，找出哪些部分是程式碼（Segment）、哪些是資料。    
3. **執行 `mmap`**：    
    - **程式碼段（.text）**：連結器會呼叫 `mmap`，並設定權限為 `PROT_READ | PROT_EXEC`。這部分在實體記憶體中**只有一份**，所有使用該函式庫的進程都會指向相同的實體頁面（Physical Pages），從而節省記憶體。        
    - **資料段（.data / .bss）**：連結器也會呼叫 `mmap`，但權限設為 `PROT_READ | PROT_WRITE`，且通常會加上 **`MAP_PRIVATE`** 標誌。       
### 2. 為什麼用 `mmap`？（三大優點）

#### A. 記憶體共享（Space Efficiency）
這是 Shared Library 名字的由來。由於程式碼段是==唯讀==的，核心可以讓 100 個不同的進程同時 `mmap` 同一個 `libc.so` 的程式碼段，但在實體記憶體（RAM）中只需要佔用一份空間。
#### B. 寫入時複製（Copy-on-Write, CoW）
對於「資料段」（全域變數），雖然大家共享初始資料，但每個進程必須有獨立的副本。
- 當使用 `mmap` 並設定 `MAP_PRIVATE` 時，核心會利用 **CoW** 機制：只要進程沒改動變數，大家就共用記憶體；一旦某個進程想修改變數，核心才會為該進程複製一份私有的實體頁面。
#### C. 延遲載入（Demand Paging）
`mmap` 並不會立刻把整個 10MB 的函式庫全部塞進 RAM。它只是先建立「虛擬位址」到「檔案內容」的映射關係。
- 只有當程式真正執行到某個函數，觸發了 **Page Fault（缺頁中斷）** 時，核心才會把那一頁的內容從硬碟讀進記憶體。這==大大提升了程式啟動的速度==。    
### 3. `mmap` 之後發生了什麼？（重定位 Relocation）
雖然 `mmap` 把檔案搬進了記憶體，但我們上一題提到的「空格」（Undefined Symbols）還沒填好。因為 `.so` 檔案每次被 `mmap` 進去的虛擬地址可能都不一樣（受 **ASLR** 隨機化影響），所以動態連結器在 `mmap` 完成後，還需要根據映射後的實際基底位址，去修正 **GOT（Global Offset Table）**，確保程式能正確跳轉到 `printf` 等函數的位址。