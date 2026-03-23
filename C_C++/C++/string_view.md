### 參考自[還在用 const std::string &? 試試 std::string_view 吧!](https://tigercosmos.xyz/post/2023/06/c++/stringview/)
## 簡介
`std::string_view` 的 API 設計哲學是：**「盡可能像 `std::string`，但剔除所有會修改原始資料的動作」**。這讓你可以很無痛地把原本吃 `const std::string&` 的函式改成 `std::string_view`。
`std::string_view` 的結構非常簡單，它只包含兩個東西：
1. **指標 (Pointer):** 指向字串資料的開頭。    
2. **長度 (Size):** 記錄字串的長度。   
因為它只有這兩個小小的變數，所以複製 `string_view` 的成本極低（就像複製兩個整數一樣快），而且**完全不需要分配記憶體 (Zero Allocation)**。
---
## 如何避免`const std::string&`不必要記憶體配置
傳統寫法:
```c++
// 參數接受 const string&
void processString(const std::string& s) {
    // 做一些操作...
}

int main() {
    // 當你傳入字串字面量 (String Literal)
    processString("Hello World"); 
    
    // 發生了什麼事？
    // 1. 編譯器看見 "Hello World" 是 const char*
    // 2. 函式需要 std::string
    // 3. 系統默認執行 `new` 分配記憶體，把 "Hello World" 複製進去
    // 4. 函式結束後，再執行 `delete` 釋放記憶體
    // => 慢！
}
```
使用 `string_view` (零成本)
```c++
#include <string_view>

// 參數改為 string_view (注意：通常直接傳值 copy，不用傳 reference)
void processString(std::string_view sv) {
    // 做一些操作...
}

int main() {
    processString("Hello World");
    
    // 發生了什麼事？
    // 1. 建立一個 string_view，指向 "Hello World" 的記憶體位置，長度設為 11。
    // 2. 沒有記憶體分配 (No malloc/new)。
    // 3. 沒有資料複製 (No memcpy)。
    // => 快！
}
```
---
## `std::string_view`函式
1. `remove_prefix(n)`、`remove_suffix(n)`
```c++
std::string_view sv = "\"Hello\""; // 長度 7

sv.remove_prefix(1); // 去掉開頭的 "
sv.remove_suffix(1); // 去掉結尾的 "

// 現在 sv 是 "Hello"，長度 5
// 原始字串完全沒變，只是 sv 的觀察範圍變了
```
2. 和 `std::string` 一模一樣的「讀取與搜尋」函數
**查詢與存取**
- **`size()`** / **`length()`**: 回傳長度。    
- **`empty()`**: 是否為空。
- **`operator[]`**: 像陣列一樣存取，例如 `sv[2]` (不檢查邊界)。
- **`at()`**: 安全存取，越界會拋出例外 (Exception)。
- **`front()`** / **`back()`**: 取得第一個或最後一個字元。
- **`data()`**: 取得原始 `const char*` 指標 (⚠️注意：不保證有 `\0` 結尾，後面會解釋)。
**子字串**
- **`substr(pos, count)`**: 切割字串。
    - _差異點：_ `string::substr` 會複製產生新字串；`string_view::substr` 只是產生一個新的 view (O(1) 速度)。
**搜尋 (Search)**
- **`find()`**: 找字串或字元。
- **`rfind()`**: 從後面找。
- **`find_first_of()`** / **`find_last_of()`**: 找集合中的任意字元。
- **`find_first_not_of()`**: 找不在集合中的字元。
2. 它「沒有」哪些函數？
- **沒有 `c_str()`**:
    - `std::string` 有 `c_str()` 保證回傳以 null (`\0`) 結尾的字串。
    - `string_view` 可能只是字串的中間一段，**不保證結尾有 `\0`**。
    - 如果你需要傳給舊的 C API (例如 `printf("%s")`)，**不能直接用 `string_view`**，必須先轉回 `string` 加上結尾。
- **沒有修改函數**:
    - 沒有 `push_back()`、`append()`、`insert()`、`operator+=`。
    - 它是唯讀的 (Read-only)。
- **沒有容量概念**:
    - 沒有 `capacity()` 或 `reserve()`，因為它不管理記憶體。ㄎ
---
## 如何用 `string_view` 寫一個超快的 Split 函數
```c++
#include <iostream>
#include <string_view>
#include <vector>

// 將字串依據 delimiter 切割，完全不複製記憶體
std::vector<std::string_view> split(std::string_view str, char delimiter) {
    std::vector<std::string_view> result;
    
    while (true) {
        // 1. 找到分隔符號的位置
        size_t pos = str.find(delimiter);
        
        if (pos == std::string_view::npos) {
            // 剩下的就是最後一段
            result.push_back(str);
            break;
        }
        
        // 2. 切出一段 (O(1) 操作)
        result.push_back(tr.substr(0, pos));
        
        // 3. 移除已處理的部分 (O(1) 操作)
        str.remove_prefix(pos + 1);
    }
    
    return result;
}

int main() {
    std::string s = "apple,banana,orange,grape";
    
    // 這裡沒有任何字串複製發生，只有產生一堆輕量的 view
    auto parts = split(s, ','); 
    
    for (auto v : parts) {
        std::cout << v << "\n";
    }
}
```
