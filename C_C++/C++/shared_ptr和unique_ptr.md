### 參考自[C++ std::shared_ptr 用法與範例](https://shengyu7697.github.io/std-shared_ptr/)和[C++ std::unique_ptr 用法與範例](https://shengyu7697.github.io/std-unique_ptr/)
#### **需要引入的標頭檔**：`<memory>`，編譯需要支援 C++11
## `shared_ptr` :
### 動機
`std::shared_ptr` 是可以讓多個 `std::shared_ptr` 共享一份記憶體，並且在最後一個 `std::shared_ptr` 生命週期結束時時自動釋放記憶體。
範例 :
```c++
void UseRawPointer() {  
    // 使用原始指標  
    Song* pSong = new Song("Just The Way You Are", "Bruno Mars");  
  
    // Use pSong...  
    pSong->DoSomething();  
  
    // 別忘了要 delete!  
    delete pSong;  
}  
  
void UseSmartPointer() {  
    // 使用智慧型指標  
    shared_ptr<Song> song2(new Song("Just The Way You Are", "Bruno Mars"));  
  
    // Use song2...  
    song2->DoSomething();  
  
} // song2 在這裡自動地被 deleted
```
>[!NOTE] 
>`shared_ptr`讓程序員不再擔心忘記釋放記憶體的問題，記憶體釋放由 control block 根據 reference count 接管，當最後一個 `shared_ptr` 消失時自動 delete。`shared_ptr` 的 control block 是 C++ 標準庫在 user space 用 heap 管理的資料結構，OS 只負責提供記憶體，不參與引用計數或物件生命週期決策。
### 初始化 :
```c++
// 盡可能地使用 make_shared  
auto sp1 = make_shared<Song>("The Beatles", "Hey Jude");  
// 或者不使用 auto 像這樣寫  
shared_ptr<Song> sp1 = make_shared<Song>("The Beatles", "Hey Jude");  
  
// 下面這樣的寫法也可以，但會有一些副作用，記憶體配置上不連續，以及分開記憶體配置效能相對低弱  
// Note: Using new expression as constructor argument  
// creates no named variable for other code to access.  
shared_ptr<Song> sp2(new Song("Lady Gaga", "Poker Face"));  
  
// When initialization must be separate from declaration, e.g. class members,  
// initialize with nullptr to make your programming intent explicit.  
shared_ptr<Song> sp5(nullptr);  
//等於: shared_ptr<Song> sp5;  
sp5 = make_shared<Song>("Avril Lavigne", "What The Hell");
```
### 存取 std::shared_ptr 的原始指標
使用`std::shared_ptr.get()`
```c++
#include <iostream>  
#include <string>  
#include <memory>  
  
class LargeObject {  
public:  
    void DoSomething() {  
        printf("DoSomething\n");  
    }  
};  
  
void LegacyLargeObjectFunction(LargeObject *lo) {  
    printf("LegacyLargeObjectFunction\n");  
    lo->DoSomething();  
}  
  
int main() {  
    printf("===1===\n");  
    // Create the object and pass it to a smart pointer  
    std::shared_ptr<LargeObject> pLarge(new LargeObject());  
  
    printf("===2===\n");  
    // Call a method on the object  
    pLarge->DoSomething();  
  
    printf("===3===\n");  
    // 傳遞原始指標給 legacy API  
    LegacyLargeObjectFunction(pLarge.get());  
  
    printf("===4===\n");  
    // 用原始指標去接  
    LargeObject *p = pLarge.get();  
    LegacyLargeObjectFunction(p);  
  
    printf("===5===\n");  
    return 0;  
}
```

### `unique_ptr` 轉成 `shared_ptr`
1. `shared_ptr<Point> p = make_unique<Point>();`
2. `unique_ptr<Point> p1 = make_unique<Point>();  shared_ptr<Point> p2 = move(p1);`
## `unique_ptr`
unique_ptr 跟 shared_ptr用法一樣，只是他的指標不能共享，是唯一無二的。
### 初始化 :
C++14以後
```C++
unique_ptr<Student> s = make_unique<Student>();
```
 C++14以前。比較不建議。
 ```c++
 unique_ptr<Student> s = unique_ptr<Student>(new Student());
 ```
 不推薦原因 :

> [!NOTE] 
>C++ 編譯器在處理參數時，執行順序是不固定的。編譯器可能會產生如下的執行順序：
>```C++
>process(std::unique_ptr<Student>(new Student()), getPriority());
>```
>1. 執行 `new Student()` (分配記憶體)。 
>2. 執行 `getPriority()`。 
>3. 執行 `std::unique_ptr` 的建構子（接管剛剛分配的記憶體）。 
**問題：** 如果步驟 2 的 `getPriority()` 丟出例外，那麼程式會中斷，步驟 3 永遠不會執行。此時，步驟 1 分配的 `Student` 記憶體就沒人接管，導致 **記憶體洩漏 (Memory Leak)**。
### 函式傳遞 unique_ptr
根據 unique_ptr 是唯一性的原則（不能被複製），所以在參數傳遞時不能使用傳值 call by value，而是要傳參考 call by reference
```c++
void func1(unique_ptr& a) {  
    // ...  
}  
  
void func2(unique_ptr b) {  
    // ...  
}  
  
int main() {  
    func1(a);  
    func2(b); // 編譯錯誤  
    return 0;  
}
```
### 使用 move 轉移 unique_ptr 擁有權
```c++
//class Student;  
  
unique_ptr<Student> s1 = make_unique<Student>();  
unique_ptr<Student> s2 = s1; // 編譯錯誤  
unique_ptr<Student> s3 = move(s1); // 移動到 s2, 之後不能去使用 s1
```
### 函式如何回傳 unique_ptr 
```c++
unique_ptr<Student> func1() {  
    auto s = make_unique<Student>();  
    // or  
    // unique_ptr<Student> s = unique_ptr<Student>(new Student());  
    return s;  
}  
  
int main() {  
    auto s = func1();  
    return 0;  
}
```
### Q : `unique_ptr` 禁止拷貝，那為什麼 return 的時候不會報錯？
>[!NOTE]
>When certain criteria are met, an implementation is allowed to omit the copy/move construction of a class object [...] This elision of copy/move operations, called _copy elision_, is permitted [...] in a return statement in a function with a class return type, **when the expression is the name of a non-volatile automatic object** with the same cv-unqualified type as the function return type [...]
When the criteria for elision of a copy operation are met and the object to be copied is designated by an lvalue, overload resolution to select the constructor for the copy is first performed **as if the object were designated by an rvalue**.
