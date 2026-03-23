# Q1 :為何`unordered_map<pair<int, int>, int> domino2number;`不行
因為`unordered_map` 是依照hash table來實作，而在c++的STL中的雜湊函數只有支援`int`, `string`, `char` 等基本型別，對於`pair` 這種複合型別，C++ 標準並沒有定義預設的雜湊方式，因為組合的可能性太多，標準庫選擇留給使用者自己定義。
### **解法一 : 用map**
```c++
#include <map> // 這樣就可以直接用了 
std::map<std::pair<int, int>, int> domino2number;
```
### **解法二 : 自訂義hash**
```c++
#include <iostream>
#include <unordered_map>
#include <utility> // for std::pair

// 1. 定義一個 Hash Functor
struct PairHash {
    size_t operator()(const std::pair<int, int>& p) const {
        // 使用 XOR 和位移來混合兩個 int 的 hash 值
        // 這是為了避免 {1, 2} 和 {2, 1} 產生碰撞的簡單寫法
        auto h1 = std::hash<int>{}(p.first);
        auto h2 = std::hash<int>{}(p.second);
        
        // 常見的 hash combine 技巧
        return h1 ^ (h2 << 1); 
    }
};

int main() {
    // 2. 在宣告 map 時，將 Hash Functor 作為第三個參數傳入
    std::unordered_map<std::pair<int, int>, int, PairHash> domino2number;

    domino2number[{1, 2}] = 100;
    std::cout << domino2number[{1, 2}] << std::endl; // 輸出 100

    return 0;
}
```
---
# Q2 : 以下程式有何問題 ? 
```c++=
#include <iostream>
#include <string>
#include <cmath>

int main(){
   int total;
   std::cin >> total;
   std::string str;
   while(total-- > 0){   
      getline (std::cin, str);
      
      double row_size = sqrt((int)str.size());
      if(row_size - (int)row_size != 0){
	      std::cout << "INVALID" << std::endl;
	      continue;
      }

      std::string answer = {};
      for(int i=0; i<row_size; ++i){
	      for(int j=0; j<row_size; ++j){
		      answer.push_back(str.at(i+j*row_size));
	      }
      }
	  std::cout << answer << std::endl;
   }
}
```
### 緩衝區殘留問題 :
當你使用 `std::cin >> total;` 讀取整數後，換行符號（`\n`）仍留在緩衝區內。隨後第一次執行 `getline(std::cin, str);`時，它會直接讀到那個殘留的換行符號，導致讀入一個空的字串。
- **解法** : 在 `std::cin >> total;` 後加入 `std::cin.ignore();` 來跳過該換行符。
### `cin >>` 和`getline()`比較

| **功能**            | **類型** | **對空白符號（空格/換行）的處理**            |
| ----------------- | ------ | ------------------------------ |
| **`std::cin >>`** | 格式化輸入  | **略過**領頭的空白，讀到非目標型別時**停止**。    |
| **`getline()`**   | 非格式化輸入 | **讀取**直到遇見換行符，並**消耗（移除）**該換行符。 |
### 說`cin >>`有buffer殘留的問題，為何`cin >> a >> b`是好的
- **`>>` 接 `>>`：** 沒問題，C++ 會幫你處理好所有的換行和空格。    
- **`getline` 接 `getline`：** 沒問題，因為 `getline` 會順手把換行符號從緩衝區抽走並丟掉。    
- **`>>` 接 `getline`：** **大問題！** 必須在中間加 `cin.ignore()` 來手動清除那個被留下的換行符號。
---
