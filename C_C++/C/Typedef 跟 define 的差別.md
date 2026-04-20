# `define` 語法
### ref: [#define in C](https://www.geeksforgeeks.org/c/c-define-preprocessor/)
#### For Defining Constants
```C
#define MACRO_NAME value
```
### For Defining Expressions
```C
#define MACRO_NAME (expression within brackets)
```
#### For Defining Expression with Parameters
```C
#define MACRO_NAME(ARG1, ARG2,...) (expression within brackets)
```

#### 例子
```c
#define PI 3.14159265359
#define PI (22/7)
#define CIRCLE_AREA(r) (3.14 * r * r)
```

---
# `Typedef` 語法
### ref: [C typedef](https://www.geeksforgeeks.org/c/typedef-in-c/)

```C
typedef existing_type new_type
```
- `existing_type`: The type that we want to alias (e.g., int, float, struct, etc.).
- `new_type`: The new alias or name for the existing type.

#### 例子
```C
typedef long long ll;

typedef struct Students {
    char name[50];
    char branch[50];
    int ID_no;
} stu;

typedef int* ip;
```

---
# 問題
`typedef` 似乎可以用 `define` 替代，例如
```C
#define ll long long
typedef long long ll;
```
以上兩個寫法一樣

>[!question] `typedef` 似乎可以用 `define` 替代，那 `typedef` 存在的意義在哪?
>它兩本就不是一路人，`define` 是在預處理階段作文本替換，而 `typedef` 是在編譯器解析程式時，把對應關係寫到符號表之中。從這個階段可以看到 `define` 是會把整個檔案的所有符合規則的文本作文字替換，而 `typedef` 會受作用域的規範。最重要的問題是 `typedef` 做的到的 `define` 不一定做得到，例如
>```C
>typedef int * IP;
>IP a, b;
>
>typedef void (*signalhandler)(int);
>signalhandler S1, S2;
>```
>這兩例 `define` 都做不到，第一個例子可能有些疑義，想一下
>```C
>#define IP int* 
>IP a, b;
>```
>`IP a, b;` 會變成 `int *a, b;` 只有 `a`  是 `int` 指針而 `b`  是 `int` 。


>[!question] 寫 `typedef long long int short` 會怎樣?
>在編譯器時會出錯誤
>```txt
>test.c:1:23: error: both ‘long’ and ‘short’ in declaration specifiers
 >   1 | typedef long long int short;
>     |                       ^~~~~
>test.c:1:1: warning: useless type name in empty declaration
 >   1 | typedef long long int short;
 >     | ^~~~~~~
>```

