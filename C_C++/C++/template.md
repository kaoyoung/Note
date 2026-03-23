### 參考自[C++ 筆記】模板（Templates）（上） - part 31](https://hackmd.io/@LukeTseng/Sy_33EzBxe)和[【C++ 筆記】模板（Templates）（下） - part 32](https://hackmd.io/@LukeTseng/BJWj7znzxx/https%3A%2F%2Fhackmd.io%2F%40LukeTseng%2FS17ZraGSxe)
## 泛型程式設計
泛型在 C++ 中稱為模板，而模板的意思可以概括為「一種用來定義函數或類別的語法機制，可讓資料型態在使用時(**編譯期**)再指定，從而支援多種資料型態的操作」。
## C++模板定義
```C++
template <typename type> ret-type func-name(parameter list)
{
   // the body of function
}
```
- typename 是 C++ 關鍵字。（改成 class 也可以）
- type 是一個 placeholder，可為任意名稱，這個名稱代表資料型態，可以是 int，也可以是 string 等等。
- ret-type 為函數的回傳型態
範例 :
```c++
#include <bits/stdc++.h>
using namespace std;

template <typename T>
T add(T a, T b) {
    return a + b;
}

int main(){
    int a = 100, b = 50;
    float f1 = 10.0, f2 = 2.8;
    string s1 = "123", s2 = "456";
    
    cout << add(a, b) << endl;
    cout << add(f1, f2) << endl;
    cout << add(s1, s2) << endl;
    return 0;
}
```
Output :
```text
150 
12.8 
123456
```
### 類別模板
```c++
template <class type> class class-name { 
. 
. 
. 
}
```
- `<class type>` 的 class 也可寫 typename。
- class-name 為類別名稱。
範例 :
```c++
#include <bits/stdc++.h>
using namespace std;

template <class T> class Box{
public:
    T value;
    Box(T v){
        value = v;
    }
    void show(){
        cout << "Value : " << value << endl;
    }
};

int main(){
    Box <int> int_Box(100);
    Box <string> str_Box("LukeTseng");
    Box <float> flo_Box(87.87);
    
    int_Box.show();
    str_Box.show();
    flo_Box.show();
    
    return 0;
}
```
Output :
```text
Value : 100 
Value : LukeTseng 
Value : 87.87
```
---
## 建立模板實例（instance）
```c++
name_of_entity<type1, type2, ...>
```
- name_of_entity：模板名稱。
```c++
#include <bits/stdc++.h>
using namespace std;

template <typename T>
T add(T a, T b) {
    return a + b;
}

int main(){
    cout << add<int>(100, 50) << endl;
    cout << add<float>(10.0, 2.8) << endl;
    cout << add<string>("123", "456") << endl;
    
    return 0;
}
```
Output :
```text
150
12.8
123456
```
---
## 多個資料型態的模板
```cpp
#include <bits/stdc++.h>

using namespace std;

template <class T1, class T2, class T3> class Box{
public:
    T1 value1;
    T2 value2;
    T3 value3;
    
    
    Box(T1 v1, T2 v2, T3 v3) : value1(v1), value2(v2), value3(v3) {}
    
    void show(){
        cout << "Value1 : " << value1 << endl;
        cout << "Value2 : " << value2 << endl;
        cout << "Value3 : " << value3 << endl;
    }
};

int main(){
    Box <int, float, string> BBB(88, 87.87, "LukeTseng");
    BBB.show();
    return 0;
}
```
Output ：　
```text
Value1 : 88
Value2 : 87.87
Value3 : LukeTseng
```
---
## 變數模板 (C++ 14)
```c++
template <typename T> constexpr T pi = T(3.14159);
```
- `constexpr` : 代表這個變數是 **「編譯期常數」(Compile-time constant)**。這保證了數值在程式編譯時就決定好，執行效率極高，且不可被修改。
- `T pi` : 宣告一個名稱為 `pi` 的變數，其型態為 `T` (也就是上面定義的泛型)。
- `= T(3.14159)` : 這將數值 `3.14159` 強制轉型 (Construct) 為型態 `T`，並賦值給 `pi`。
#### Q : 為什麼不是寫(T)3.14159
> [!NOTE] 
>`T(3.14159)` 的語意是：「請用 3.14159 這個數值，**初始化 (Initialize)** 一個新的 `T` 型別物件。」，這是 C++ 對待型別的方式：即使是 `int` 或 `float`，也被視為某種物件。`(T)3.14159`是C-style cast，他的語意是：「不管 3.14159 是什麼，給我**暴力轉成** `T` 型別。」。在現代c++中初始化優先順序是 : 
>1. **大括號初始化** (Brace Initialization): `T{3.14}`
>2. **函數式/建構式初始化**: `T(3.14)` (最常用於模板)
>3. **C++ 轉型**: `static_cast<T>(3.14)`
>4. **C 風格轉型**: `(T)3.14` (最不推薦)

範例 :
```c++
#include <iostream>
#include <iomanip>

using namespace std;

// 變數模板, pi 圓周率常數
template<typename T> constexpr T PI = T(3.14159265358979323846);

// 函數模板, 計算圓面積
template<typename T> T area(T r) {
    return PI<T> * r * r;
}

int main() {
    cout << fixed << setprecision(15);
    cout << "PI<int>: " << PI<int> << ", area(2): " << area(2) << endl;
    cout << "PI<float>: " << PI<float> << ", area(2.0f): " << area(2.0f) << endl;
    cout << "PI<double>: " << PI<double> << ", area(2.0): " << area(2.0) << endl;
    return 0;
}
```

---
## 預設模板
```c++
#include <bits/stdc++.h>
using namespace std;

template <typename T1, typename T2 = double, typename T3 = string> class Box{
public:
    T1 value1;
    T2 value2;
    T3 value3;
    
    
    Box(T1 v1, T2 v2, T3 v3) : value1(v1), value2(v2), value3(v3) {}
    void show(){
        cout << "Value1 : " << value1 << endl;
        cout << "Value2 : " << value2 << endl;
        cout << "Value3 : " << value3 << endl;
    }
};

int main(){
    Box <int, float, string> B1(88, 87.87, "LukeTseng");
    B1.show();
    cout << "------------------------------" << endl;
    Box <int> B2(88, 87.87, "LukeTseng");
    B2.show();
    return 0;
}

```
Output : 
```text
Value1 : 88
Value2 : 87.87
Value3 : LukeTseng
------------------------------
Value1 : 88
Value2 : 87.87
Value3 : LukeTseng
```
---
## 模板特製化（Template Specialization）
範例 :
```c++
#include <iostream>
using namespace std;

template <typename T> 
void print(T value){
    cout << "一般型態" << endl;
}

template <> // 特製化語法
void print<int>(int value){
    cout << "這是 int 型態" << endl;
}

int main(){
    print('a');
    print(123);
    print(14.87);
    return 0;
}
```
Output :
```text
一般型態
這是 int 型態
一般型態
```
---
## 可變參數模板（Variadic Templates）
這是 C++ 11 引入的特性。
```c++
template<typename... Types>
void foo(Types... params);
```
- 模板參數包（template parameter pack）：用省略號`...`表示的可變長度模板參數列表。
- 函數參數包（function parameter pack）：與模板參數包對應的函數參數列表，也用`...`表示，如上範例的 `Types... params` 代表零個或多個函數參數。
範例 :
```c++
#include <bits/stdc++.h>

using namespace std;

// 基本的模板：只接收一個參數
template<typename T> T sum(T value) {
    return value;
}

// 可變參數模板：可接收多個參數
template <typename T, typename... Args> T sum (T f, Args... rest){
    return f + sum(rest...);
}

int main(){
    cout << "sum(1, 2, 3): " << sum(1, 2, 3) << endl;
    cout << "sum(1.5, 2.5, 3.0, 4.0): " << sum(1.5, 2.5, 3.0, 4.0) << endl;
    cout << "sum(10): " << sum(10) << endl;

    return 0;
}
```
Output :
```c++
sum(1, 2, 3): 6
sum(1.5, 2.5, 3.0, 4.0): 11
sum(10): 10
```
