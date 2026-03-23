## 參考自[C++ 的 forward declaration](https://viml.nchc.org.tw/forward-declarations/)
# 緣起

有Imcomplete type，我們可以先告訴編譯器有這樣一個類別存在（宣告），但是這個時候還先不告訴你他的實際內容（定義）。

---
# 使用時機

兩個類別要使用到彼此的時候，通常就會需要用到這種技巧。在使用函式庫時，可以減少建置專案的時間，也可某程度上避免不同函式庫之間重複定義。
```c
class Vector; // forward declaration
 
class Matrix
{
  // ...
  friend Vector operator*(const Matrix&, const Vector&);
};
 
class Vector
{
  // ...
  friend Vector operator*(const Matrix&, const Vector&);
};
```

==Forward declaration也可以剝離實作和使用介面==。

---
# 一般使用含式庫的的方法
假設有一函數庫提供的header(LibA.h)如下 : 
```c++
#pragma once
//#include "A01.h"
// ...
//#include "A09.h"
 
enum ErrorCode
{
  OK,
  Error1,
  Error2
};
 
class LibAObj
{
public:
  ErrorCode func()
  {
    return OK;
  }
  //...
};
```
他提供 : 
- 一個類別 LibAObj 作為主要的操作對象
- func() 的函式
- ErrorCode 這個列舉型別，來描述函示執行的結果。

我們接這用LibA.h來開發我們的函式庫(MyLib.h)
```c++
#pragma once
// MyLib.h
#include "libA.h"
 
class MyLib
{
public:
  bool func()
  {
    if (mObj.func() == OK)
      return true;
    
    return false;
  }
 
protected:
  LibAObj mObj;
};
```
使用時 : 
```c++
#include <iostream>
 
#include "MyLib.h"
 
int main()
{
  MyLib a;
  if (a.func())
    std::cout << "OK" << std::endl;
  else
    std::cout << "Error" << std::endl;
}
```
### 可能遇到的問題 : 
1. **重複定義同名的巨集** : 假設要同時include MyLib.h 和 libB.h，而libB.h也有`enum ErrorCode`。其實可用(Macro Guard)或#pragma once解。
2. **效率問題** : MyLib.h 中會 include libA.h 這個 header、而 libA.h 裡面可能又 include 了更多的 header，這會導致在編譯應用程式的檔案時，編譯器只要遇到有 include MyLib.h 的檔案的時候，就需要去處理 libA.h 有引用到的 header，所以有可能會多花不少時間。拉出一坨粽子。
---
# 使用forward declaration

首先開發我們的函式庫(MyLib.h)
```c
#pragma once
// MyLib.h
#include <memory>
 
class LibAObj; // Forward declaration
 
class MyLib
{
public:
  MyLib();
  ~MyLib();
 
  bool func();
 
protected:
  std::unique_ptr<LibAObj> mObj;
};
```
**修改點** : 
- 不在 header 檔中 `include libA.h`
- 針對 libA.h 需要使用的型別（libAObj）透過 forward declaration 宣告
- 使用 pointer 的形式取代 libAObj 的物件（這邊是用 [smart point](https://kheresy.wordpress.com/2012/03/03/c11_smartpointer_p1/)）
- 將會使用到 libAObj 的內容的部分都從 .h 移到 .cpp（包括 new 和 delete）

對應的MyLib.cpp
```c++
#include "libA.h"
#include "MyLib.h"
 
MyLib::MyLib()
{
  mObj = std::make_unique<LibAObj>();
}
 
MyLib::~MyLib(){}
 
bool MyLib::func()
{
  if (mObj->func() == OK)
    return true;
 
  return false;
}
```
**改動與對應好處** : 
1. libA.h 有關的東西，都會被藏在 MyLib.cpp 裡面 : 和 libA.h 有關的 header 只需要在編譯 MyLib.cpp 才需要處理。如果在專案中有很多檔案有用到 MyLib.h 的時候，這樣的寫法是可以減少編譯時需要的時間的。假設你的專案中有 100 個檔案都用到了 `MyLib`，就算你大改 `libA.h` 的內容，只要 `MyLib.h` 的公開介面 (如 `bool func()`) 不變，那 100 個檔案都**不需要重新編譯**。唯一需要重新編譯的只有 `MyLib.cpp` 這一個檔案。
2. 由於 libA.h 內定義的東西，也都被藏在 MyLib.cpp 裡面 : 所以就算應用程式那邊還有 `include LibB.h`、也有同名的 ErrorCode 列舉型別，也不會造成命名的衝突，因為Mylib.h沒有ErrorCode。

---
# 反對的想法

參考自[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Forward_Declarations)
**Pros** : 
- Forward declarations can save compile time, as `#include`s force the compiler to open more files and process more input.
- Forward declarations can save on unnecessary recompilation. `#include`s can force your code to be recompiled more often, due to unrelated changes in the header.
**Cons** : 
- Forward declarations can hide a dependency, allowing user code to skip necessary recompilation when headers change.
- A forward declaration as opposed to an `#include` statement makes it difficult for automatic tooling to discover the module defining the symbol.
- A forward declaration may be broken by subsequent changes to the library. Forward declarations of functions and templates can prevent the header owners from making otherwise-compatible changes to their APIs, such as widening a parameter type, adding a template parameter with a default value, or migrating to a new namespace.
- Forward declaring symbols from namespace `std::` yields undefined behavior.
- It can be difficult to determine whether a forward declaration or a full `#include` is needed. Replacing an `#include` with a forward declaration can silently change the meaning of code:
```c++
// b.h:
struct B {};
struct D : B {};
 
// good_user.cc:
#include "b.h"
void f(B*);
void f(void*);
void test(D* x) { f(x); }  // Calls f(B*)
```
**解說** : 
 b.h的部分 : 
 + `struct D : B {};`：定義了一個衍生類別 `D`，它公開繼承 (publicly inherits) 自 `B`。
+ **重要概念：** 因為 `D` 繼承自 `B`，所以 `D` 類別的物件 _is-a_ (是一種) `B` 類別的物件。因此，一個指向 `D` 的指標 (`D*`) **可以被隱式地轉換 (implicitly converted)** 成一個指向 `B` 的指標 (`B*`)。
`f(x)`的部分 : 
- `f(B*)` : 這是一個「衍生類別到基礎類別的轉換」(Derived-to-Base Conversion)**。
- `f(void*)` : 這是一個「標準指標轉換」(Standard Pointer Conversion)**。
選擇`f(B*)` : 
- `D*` 轉換到 `B*` 被認為是**更精確 (more specific)** 的匹配。    
- `D*` 轉換到 `void*` 被認為是**更通用 (more general)** 的匹配。   
**核心規則：** 在 C++ 中，**「衍生類別到基礎類別的轉換」(`D*` -> `B*`)** 的優先級**高於**「標準指標轉換」(`D*` -> `void*`)。
**如果使用forward declatation** : 
如果把 `include “b.h` 改成 B 和 D 的 forward declaration 的話，`test()` 就會不知道 B 和 D 的繼承關係，而不會去呼叫 `f(B*)`、而改去呼叫 `f(void*)`。

---
# 例子 
## Jserv大大在[你所不知道的 C 語言：指標篇](https://hackmd.io/@sysprog/c-pointer)的forward declaration 搭配指標的技巧中的例子。
### oltk.h
```c
struct oltk; // 宣告 (incomplete type, void) 
struct oltk_button; 
typedef void oltk_button_cb_click(struct oltk_button *button, void *data); 
typedef void oltk_button_cb_draw(struct oltk_button *button, struct oltk_rectangle *rect, void *data); 
struct oltk_button *oltk_button_add(struct oltk *oltk, int x, int y, int width, int height);
```
`struct oltk` 和 `struct oltk_button` 沒有具體的定義 (definition) 或實作 (implementation)，僅有宣告 (declaration)。
### oltk.c
```c
struct oltk { 
	struct gr *gr; 
	struct oltk_button **zbuttons; 
	... 
	struct oltk_rectangle invalid_rect; 
};
```
### 想法 (實作和使用介面分離)
軟體界面 (interface) 揭露於 oltk.h，不管 struct oltk 內容怎麼修改，已公開的函式如 `oltk_button_add` 都是**透過 pointer 存取給定的位址，而不用顧慮具體 struct oltk 的實作**，如此一來，不僅可以隱藏實作細節，還能兼顧二進位的相容性 (binary compatibility)。

同理，struct oltk_button 不管怎麼變更，在 struct oltk 裡面也是用給定的 pointer 去存取，保留未來實作的彈性。