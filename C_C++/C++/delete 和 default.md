### 參考自[C++11 明確地控制預設函式：delete 與 default](https://kheresy.wordpress.com/2014/10/09/c11-default-and-delete/)
## 動機 : 
在 C++03 的時候，如果程式開發者自己定義一個新的類別的話，就算在什麼都沒有寫的情況下，編譯器也會自動產生一些預設的函式，這些函式包括了：
- 預設建構函式（default constructor）  
    `sampleClass()`
- 複製建構函式（copy constructor）  
    `sampleClass( const sampleClass& )`
- 複製指派運算子（copy assignment operator）  
    `sampleClass& operator= ( const sampleClass& )`
- 解構函式（destructor）  
    `~sampleClass()`
當定義了一個類別後，又不希望這個類別可以被用預設的 copy constructor 與 copy assignment operator 複製的話，就必須要自己去定義這兩個函式，**常把他們定義再private，避免被外部呼叫到**
```cpp
class NonCopyable  
{  
public:  
    NonCopyable(){};

private:  
    NonCopyable(const NonCopyable&);  
    NonCopyable& operator=(const NonCopyable&);  
};
```
還有問題 : 
- NonCopyable 本身還是可以使用。
- 由於定義了 copy constructor，所以編譯器不會產生 default constructor，必須自己定義。而一般來說，自定義的 default constructor 效率會比編譯器自行產生的差
## C++11引入`defualt`和`delete`
要建立一個不可以被複製的類別的例子來說，在 C++11 可以寫成：
```cpp
class NonCopyable  
{  
public:  
    NonCopyable() = default;  
    NonCopyable(const NonCopyable&) = delete;  
    NonCopyable& operator=(const NonCopyable&) = delete;  
};
```
- 在 copy constructor 與 copy assignment operator 的宣告後面都加上了「= delete」，藉此讓編譯器知道這兩個函式是不需要的，之後如果呼叫的話，就會產生編譯階段的錯誤。
- 在 default constructor 後面加上了「= default」，事告訴編譯器這邊雖然重新宣告了 default constructor，但是還是要使用編譯器預設產生的版本。
### Q : 刪除拷貝為何需要`NonCopyable(const NonCopyable&) = delete; `
>[!NOTE] 
>「拷貝」這個動作其實包含了兩個完全不同的行為
>1. **`NonCopyable(const NonCopyable&) = delete;` (拷貝建構子)** 它是用一個已存在的物件來**初始化（建立）**一個新的物件。(從無到有的路（建構子）)
>```c++
>NonCopyable a;
NonCopyable b(a); // 呼叫建構子：從無到有建立 b
NonCopyable c = a; // 呼叫建構子：雖然有等號，但這是在「初始化」c
>```
>2. **`NonCopyable& operator=(const NonCopyable&) = delete;` (拷貝賦值運算子)** 它是將一個物件的值，覆蓋掉另一個**已經存在**的物件。(覆蓋舊有的路（賦值運算子）)
>```c++
>NonCopyable a;
NonCopyable b;
b = a; // 呼叫賦值運算子：a 和 b 都已經存在了，只是要把 a 的東西塞進 b
>```

---
## `delete`特殊用途
1. 避免函數傳參時隱性轉換
```cpp
void testFunc( float fVal ){}

int main()  
{  
    double d = 1;  
    int i = 1;

    testFunc(d);  
    testFunc(i);  
}
```
上面代碼會將int, double隱性轉換成float，但我不想要怎麼辦
```cpp
template<typename T>  
void testFunc(T) = delete;

void testFunc( float fVal ){}

int main()  
{  
    double d = 1;  
    int i = 1;

    testFunc(d);  
    testFunc(i);  
}
```
Output :
```text
delete1.cpp: In function ‘int main()’:
delete1.cpp:13:13: error: use of deleted function ‘void testFunc(T) [with T = double]’
   13 |     testFunc(d);
      |     ~~~~~~~~^~~
delete1.cpp:4:6: note: declared here
    4 | void testFunc(T) = delete;
      |      ^~~~~~~~
delete1.cpp:14:13: error: use of deleted function ‘void testFunc(T) [with T = int]’
   14 |     testFunc(i);
      |     ~~~~~~~~^~~
delete1.cpp:4:6: note: declared here
    4 | void testFunc(T) = delete;
      |      ^~~~~~~~
```
2. 避免自定義的類別被用 new 來做動態配置
```cpp
class testClass  
{  
public:  
    void* operator new( std::size_t ) = delete;  
};
```
