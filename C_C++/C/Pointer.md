## 參考自jserv的[你所不知道的C語言：指標篇](https://hackmd.io/@sysprog/c-pointer)
# 為何C語言要這樣設計

這個問題可從三個面向去思考 : 
1. 開發UNIX
2. 充分掌握硬體
3. 思維方式、文化(UNIX)
---
# 頭腦體操

用內部爆炸法去思考，由內向外推進。
## 例子一
```c
(*(void(*)())0)();

// 可以改寫成
typedef void (*funcptr)(); 
(* (funcptr) 0)();
```
意思 : 呼叫於記憶體位置0的函式。
### 拆解過程
1. `void(*)()` : 回傳void且無參數的函式指針。
2. `(void(*)()) 0`：將數字 `0` 強制轉換為上述型別的指針(一個指向位址 0 的回傳void且無參數的函式指針)。
3. `(*(void(*)()) 0)`：解參考(一個指向位址 0 的回傳void且無參數的函式指針)
4.  `(*(void(*)()) 0)` () : 執行該函式
5. 
### Q1 .為啥是 `typedef void (*funcptr)(); ` 而不是`typedef void (*)() funcptr; ?

>C 語言的 `typedef` 語法遵循一個核心原則：「**宣告模仿使用 (Declaration mimics use)**」。
>人話就是用typedef一個類型，就跟宣告一個該類型的變數一樣，例子如下: 
```C
void (*funcptr)();  //宣告一個funcptr變數
typedef void (*funcptr)(); // 定義funcptr為void (*)()的型別名稱

int a;
typedef int a;
```
[[型別]]是啥。
### Q2. 為啥`(* (funcptr) 0)()`解讀順序不為先`* (funcptr)` 解引再處理0
>因為不能解參考一個型別，型別又不是一個地址。

## 例子二
```c
void **(*d) (int &, char **(*)(char *, char **));
```
這事c++語法。
- d is a pointer to a function that takes two parameters:
    - a reference to an int and
    - a pointer to a function that takes two parameters:
        - a pointer to a char and
        - a **pointer to a pointer** to a char
    - and returns a pointer to a pointer to a char
- and returns a pointer to a pointer to void
## 例子三
```c
void ( *signal(int sig, void (*handler)(int)) ) (int); 

// another version
typedef void (*sighandler_t)(int);
sighandler_t signal(int sig, sighandler_t handler);
```
解答參考自 [How to read this prototype?](https://stackoverflow.com/questions/15739500/how-to-read-this-prototype)
```c
signal                                          -- signal
        signal(                               )         -- is a function
        signal(    sig                        )         -- with a parameter named sig
        signal(int sig,                       )         --   of type int
        signal(int sig,        handler        )         -- and a parameter named handler
        signal(int sig,       *handler        )         --   which is a pointer
        signal(int sig,      (*handler)(   )) )         --   to a function
        signal(int sig,      (*handler)(int)) )         --   taking an int parameter
        signal(int sig, void (*handler)(int)) )         --   and returning void
       *signal(int sig, void (*handler)(int)) )         -- returning a pointer
     ( *signal(int sig, void (*handler)(int)) )(   )    -- to a function
     ( *signal(int sig, void (*handler)(int)) )(int)    --   taking an int parameter
void ( *signal(int sig, void (*handler)(int)) )(int);   --   and returning void
```
➡️ **接受**：一個訊號編號 + 一個 handler  
➡️ **回傳**：舊的 handler（或 `SIG_ERR` 表示錯誤）
**例子**
```c
#include <signal.h>

static int interrupt = 0;

/**
 * The following function will be called when a SIGINT is
 * detected (such as when someone types Ctrl-C)
 */
void interrupt_handler( int sig )
{
  interrupt = 1;
}

int main( void )
{
  /**
   * Declare a pointer to the old interrupt handler function
   */
  void (*old_interrupt_handler )(int);

  /**
   * Save the old interrupt handler while setting the new one
   */
  old_interrupt_handler = signal( SIGINT, interrupt_handler );
  while ( !interrupt )
  {
    // do stuff until someone hits Ctrl-C
  };

  /**
   * restore the original interrupt handler
   */
  signal( SIGINT, old_interrupt_handler );
  return 0;
}
```
---
# C規格書的觀念
## Object

>region of data storage in the execution environment, the contents of which can represent values

object != object-oriented
- 前者的重點在於「資料表達法」，後者的重點在於 "everything is object"
## Storage durations of objects

> An object has a storage duration that determines its lifetime. There are three storage durations: static, automatic, and allocated.

>The lifetime of an object is the portion of program execution during which storage is guaranteed to be reserved for it. An object exists, has a constant address and retains its last-stored value throughout its lifetime. If an object is referred to outside of its lifetime, the behavior is undefined.

在 object 的生命週期以內，其存在就意味著有對應的**常數**記憶體位址。注意，**C 語言永遠只有 call-by-value**。常數才能確保找的位置不會跑掉，找到位置抄家找資料。
## Types

> A pointer type may be derived from a **function type**, an **object type**, or an **incomplete type**, called the referenced type. **A pointer type describes an object whose value provides a reference to an entity of the referenced type**. A pointer type derived from the referenced type T is sometimes called ‘‘pointer to T’’. The construction of a pointer type from a referenced type is called ‘‘pointer type derivation’’.

這是c語言中call-by-value的實證(A pointer type describes an object whose **value** provides a reference to an entity of the referenced type.)，就算是指針傳的也只是地址的值而已。

>Arithmetic types and pointer types are collectively called scalar types. Array and structure types are collectively called aggregate types.

我們知道scalar types分為**pointer type**跟**arithmetic type**，又因為在Multiplicative operators(\*、/、%)中規定(Each of the operands shall have arithmetic type.)，所以
```c
char A = 'a';
char *ptrA = &A;

++ptrA; // legal operation
--ptrA; // legal operation

ptrA = ptrA*1; // illegal operation
prtA = ptrA/1; // illegal operation
```
這樣設計的理由是因為`ptrA`的起始位置是隨機且為一正值，在做乘法時會把這起始位置乘進去整個值會變得沒啥意義；另一方面在做加減法時，會加減整個type的大小，是有意義因為跟起始位置無關且可做在矩陣取值。

>An array type of unknown size is an incomplete type. It is completed, for an identifier of that type, by specifying the size in a later declaration (with internal or external linkage). **A structure or union type of unknown content is an incomplete type**. It is completed, for all declarations of that type, by declaring the same structure or union tag with its defining content later in the same scope.

**例子** : `int b[]、char c[]、struct GraphicsObject`
**用途** : [[forward declaration 技巧的]]原理，在標頭檔宣告`struct GraphicsObject;` 不給細節然後`struct GraphicsObject *initGraphics(int width, int height);` 是合法的，但 `struct GraphicsObject obj;` 不合法
**總結** : 因為沒有細部定義，所以是 Incomplete Type，沒辦法建立實體，卻可以用指標。(給個大秘寶去找吧)

>Array, function, and pointer types are collectively called derived declarator types. A declarator type derivation from a type T is the construction of a derived declarator type from T by the application of an array-type, a function-type, or a pointer-type derivation to T.

array function pointer 其實是一樣的，重點都在位址。

**練習** : 設定絕對地址為 `0x67a9` 的 32-bit 整數變數的值為 `0xaa6`，該如何寫？
```c
*(int32_t * const) (0x67a9) = 0xaa6; /* Lvalue */
```
甚麼是lvalue，參考[[lvalue_rvalue]]
#### Q: 如果寫`*(int32_t *) (0x67a9) = 0xaa6;` 會怎樣?
#### A: 會一樣，在\*旁的const指確保指標本身不被修改。

>A pointer to void shall have the same representation and alignment requirements as a pointer to a character type.

規範 `void *` 和 `char *` 彼此可互換的表示法。
**例子** : `void *memcpy(void *dest, const void *src, size_t n);`
**標準規定** : 
	- `void *` 可以隐式转换为任何对象指针类型
	- 任何对象指针类型可以隐式转换为 `void *`
	- `char *` 与 `void *` 有相同的表示和对齐要求(代表char*、void*都是一個byte)。
**注意** : `void *` 跟其他對象指針間轉型時對齊大小的問題。
#### Q : 為何規定`void *` 和 `char *` 彼此可互換?
#### A :  確保向下相容性 (backward compatibility) 和互通性 (interoperability)，在 `void*` (泛型指標) 被正式加入 C 語言標準之前，**`char*` (字元指標) 一直被當作「泛型指標」來使用**，都是讀一個byte。

---
# `void *` 之謎

`void` 在最早的 C 語言是不存在的，直到 C89 才確立。

### Q : 為何要設計這樣的型態呢？
### A : [最早的 C 語言中](https://www.bell-labs.com/usr/dmr/www/primevalC.html)，任何函式若沒有特別標注返回型態，一律變成 `int` (伴隨著 `0` 作為返回值)，但這導致無從驗證 [function prototype](https://en.wikipedia.org/wiki/Function_prototype) 和實際使用的狀況

`void *` 的設計，導致開發者必須透過 ==explicit (顯式)== 或強制轉型，才能存取最終的 object，否則就會丟出編譯器的錯誤訊息，從而避免危險的指標操作。
```c
void *p = ...; 
void *p2 = p + 1; /* what exactly is the size of void? */
```
`void *` 無法透漏要取幾byte的資訊。在`gcc -pedantic test.c` 的編譯下跳緊告`warning: pointer of type ‘void *’ used in arithmetic`。

**硬體架構的對齊 :**  在32-bit(uint32_t)讀取uint16_t。
```c
/* may receive wrong value if ptr is not 2-byte aligned */ /* portable way of reading a little-endian value */ uint16_t value = *(uint16_t *) ptr; uint16_t value = *(uint8_t *) ptr | ((*(uint8_t *) (ptr + 1)) << 8);
```

## `void *` 真的萬能嗎？

>A pointer to a function of one type may be converted to a pointer to a function of another type and back again; the result shall compare equal to the original pointer. If a **converted pointer is used to call a function whose type is not compatible with the pointed-to type, the behavior is undefined.**

C99 不保證 **pointers to data** (in the standard, "objects or incomplete types" e.g. `char *` or `void *`) 和 **pointers to functions** 之間相互轉換是正確的。

--- 
# 沒有「雙指標」只有「指標的指標」

「雙馬尾」(左右「獨立」的個體) 和「馬尾的馬尾」(由單一個體關聯到另一個體的對應) 不同

- 漢語的「[雙](https://www.moedict.tw/%E9%9B%99)」有「對稱」且「獨立」的意含，但這跟「指標的指標」行為完全迥異
- 講「==雙==指標」已非「懂不懂 C 語言」，而是漢語認知的問題

C 語言中，**萬物皆是數值 (everything is a value)**，函式呼叫當然只有 call-by-value。「指標的指標」(英文就是 a pointer of a pointer) 是個常見用來改變「傳入變數原始數值」的技巧。

---
# Pointer vs. Arrays
## 概覽
- in declaration
    - extern, 如 `extern char x[];` → 不能變更為 pointer 的形式
    - definition/statement, 如 `char x[10]` → 不能變更為 pointer 的形式
    - parameter of function, 如 `func(char x[])` → 可變更為 pointer 的形式 → `func(char *x)`
- in expression
    - array 與 pointer 可互換

```c
int main() { 
	int x[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}; 
	printf("%d %d %d %d\n", x[4], *(x + 4), *(4 + x), 4[x]); }
```
`x[i]`只是語法糖，總是被編譯器改寫為 `*(x + i)`← in expression。

在 [The C Programming Language](http://www.amazon.com/The-Programming-Language-Brian-Kernighan/dp/0131103628) 第 2 版，Page 99 寫道:

> As formal parameters in a function definition,

該書 Page 100 則寫:

> `char s[]`; and `char *s` are equivalent.

**理解 :** 「陣列 (Array) 宣告」和「指標 (Pointer) 宣告」只有在「函式參數」這個特定情境下，才是等價的。
**核心理念 :** C 語言函式永遠不會「複製」整個陣列來傳遞。
**例子 :** 這兩寫法都一樣
```c
// 寫法一 : 
void my_function_array(char s[]) { 
	// s 是一個指標，所以可以做指標運算
	char c = s[0]; // OK 
	s++; // OK // 因為 s 是一個指標，sizeof(s) 是指標的大小 (例如 8 bytes) 
}

// 寫法二 : 
void my_function_pointer(char *s) { 
	// s 是一個指標 
	char c = s[0]; // OK 
	s++; // OK // s 是一個指標，sizeof(s) 是指標的大小 (例如 8 bytes) 
}
```