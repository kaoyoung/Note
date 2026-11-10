# C 語言記憶體觀念考卷：解答

## 快速對答案（選擇題）

| 題號 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 答案 | A | D | D | A | D | B | A | C | B | C | D | B |

---

## 一、選擇題詳解

### 1. 答案：(A) .bss

`static` 區域變數雖然寫在函式裡，但生命週期是整個程式，所以不放在 stack。它沒有初始化，因此放在 .bss，並自動被設成 0。如果寫成 `static int s = 5;`，就會放在 .data。

### 2. 答案：(D) .rodata

`p` 這個指標本身在 stack，但它指向的字串常數 `"abc"` 放在唯讀的 .rodata。所以執行 `p[0] = 'x';` 是 undefined behavior，常見結果是 segmentation fault。

對照：`char a[] = "abc";` 是把字串**複製**到 stack 上的陣列，可以修改。

### 3. 答案：(D) 4 和 8

- `a` 是陣列，內容是 `'a' 'b' 'c' '\0'`，共 4 bytes。
- `p` 是指標，64-bit 系統上指標是 8 bytes，跟它指向什麼無關。

延伸：陣列傳進函式後會退化成指標，所以在函式裡對參數做 `sizeof` 只會得到 8。這是很常見的 bug。

### 4. 答案：(A)

兩者都在 heap 上配置，也都要 `free`，失敗時也都回傳 `NULL`。差別是 `calloc(n, size)` 會把內容清成 0，`malloc` 的內容則是殘留的垃圾值。

### 5. 答案：(D)

`realloc` 失敗時回傳 `NULL`，但原本那塊記憶體還在。直接寫回 `p`，等於把唯一指向它的指標蓋成 `NULL`，那塊記憶體就再也 `free` 不到了（memory leak）。正確寫法見第 15 題。

另外，`realloc` 不一定會搬家；需要搬家時，也會把舊資料複製到新位址。

### 6. 答案：(B)

`x` 是區域變數，放在 stack，`f` 回傳後那塊空間就失效了。呼叫端拿到的是 dangling pointer，有時讀起來好像還是 10，但下一次函式呼叫就可能把它蓋掉。

修正方式：改用 `malloc`、改成 `static`，或讓呼叫端傳入 buffer。

### 7. 答案：(A)

`free` 之後 `p` 仍然指向舊位址：再拿來用就是 use-after-free，再 `free` 一次就是 double free。設成 `NULL` 後，誤用通常會直接 crash，比較容易抓到；而且 C 標準規定 `free(NULL)` 什麼都不做，重複 `free` 也不會出事。

### 8. 答案：(C) 12

```
| c | pad pad pad | i i i i | d | pad pad pad |
  1      3           4       1       3        = 12
```

`i` 必須從 4 的倍數位址開始，所以 `c` 後面補 3 bytes。整個 struct 的大小也要是 4 的倍數（這樣放在陣列裡，每個元素的 `i` 才都能對齊），所以 `d` 後面再補 3 bytes。

### 9. 答案：(B) 8

```
| i i i i | c | d | pad pad |
     4      1   1     2      = 8
```

成員一樣，只改順序就省了 4 bytes。一般原則是把大的型別放前面，在記憶體吃緊的嵌入式系統裡很有用。

### 10. 答案：(C)

- **碎片化**：長時間運作的機台一直 `malloc`／`free`，heap 會變得零碎，總空間夠，卻配不到連續的一塊。
- **時間不固定**：配置要花多久不確定，不適合有即時性要求的控制。
- **失敗難處理**：機台跑到一半配置失敗，通常沒有好的補救方法。

實務上會改用靜態配置或 memory pool；MISRA C 等規範也限制動態配置。

### 11. 答案：(D)

硬體暫存器、ISR 會修改的變數，值可能在程式不知道的情況下改變。沒有 `volatile` 的話，編譯器可能把讀取最佳化掉，只讀一次就一直用舊值。

**常見追問陷阱**：`volatile` 不保證 atomic，也不能取代 mutex。

### 12. 答案：(B)

函式裡的區域陣列放在 stack，8 KB 已經超過 4 KB 的 stack。

- (A) `static` 陣列放在 .bss，不佔 stack（RAM 不夠的話，通常在連結階段就會報錯）。
- (C) 空間在 heap，stack 上只有一個指標。
- (D) 只有 3 bytes。

---

## 二、程式與簡答題詳解

### 13. 記憶體分區填空

| 標號 | 存放區 | 說明 |
|---|---|---|
| (1) `g` | .bss | 未初始化的全域變數 |
| (2) `h` | .data | 有初始化的全域變數 |
| (3) `cnt` | .bss | 未初始化的 static 區域變數 |
| (4) `init` | .data | 有初始化的 static 區域變數 |
| (5) `x` | stack | 一般區域變數 |
| (6) `buf` | stack | 區域陣列 |
| (7) `s` 本身 | stack | 指標本身是區域變數 |
| (8) `"hello"` | .rodata | 字串常數 |
| (9) `q` 指向的空間 | heap | `malloc` 配置的 |
| (10) `f` 的機器碼 | .text | 程式碼區 |

### 14. 找錯

**問題 1：少配置了 `'\0'` 的空間。** `strlen` 不含結尾的 `'\0'`，但 `strcpy` 會把它一起複製，所以會寫出界 1 byte（off-by-one 的 buffer overflow）。

**問題 2：沒有檢查 `malloc` 是否回傳 `NULL`。** 配置失敗時，`strcpy` 會寫入 `NULL`，程式直接 crash。

**改正後：**

```c
char *dup(const char *s) {
    size_t len = strlen(s) + 1;   /* 多留 '\0' 的空間 */
    char *r = malloc(len);
    if (r == NULL)
        return NULL;              /* 配置失敗就回報給呼叫端 */
    memcpy(r, s, len);            /* 長度已知，用 memcpy 即可 */
    return r;                     /* 呼叫端用完要記得 free */
}
```

**評分重點**：兩個問題各 4 分，改正版本 2 分。有提到「呼叫端要負責 `free`」可酌量加分。

### 15. 安全地使用 realloc

```c
int *tmp = realloc(arr, 2 * n * sizeof *arr);
if (tmp == NULL) {
    /* 配置失敗：arr 仍然有效，原本的資料都還在 */
    /* 依情況繼續使用 arr，或 free(arr) 後回報錯誤 */
    return -1;
}
arr = tmp;
n = 2 * n;
```

**評分重點**：

- 用暫存變數接 `realloc` 的回傳值：5 分
- 確認不是 `NULL` 之後才指定回 `arr`：3 分
- 記得更新 `n`：2 分

**加分**：如果 `n` 很大，`2 * n * sizeof *arr` 可能溢位，嚴謹的寫法會先檢查 `n <= SIZE_MAX / (2 * sizeof *arr)`。

### 16. 簡答：stack 和 heap 的差別

| 比較項目 | stack | heap |
|---|---|---|
| 誰負責管理 | 編譯器自動處理，進出函式時配置與釋放 | 程式設計師自己 `malloc`／`free` |
| 生命週期 | 離開作用域（函式結束）就消失 | 從 `malloc` 到 `free` 為止，可以跨函式使用 |
| 大小限制 | 小；Linux 預設通常 8 MB，MCU 可能只有幾 KB | 大，受限於可用記憶體 |
| 配置速度 | 很快，只是移動 stack pointer | 較慢，配置器要找合適的空間 |
| 碎片化 | 不會 | 會 |
| 常見錯誤 | stack overflow（遞迴太深、區域陣列太大）、回傳區域變數位址 | memory leak、double free、use-after-free |

**評分重點**：答對任三點即滿分，每點約 3～4 分；說明清楚、有舉例者給滿分。
