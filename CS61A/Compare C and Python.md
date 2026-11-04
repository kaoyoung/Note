# Static Programming vs.  Dynamic Programming
### 指定函式返回值型別


### 運算子 (operand) 和運算元 (operator) 分離



### 比較運算元回傳


---
# Operator precedence








---
# Procedural vs. multi-paradigm language






---
# Meomory and Address vs. Mutable Object



---
# Everything is a object
- function is a object
	- using function to do the OOP




---
# Nested Functon




---
# Iterator & Generator





---
# Other
- Python 沒有獨立的字元型別；C 沒有字串型別。
	- 舉體說明可以看這篇文章 [[為何 python 的 char 直接用 string 表達不用 ASCII]]
- string vs. character
- closure
- tuple unpacking
- Names refer to objects.
- function argument evaluation order
- compile-time name resolution vs runtime name resolution
	- **C/C++**：名字必須在**源碼中被使用的位置之前**就已宣告（編譯期、靜態、看文字順序）。定義可以在後，但宣告要在前。
	- **Python**：名字只要在**該行被實際執行的時刻**已綁定即可（執行期、動態、看時間順序）。
	- PYTHON 編譯期只做**作用域分類**（決定執行時去哪查），以及擋掉少數 `global`/`nonlocal` 的**結構性語法錯誤**。
- mutable object
- function vs. callable object
- dunder 方法
- class attribute、instance attribute 可以隨便加