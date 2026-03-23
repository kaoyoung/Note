### 參考自[# C++ nullptr 與 NULL 的差異](https://shengyu7697.github.io/cpp-nullptr/)、# [What is the nullptr keyword, and why is it better than NULL?](https://stackoverflow.com/questions/1282295/what-is-the-nullptr-keyword-and-why-is-it-better-than-null)
# 動機
直接看一函數重載的程式
```c++
#include <iostream>  
using namespace std;  
  
void func(char* p) {  
    std::cout << "pointer\n";  
}  
   
void func(int n) {  
    std::cout << "integer\n";  
}  
  
int main() {  
    func(0);  
    func(NULL); // compile error  
    return 0;  
}
```
他會輸出
```text
test2.cpp:13:13: error: call of overloaded ‘func(NULL)’ is ambiguous
   13 |         func(NULL);
      |         ~~~~^~~~~~
test2.cpp:2:6: note: candidate: ‘void func(char*)’
    2 | void func(char *p){
      |      ^~~~
test2.cpp:6:6: note: candidate: ‘void func(int)’
    6 | void func(int p){
```
破解這問題的關鍵在於compiler的頭文件
```c
#ifdef __cplusplus ---简称：cpp c++ 文件
#define NULL 0
#else
#define NULL ((void *)0)
#endif
```
- 在c++中NULL被定義為0
所以以下兩函式呼叫被視為相同
```C++
func(0);
func(NULL);
```
>[!Why we define NULL as 0 in C++]
>在c++中，不允許將`void *` ，隱式轉換為其他指標，必須要強制轉型 (Explicit Cast)。對於c這件事是允許的，所以他才將`NULL`定義為`((void *) 0)`。C++有這規定是為了避免在類型轉換時出問題，如果`int *`轉成`void *`，而這時如果允許`void *`轉型成`double *`會出事。

`void *`和其他指針的轉換關係

|**方向**|**程式碼範例**|**C 語言**|**C++**|**原因**|
|---|---|---|---|---|
|**具體 $\rightarrow$ Void**|`void *v = int_ptr;`|✅ 合法|**✅ 合法**|只是「忘記」型別，安全。|
|**Void $\rightarrow$ 具體**|`int *p = void_ptr;`|✅ 合法|**❌ 非法**|電腦不知道 `void*` 原本是什麼，C++ 為了安全禁止瞎猜。|
>[!IMPORTANT]
>為了解決上述函式 Overload的問題，所以在C++11中引入nullptr的觀念，將0和空指針的關係分開。

### 細看`nullptr`
在c++11
```c++
namespace std
{
typedef decltype(nullptr) nullptr_t; // (since C++11)
// OR (same thing, but using the C++ keyword `using` instead of the C and C++ 
// keyword `typedef`):
using nullptr_t = decltype(nullptr); // (since C++11)
} // namespace std
```
我們將`nullptr`的型別定義為`nullptr_t`。