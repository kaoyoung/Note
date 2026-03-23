### 參考自[C++ 筆記】例外處理（Exception Handling） - part 28](https://hackmd.io/@LukeTseng/H1JdIKyBgl)
## 基本例外處理
```cpp
try { 
	// 這可能會拋出一個例外 
	// Code that might throw an exception 
	
	throw val
} 
catch (ExceptionType e) { 
	// 例外處理的地方 
	// exception handling code 
}
```
- `ExceptionType e`：看 catch 要捕捉什麼，如 `int e` 就是捕捉 int 型態的例外。
### 範例
```cpp
#include <iostream>
#include <string>

using namespace std;

int main(){
    // 整數型態的例外捕捉
    try {
        int val = 10;
        if (val > 5){
            throw val;
        }
        cout << "No Exception." << endl;
    }
    catch (int e){
        cout << "Caught an integer exception with value : " << e << endl;
    }
    
    // 字串型態的例外捕捉
    try{
        string error_msg = "Something wrong!";
        throw error_msg;
    }
    catch (string e){
        cout << "Caught a string exception : " << e << endl;
    }
    return 0;
}
```
Output :
```text
Caught an integer exception with value : 10 
Caught a string exception : Something wrong!
```
--- 
## 標準例外
標準例外是 C++ STL 裡面定義的一組例外類別，大多被定義在 `<stdexcept>` 或 `<typeinfo>`（如 `bad_cast`） 中。
#### **主要分成兩大類 :**
- 邏輯錯誤（logic_error）
- 執行期錯誤（runtime_error）
![[C++ exception hierarchy.png]]

| 例外類型 | 說明 |
|---------|------|
| `std::logic_error` | 邏輯錯誤。表示程式中存在不合理的邏輯問題，通常是可預先檢查出的錯誤。 |
| `std::invalid_argument` | 無效的參數。通常因傳遞錯誤或非法的參數值而引發，屬於 `logic_error` 的子類。 |
| `std::domain_error` | 數學定義域錯誤。當參數不在函數允許的數學定義域內時拋出，例如平方根的負數輸入。屬於 `logic_error` 的子類。 |
| `std::length_error` | 長度錯誤。當容器長度超過其最大容量限制時拋出，屬於 `logic_error` 的子類。 |
| `std::out_of_range` | 超出範圍。當使用無效索引存取容器元素時拋出，屬於 `logic_error` 的子類。 |
| `std::runtime_error` | 執行期間錯誤。表示在執行階段發生的非預期狀況，無法在編譯期間預測。 |
| `std::range_error` | 範圍錯誤。當數值運算結果超出有效範圍但仍合法（如浮點精度損失）時拋出，屬於 `runtime_error` 的子類。 |
| `std::overflow_error` | 溢位錯誤。當數值計算超過資料型態的上限時拋出，屬於 `runtime_error` 的子類。 |
| `std::underflow_error` | 下溢錯誤。當浮點數計算結果太接近零而無法正確表示時拋出，屬於 `runtime_error` 的子類。 |
| `std::bad_alloc` | 記憶體配置失敗。當 `new` 無法配置足夠記憶體時拋出，屬於 `std::exception` 的子類。 |
| `std::bad_function_call` | 錯誤的函數呼叫。當呼叫尚未設定目標的 `std::function` 物件時拋出。 |
| `std::bad_cast` | 錯誤的型態轉換。當使用 `dynamic_cast` 進行不合法的轉型時拋出。 |
範例 : 
```cpp
#include <iostream>
#include <stdexcept>
#include <cmath>

using namespace std;

double safe_sqrt(double x){
    if (x < 0){
        throw domain_error("輸入錯誤！平方根的定義域為正數");
    }
    return sqrt(x);
}

int main(){
    try{
        double val = -5.0;
        cout << "計算 " << val << " 的平方根..." << endl;
        double result = safe_sqrt(val);
        cout << "結果是: " << result << endl;
    }
    catch (domain_error e){
        cout << "捕捉到 domain_error 例外: " << e.what() << endl;
    }
    return 0;
}

```
Output :
```cpp
計算 -5 的平方根...
捕捉到 domain_error 例外: 輸入錯誤！平方根的定義域為正數
```
---
## catch 多個例外
```cpp
try {         
    // Code that might throw an exception
} 
catch (type1 e) {   
    // executed when exception is of type1
}
catch (type2 e) {   
    // executed when exception is of type2
}
catch (...) {
    // executed when no matching catch is found
}
```
範例 : 
```cpp
#include <iostream>
#include <stdexcept>

using namespace std;

int divide(int a, int b){
    if (b == 0){
        throw invalid_argument("除數不能為0");
    }
    if (a == 0){
        throw 0;
    }
    return a / b;
}

int main(){
    int x, y;
    cout << "請輸入兩個整數 (a, b) :";
    cin >> x >> y;
    
    try{
        int result = divide(x, y);
        cout << "除法運算結果 : " << result << endl;
    }
    catch (invalid_argument e){
        cout << "捕捉到 invalid_argument 例外: " << e.what() << endl;
    }
    catch (int e){
        cout << "捕捉到整數型態例外，值為: " << e << endl;
    }
    
    return 0;
}
```
#### catch 所有例外
`catch(...)` 即可捕捉所有的例外，延續上個範例，增加這個上去：
```cpp
#include <iostream>
#include <stdexcept>

using namespace std;

int divide(int a, int b){
    if (a == 9999){ // 新增處
        throw "abc";
    }
    if (b == 0){
        throw invalid_argument("除數不能為0");
    }
    if (a == 0){
        throw 0;
    }
    return a / b;
}

int main(){
    int x, y;
    cout << "請輸入兩個整數 (a, b) :";
    cin >> x >> y;
    
    try{
        int result = divide(x, y);
        cout << "除法運算結果 : " << result << endl;
    }
    catch (invalid_argument e){
        cout << "捕捉到 invalid_argument 例外: " << e.what() << endl;
    }
    catch (int e){
        cout << "捕捉到整數型態例外，值為: " << e << endl;
    }
    catch (...){ // 新增處
        cout << "捕捉到未知的例外";
    }
    
    return 0;
}
```
- 當輸入 a = 9999, b = 任意數時，就會輸出「捕捉到未知的例外」。
---
## catch by reference
只需要傳遞對拋出例外的參考，不需要建立他的 copy，能減少這部分的效能開銷。除了這之外，主要拿來捕捉多型的例外。
```cpp
#include <iostream>
#include <stdexcept>

using namespace std;

int main() {
    try {
        throw runtime_error("發生錯誤：無效的操作");
    }
    catch (const runtime_error& e) {
        cout << "捕捉到例外: " << e.what() << endl;
    }

    return 0;
}
```
output : 
```text
捕捉到例外: 發生錯誤：無效的操作
```
#### 捕捉多型的例外
```cpp
#include <iostream>
#include <exception>
struct Base : std::exception {
    virtual const char* what() const noexcept override { return "Base"; }
    virtual void info() const { std::cout << "Base::info\n"; }
};
struct Derived : Base {
    const char* what() const noexcept override { return "Derived"; }
    void info() const override { std::cout << "Derived::info\n"; }
};

int main() {
    try {
        throw Derived{};
    } catch (Base b) {           // 以值捕捉 -> 切片發生
        std::cout << b.what();  // 輸出 "Base"
        b.info();               // 呼叫 Base::info（不是 Derived::info）
    }

    try {
        throw Derived{};
    } catch (const Base& b) {   // 以參照捕捉 -> 保留多型
        std::cout << b.what();  // 輸出 "Derived"
        b.info();               // 呼叫 Derived::info
    }
}
```
Q : 為何不寫`catch (Derived b)` ?
A : 你「可以」寫 `catch (Derived b)`，而且在「只有拋 `Derived`」這個前提下它是正確的；  
但它不具備「多型捕捉」的能力，因此不是好的設計，也不是通用寫法。
