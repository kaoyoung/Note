# C 語言記憶體觀念考卷

| 項目 | 內容 |
|---|---|
| 作答時間 | 建議 40 分鐘 |
| 滿分 | 100 分 |
| 環境假設 | 64-bit Linux、gcc；`int` 為 4 bytes，指標為 8 bytes（題目另有說明時以題目為準） |

> **作答方式**：選擇題在「答：」後面填入選項代號；程式題和簡答題直接寫在「答：」下方。

---

## 一、選擇題（每題 5 分，共 60 分）

### 1. 下面這段程式中，變數 `s` 存放在哪一區？

```c
void f(void) {
    static int s;
}
```

- (A) .bss
- (B) stack
- (C) heap
- (D) .rodata

**答：**

---

### 2. 下面這行程式中，字串 `"abc"` 本身存放在哪裡？

```c
char *p = "abc";
```

- (A) stack，跟 `p` 放在一起
- (B) heap
- (C) .bss
- (D) .rodata（唯讀資料區）

**答：**

---

### 3. `sizeof(a)` 和 `sizeof(p)` 分別是多少？

```c
char a[] = "abc";
char *p = "abc";
```

- (A) 3 和 3
- (B) 4 和 4
- (C) 3 和 8
- (D) 4 和 8

**答：**

---

### 4. `malloc` 和 `calloc` 最主要的差別是什麼？

- (A) `calloc` 會把配置到的記憶體清成 0，`malloc` 不會
- (B) `calloc` 配置在 stack，`malloc` 配置在 heap
- (C) `calloc` 要來的記憶體不需要 `free`
- (D) `malloc` 保證不會失敗，`calloc` 可能失敗

**答：**

---

### 5. 這行程式有什麼潛在問題？

```c
p = realloc(p, 100);
```

- (A) 沒有問題，這是標準寫法
- (B) `realloc` 一定會搬到新位址，所以舊資料會遺失
- (C) `realloc` 會把內容清成 0
- (D) `realloc` 失敗時回傳 `NULL`，原本的指標被蓋掉，那塊記憶體就漏掉了

**答：**

---

### 6. 這個函式的問題是什麼？

```c
int *f(void) {
    int x = 10;
    return &x;
}
```

- (A) 沒有問題
- (B) 回傳了 stack 上已經失效的位址（dangling pointer）
- (C) memory leak
- (D) double free

**答：**

---

### 7. `free(p);` 之後馬上寫 `p = NULL;`，主要的好處是什麼？

- (A) 避免之後誤用成 dangling pointer，而且就算不小心再 `free` 一次也安全
- (B) 讓 `free` 執行得更快
- (C) 這樣記憶體才會真的還給作業系統
- (D) 編譯器規定一定要這樣寫

**答：**

---

### 8. 在 `int` 為 4 bytes、按 4 bytes 對齊的平台上，`sizeof(struct A)` 是多少？

```c
struct A { char c; int i; char d; };
```

- (A) 6
- (B) 8
- (C) 12
- (D) 16

**答：**

---

### 9. 同樣的平台，把成員順序改成下面這樣，`sizeof(struct B)` 是多少？

```c
struct B { int i; char c; char d; };
```

- (A) 6
- (B) 8
- (C) 12
- (D) 16

**答：**

---

### 10. 嵌入式系統常常避免在執行中使用 `malloc`，最主要的原因是什麼？

- (A) 嵌入式的編譯器都不支援 `malloc`
- (B) heap 一定比 stack 小
- (C) 記憶體會碎片化、配置時間不固定，而且執行到一半才失敗很難處理
- (D) `malloc` 配置的記憶體無法被 CPU 讀取

**答：**

---

### 11. `volatile` 關鍵字的作用是什麼？

- (A) 讓變數存放在 heap
- (B) 讓變數變成唯讀
- (C) 讓變數的存取變成 thread-safe（atomic）
- (D) 告訴編譯器這個值可能被程式以外的東西改變，每次都要真的去記憶體讀寫，不能最佳化掉

**答：**

---

### 12. 在 stack 只有 4 KB 的 MCU 上，下列哪個寫法最可能造成 stack overflow？

- (A) 在檔案最外層宣告 `static char buf[8192];`
- (B) 在函式裡宣告 `char buf[8192];`
- (C) 在函式裡寫 `char *buf = malloc(8192);`
- (D) 在函式裡宣告 `const char buf[] = "hi";`

**答：**

---

## 二、程式與簡答題（每題 10 分，共 40 分）

### 13. 記憶體分區填空（每格 1 分）

請寫出每個標號所指的東西存放在哪一區。可用的答案：stack、heap、.data、.bss、.rodata、.text。

```c
#include <stdlib.h>

int g;                                   // (1) g
int h = 5;                               // (2) h

void f(void) {
    static int cnt;                      // (3) cnt
    static int init = 1;                 // (4) init
    int x;                               // (5) x
    char buf[16];                        // (6) buf
    char *s = "hello";                   // (7) s 這個指標本身
                                         // (8) "hello" 這個字串
    int *q = malloc(sizeof(int) * 4);    // (9) q 指向的那塊空間
    free(q);
}
                                         // (10) 函式 f 的機器碼
```

| 標號 | 存放區 |
|---|---|
| (1) | |
| (2) | |
| (3) | |
| (4) | |
| (5) | |
| (6) | |
| (7) | |
| (8) | |
| (9) | |
| (10) | |

---

### 14. 找錯

這個函式想複製一個字串，裡面至少有兩個問題。請指出問題，並寫出改正後的版本。

```c
char *dup(const char *s) {
    char *r = malloc(strlen(s));
    strcpy(r, s);
    return r;
}
```

**答：**

問題 1：

問題 2：

改正後：

```c

```

---

### 15. 安全地使用 realloc

`arr` 是用 `malloc` 配置、目前有 `n` 個元素的 `int` 陣列。請寫一段程式，把它擴充成 `2n` 個元素，並且做到：**配置失敗時，原本的資料不能遺失**。

**答：**

```c

```

---

### 16. 簡答

用自己的話說明 stack 和 heap 的差別，至少講三點（例如：誰負責管理、生命週期、大小限制、配置速度、常見錯誤）。

**答：**

1.

2.

3.



# 另外
## Q1. 啥是 Heap ，其如何在給定的大小為 N 的陣列中找第 K 大的值?

