- header : `#include <sstream>`
## 核心操作 
- `<<` (插入/寫入) : 把資料（數字、字串等）「流」進 stringstream 裡。
- `>>` (提取/讀取) : 從 stringstream 裡把資料「流」出來，存到變數裡。
---
## 常見用途
### 資料型態轉換 : (數字 $\leftrightarrow$ 字串)
- 現已被`to_string()`、`stoi()`、`stod()`等取代
```c++=
#include <iostream>
#include <sstream> 
#include <string>
using namespace std;

int main() {
	// number to string
    int number = 12345;
    stringstream ss;
    
    ss << number; 
    
    string str_num;
    ss >> str_num; 

    cout << "字串內容: " << str_num << endl; 
    
    // string to number
	string data = "99";
	stringstream ss;
	ss << data; // 把字串丟進去

	int val;
	ss >> val; // 用 int 接住它，自動轉換

	cout << val + 1 << endl; // 輸出 100 (證明是數字運算)
    return 0;
}
```
### 切割字串 (split string)
- `stringstream` : 它會**自動忽略空白（空格、Tab、換行）**，這使得解析一句話裡的單字變得非常容易。
```c++
#include <iostream>
#include <sstream>
#include <string>
using namespace std;

int main() {
    string sentence = "Hello world using stringstream";
    stringstream ss(sentence); // 初始化時直接放入字串
    
    string segment;
    
    // 當 ss 還有東西可以讀取時，迴圈繼續
    while (ss >> segment) {
        cout << "讀取到: " << segment << endl;
    }
    
    return 0;
}
```
### 重複使用 
同一個 `stringstream` 物件中處理多筆不同的資料，**必須記得清除狀態**，否則會出錯。
重複步驟 : 
1. `ss.str("")`
2. `ss.clear()`
#### Q : 為何要這兩操作
stringstream 內有兩樣東西在運作 : 
1. buffer : 存放字串的地方
2. stage flag : 紀錄目前的健康狀況(是否讀完、是否出錯)
與之對應的清空操作 : 
- `ss.str("")` $\leftrightarrow$ 清空buffer
- `ss.clear()` $\leftrightarrow$ 清除狀態。沒清除的話第二次寫入會出錯，應為寫入第一次後狀態為EOF，你必須先清掉才能繼續寫入。
```c++
#include <iostream>
#include <sstream>
#include <string>
using namespace std;

int main() {
	stringstream ss;

	// 第一回合
	ss << "100";
	int a;
	ss >> a;

	// 準備第二回合
	ss.str(""); // 清空內容
	ss.clear(); // 清除狀態旗標 (重要！)

	ss << "200";
	int b;
	ss >> b;
	
	return 0;
}
```
---
## `stringstream`跟`getline()`的結合
### 動機一
原本`stringstream`要碰到空白才會停，那如果我要指定分隔號如何操作? 
例如 : "apple,banana,orange" 想從 , 來猜分
### `getline()`語法拆解
$$getline(\underbrace{ss}_{\text{來源:串流}}, \underbrace{target}_{\text{目的地:字串}}, \underbrace{delimiter}_{\text{切割刀:符號}})$$
- **來源 (ss)**：裝著你原本長字串的 `stringstream`。
- **目的地 (target)**：切下來的那一塊要存到哪個 `string` 變數裡。    
- **切割刀 (delimiter)**：遇到這個符號就「切斷」並吐出前面的部分（通常是 `','`）。
```c++
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

using namespace std;

int main() {
    string data = "John,25,Engineer";
    stringstream ss(data); // 1. 把字串塞入串流
    
    string segment; //用來暫存切下來的每一小塊
    vector<string> result;

    // 2. 重點在這裡！
    // 意思：從 ss 讀取資料寫入 segment，直到遇到 ',' 才停下
    while (getline(ss, segment, ',')) {
        result.push_back(segment); // 把切下來的東西存起來
        cout << "切下一塊: " << segment << endl;
    }

    return 0;
}
```
### 動機二 : 
處理不同長度的輸入。
```c++=
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

int main() {
    string s1;
    getline(cin, s1); // 從鍵盤讀取整行，例如 "10 20 30"
    
    // 正確寫法：宣告一個新的變數(通常叫 ss)，並用圓括號把字串塞進去
    stringstream ss(s1); 
    
    // 測試一下
    string temp;
    while (ss >> temp) {
        cout << temp << endl;
    }
    
    return 0;
}
```
