# Section 2.1 (Introduction)

>[!note] 概念一
>Every value in Python has a _class_ that determines what type of value it is. Values that share a class also share behavior

這邊用類似 abstraction 的想法，相同的 class 有一樣的行為。

>[!note] 概念二
>Native data types have the following properties:
>1. There are expressions that evaluate to values of native types, called _literals_.
>2. There are built-in functions and operators to manipulate values of native types.

在程式語言中，literal（字面量／字面值／直接量) 指的是直接寫在程式碼裡的固定值，它本身就代表那個值，而不是透過變數、運算或函式呼叫才產生。
Native data type 存在可以說明其值的表達式和可以操作該 type 的函式。Python 包含散個 native numeric types:
- integers
- real number
- complex number
舉例:
```python
type(1.5)    # 輸出為 <class 'float'>
type(1+1j)   # 輸出為 <class 'complex'>
```

>[!note] 概念三
>float values should be treated as approximations to real values. These approximations have only a finite amount of precision

對於 floating number 來說，因為只能用有限的位元，我們不能完整的表達所有的實數，所以 floating number 一定有誤差，而誤差會隨著計算的增加而疊加。例如:
```python
7/3*3          # 答案會是 7
1/3 * 7 * 3    # 答案會是 6.999999999999999
```

更具體來說 python 的 float 用 IEEE 754 雙精度 (double) 的標準，所以有以下等式
```python
1/3 == 0.333333333333333312345  # 兩者相等，Beware of float approximation
```

>[!question] 問題一
>在 IEEE 754 雙精度 (double) 的標準下 float 我可以相性幾位數？

先看 IEEE 754 雙精度的結構:
- sign bit: 1 位元。0 為正數 1 為負數。
- exponent: 11 位元。以 2 為底數
- fraction: 52 位元
Sign bit 單純是在看正負號；Exponent 是在看指數的部分（以二為底數），採用偏移表示法，計算方式是其無號值減去 1023 另外保留無號值為 0 跟 2047 這兩情況，看些例子:
- 無號值為 0 對應到: 次正規數 (subnormal number) 跟 0。次正規數是在處理很小的值時在用的，相當於 $0.fraction$ 乘上 $2^{-1022}$
- 無號值為 1023 對應到: 0 次方。
- 無號值為 1025 對應到: 3 次方。
- 無號值為 2047 對應到: NaN 跟 $\infty$。區分 NaN 跟 $\infty$ 的方式是看 fraction，如果 fraction 全為 0 是 $\infty$ ；反之為 NaN
接著討論 fraction 除了 subnormal 以外 fraction 表達的是在科學記號中 1. 後面的部分，寫出來是 $1.fraction \times 2^k$ 裡 fraction 的部分；對於 subnormal 則是 $0.fraction \times 2^{-1022}$ 。介紹完 IEEE 754 規範後，可以來回答這問題，最多只有 53 個位元來表達科學記號中係數的部分，換算回 10 進制有 $\log_10 (2^{53}) = 53 * \log 2 = 15.95$ 所以大概可以確保 15 位精度。至於次方的部分，範圍至少為 $2^{1022}$ 比 fraction 大，所以不考慮。

>[!question] 問題二
>為啥在 exponent 要偏移 $2^{11-1} - 1$?

先回答為啥要向左偏移，根本原因是可以直接用無號數比兩浮點數指數大小，至於為啥選 $2^{11-1} - 1$，原本的範圍是 1 到 2046 (0 跟 2047 保留給特殊值) ，一半可以選 1023 或 1024，如果選 1023 那正數最小是 $2^{-1022}$ 最大是 $2^{1023}$，如此最小值的倒數不會溢位，而選 1024 正數最小是 $2^{-1023}$ 最大是 $2^{1022}$，如此最小值的倒數會溢位，不好。

---
# Section 2.2 (Data Abstraction)

>[!note] 概念一
>The general technique of isolating the parts of a program that deal with how data are represented from the parts that deal with how data are manipulated is a powerful design methodology called data abstraction.

核心思想是「簡化使用者的認知負擔」

>[!note] 概念二
>The basic idea of data abstraction is to structure programs so that they operate on abstract data. That is, our programs should use data in such a way as to make as few assumptions about the data as possible. At the same time, a concrete data representation is defined as an independent part of the program.

這邊的思路是上一個概念在實作面的樣子，把抽象層跟實際操作分離，並藉由一些函數使抽象層可以在不知實作的情況下，拿到想得到的資料。概念類似於網路架構中的「下層架構透過接口向上層提供服務」。

>[!note] 概念三
> Python provides a compound structure called a list, which can be constructed by placing expressions within square brackets separated by commas.

這單純在介紹 python 的一個叫 list 的結構，例子: `[10,20]`

>[!note] 概念四
>The elements of a list can be accessed in two ways.
>1. The first way is multiple assignment
>2. A second method is by the element selection operator

描述兩個存取 list 內的元素常見的方法，第一個是 multiple assignment
```python
coordinates = [10, 20, 30]
x, y, z = coordinates
```
另一個是透過 element selction operator
```python
coordinates = [10, 20, 30]
x = coordinates[0]
y = coordinates[1]
z = coordinates[2]
```

>[!note] 概念五
>An abstraction barrier violation occurs whenever a part of the program that can use a higher level function instead uses a function in a lower level.

盡量用可以用的函數中最抽象的那個函數來操作，這樣在跟改底層實作時受到影響較小，因為在抽象跟實作分離的架構中，我們希望調用的接口是下面那一層的，如果跨層調用在 debug 時會十分複雜，容易 spaghetti 一樣混在一起。

>[!question] 問題一
>這段程式在幹嘛
>```python
>def pair(x, y):
        """Return a function that represents a pair."""
        def get(index):
            if index == 0:
                return x
            elif index == 1:
                return y
        return get
>def select(p, i):
        """Return the element at index i of pair p."""
        return p(i)
>p = pair(20, 14)
>select(p, 0)      # 會輸出 20
>select(p, 1)      # 會輸出 14
>```

這程式是在說資料的抽象表達跟底層實作是分開的，所以現在這含蓄型的表達是行的。要理解這程式需要先理解 python 的萬物皆 object 的想法，object 包括
- 基本數值 (int, string)
- 資料容器 (list, dictionary, tuple, set)
- 函式
- 模組

所有的 object 可以用 `type()` 看型別，`id()` 看該物件的唯一身份識別碼。還有一個概念是「閉包 (closure)」一般函式在做完後，其內的變數會被丟棄，但有閉包後，如果我們還傳該函式，其內的變數會被保留供你在使用該函數時使用。有了「萬物皆 object 」跟「閉包 (closure)」這兩程式，應該看得懂這程式在幹嘛，在 `p = pair(20, 14)` 呼叫了函式 `pair()` 而該函示回傳 `get()` 這個函式本身，其包含 `x,y` 這兩變數和得到 index 的操作流程，由於閉包 `get()` 會使用到的變數會被保留，最後 `select()` 函式在處理時直接套 `get()` 的邏輯，結束。
可以看這邊的值 20 跟 14 沒被儲存到任一個容器內，而是被藏在 `get()` 函式內，所以**資料怎樣被實作不是重點，重要的是處理該資料的操作**。

>[!question] 問題二
>Python 函式中的變數是存在 stack 中嗎?

Python 對於一個變數來說，要存「值」跟「名子（物件的綁定, reference）」。
- 值 (物件本身): 在 CPython 裡，所有物件都配置在 heap 上，不管是不是區域變數
- 名子: 放在 frame 的區域變數槽，生命週期跟著 frame，frame 結束就消失；例外是被捕捉的變數，改存進 heap 上的 cell object，所以能活得比 frame 久。

因為被捕捉的變數存在 heap ，所以該函式執行完後依舊可以存取該變數，而該 heap 上的物件沒被回收掉是因為內層函式的 `__closure__` 一直抓著那個 cell，refcount 不歸零。

---
# Section 2.3 (Sequences)






---
# Section 2.4 (Mutable Data)




---
# Section 2.5 (Object-Oriented Programming)





---
# Section 2.6 (Implementing Classes and Objects)



---
# Section 2.7 (Object Abstraction)





---
# Section 2.8 (Efficiency)





---
# Section 2.9 (Recursive Objects)













