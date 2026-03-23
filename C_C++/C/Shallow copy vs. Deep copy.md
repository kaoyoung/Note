## Shallow copy
淺拷貝是指 **複製一個對象時，只複製其表面層次的內容**。如果這個對象內部包含指標，那麼只會複製指標的值（即記憶體地址），而不會複製指標所指向的實際資料。
>keep in mind : 一個資料可以直接表達(int、char，bool、struct等)，或是透過地址去表達(char*, struct str* 等)，而淺拷貝是只去抄了地址，內容沒抄阿，所以沒產生拷貝。問題會發生在有個人淺拷貝一個物件，兩人誰動，地址內資料會一起動，這就違背拷貝的初衷，相當於COW(copy on write)，在子或父有一方改資料(write)時，你沒copy太荒謬。
### 例一 : 結構體（Struct）之間的直接賦值
```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

typedef struct {
    int id;
    char *data; // 結構體內含一個指標
} Record;

void shallow_copy_example() {
    // 1. 原始結構體
    Record original;
    original.id = 100;
    // 為 data 分配獨立的記憶體空間
    original.data = malloc(10 * sizeof(char)); 
    strcpy(original.data, "Hello");

    // 2. 淺拷貝發生在這裡！
    Record copied = original; 

    // 兩個結構體有不同的 id 值，但 data 指標指向同一個地址
    printf("Original ID: %d, Data Address: %p\n", original.id, (void*)original.data);
    printf("Copied ID:   %d, Data Address: %p\n", copied.id, (void*)copied.data);

    // ... 後續導致問題
}
```
### 例二 : 傳遞結構體作為函式引數（Call by Value）
```c
void print_record(Record r) {
    // r 是傳入結構體的副本，也會包含指標的淺拷貝
    printf("Copied data: %s\n", r.data);
}
```
### 可能問題 :
- data corruption
- double free
## Deep copy 
```c
// 函式用於執行深拷貝
Record deep_copy(const Record *src) {
    Record dest;
    
    // 1. 複製非指標成員 (淺拷貝部分)
    dest.id = src->id;
    
    // 2. 複製指標成員 (核心深拷貝)
    size_t len = strlen(src->data) + 1; // 獲取長度 (+1 for \0)
    dest.data = malloc(len * sizeof(char)); // [關鍵步驟] 分配新的記憶體
    if (dest.data == NULL) {
        // 處理記憶體分配失敗
        perror("malloc failed");
        exit(EXIT_FAILURE);
    }
    strcpy(dest.data, src->data); // [關鍵步驟] 複製資料內容
    
    return dest;
}
```