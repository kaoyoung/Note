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

>[!note] 概念一
>There are many kinds of sequences, but they all share common behavior. In particular,
>- **Length.** A sequence has a finite length. An empty sequence has length 0.
>- **Element selection.** A sequence has an element corresponding to any non-negative integer index less than its length, starting at 0 for the first element.

先給 sequences 的抽象想法:「一個有順序、可以用索引取出元素的物件」。在 Python 中有三個常見的物件
- list
- range
- string

>[!note] 概念二
>List 常見操作
>- `len()`
>- lists can be added together and multiplied by integers
>- element selection can be applied multiple times in order to select a deeply nested element.
>- `for` statement can iterate over the element values directly without introducing the name `index` at all.
>- Sequence unpacking: A `for` statement may include multiple names in its header to "unpack" each element sequence into its respective elements.
>- list constructor
>- Aggregation: Aggregate all values in a sequence into a single value. (`sum()`、`min()`、`max()` 等 builtin 的函式)
>- Higher-Order Function
>- Membership: A value can be tested for membership in a sequence. (`in`、`not in`)
>- Slicing: A slice of a sequence is any contiguous span of the original sequence, designated by a pair of integers.

List 本身是一個 sequence 所以他必須要有兩件事 length 跟 element selection。這兩件事可以看以下例子
```python
digits = [1, 8, 2, 8]
print(len(digits))    # 4
digits[3]

pairs = [[10, 20], [30, 40]]
print(pairs[1])    # [30, 40]
print(pairs[1][0]) # 30
```

過來看一些 list 常見用法，例如: addition、multiplication、for iteration、list constructor
```python
# addition、multiplication
digits = [1, 8, 2, 8]
print([2, 7] + digits*2)

# for iteration
def count(s, value):
       """Count the number of occurrences of value in sequence s."""
       total = 0
       for elem in s:
           if elem == value:
               total = total + 1
       return total
       
# list constructor
print(list(range(4)))  # [0, 1, 2, 3]
```

Sequence unpacking 是在用 `for` 做 iteration 時的小技巧，如果 list 內的元素格式固定，則可以在 iteration 中把 list 內的元素命名成想要的變數
```python
pairs = [[1, 2], [2, 2], [2, 3], [4, 4]]
for x, y in pairs:
        if x == y:
            same_count = same_count + 1
```

Higher-Order Function 只是把 list 跟 function 結合，使得表達更加的簡潔。
```python
def keep_if(filter_fn, s):
        return [x for x in s if filter_fn(x)]
```

Membership 跟 slicing 提供了 list 更多的操作，延伸了 list 的 abstraction
```python
digits = [1, 8, 2, 8]
print(2 in digits)  # True

print(digits[0:2])  # [1, 8]
print(digits[1:])   # [8, 2, 8]
print(digits[:-1])  # [1, 8, 2]
print(digits[0:])   # [1, 8, 2, 8]
```

>[!note] 概念三
>A range is another built-in type of sequence in Python, which represents a range of integers.


>[!note] 概念四
>String 的基本操作
>- String literals can express arbitrary text, surrounded by either single or double quotation marks
>- `len()`
>- element  selection
>- String can also be combined via addition and multiplication
>- Membership
>- Multiline Literals: String aren't limited to a single line
>- String Coercion

String 的基本表示法如下，可以用單引號 (`'`) 或雙引號 (`"`)。
```python
'I am string!'
"I've got an apostrophe"
'您好'
```
需要注意的是字串內部用的引號要跟外部的不同，不然要用跳脫字符 (反斜線)。
String 也是 sequence 的一種，所以他也有 length 跟 element election (string 的 `len()` 是算 unicode code point 的數目)
```python
city = 'Berkeley'
print(len(city))  # 8
print(city[3])    # k
```
跟 list 一樣，string 有 addition、multiplication 跟 membership
```python
print('Berkeley' + ', CA')   # Berkeley, CA
print('Shabu ' * 2)   # Shabu Shabu 

'here' in "Where's Waldo?"  # True
```

String 可以用三重雙引號 (Triple quotes) 支援多行字串變成一字串 (string literal)
```python
print("""The Zen of Python
claims, Readability counts.
Read more: import this.""")
```
輸出是
```txt
The Zen of Python
claims, Readability counts.
Read more: import this.
```

可以透過 `str()` 把 object 轉成 string
```python
digits = [1, 8, 2, 8]
print(str(2) + ' is an element of ' + str(digits))
```

>[!question] 問題一
>"String literals can express arbitrary text, surrounded by either single or double quotation marks." 這句話真的表示單引號跟雙引號功能一樣嗎 ?

單雙引號是等價的寫法，差別在於跳脫 (escape) 的便利性，字串內用單引號，外部就要用雙引號，同理在雙引號上。

>[!question] 問題二
>Python 中沒有 character 的概念嗎 ? 是比較 C/C++ 跟 Python 對於 string 觀念的差異 ?

在 python 中只有 string 沒有 character，`a` 只是一個長度為一的字串。在 python 中不用 `\0` 放在字串尾巴表示字串終結，因為 python 的 string 本身帶長度的訊息，所以可以直接判斷是不是在字串的範圍。

>[!note] 概念五
>In general, a method for combining data values has a **closure property** if the result of combination can itself be combined using the same method.

這概念在說如果一個資料結構可以成為下一次創造資料結構的元素，那便可以無限套娃出一個層狀的結構。
```python
nested = [[1, 2], [],
		 [[3, False, None],
		 [4, lambda: 5]]]
```

---
# Section 2.4 (Mutable Data)

>[!note] 概念一
>One powerful technique for creating modular programs is to incorporate data that may change state over time.

如同現實世界中的實體一樣，程式中的資料會隨著程式的執行而改變其狀態，所以我們想讓一物件帶有隨程式執行改變其狀態的特性。在帶有狀態的系統中，我們想藉由一個個帶有狀態的子物件，把原系統改模組化，讓子物件可以自己處理自己範圍內的狀態，如此減少整個系統的複雜度。

>[!note] 概念二
>_Objects_ combine data values with behavior. Objects represent information, but also _behave_ like the things that they represent.

給出 object 的抽象想法，它是一個帶有資料和行為的東西，我們把它視為一現實或想像中的東西在程式裡的投射。看下面這例子

```python
from datetime import date

tues = date(2014, 5, 13)
print(date(2014, 5, 19) - tues)      # 6 days, 0:00:00
tues.year                            # 2014
tues.strftime('%A, %B %d')           # 'Tuesday, May 13'
```
從上面這程式可以看出許多 object 的特性:
- `date` 本身就是 class (或是叫 object 因為 python 中 everything is a object)，透過 `date(2014, 5, 13)` 造出一個實例 (instance)
- attribute: 代表該 object 某一個被命名變數的值，`tues.year` 表示 `tues` 這個 instance 中被命名為 year 變數的值。語法是: `<expression> . <name>`
- method: 值本身是函式的 attribute（function-valued attribute），`tues.strftime('%A, %B %d')` 由 `'%A, %B %d'` 提供輸出字串的樣式，而 `tues` 提供輸出所需的值。這想法正如文中所說的 "By bundling behavior and information together, this Python object offers us a convincing, self-contained abstraction of a date."

>[!note] 概念三
>All values in Python are objects. That is, all values have behavior and attributes. They act like the values they represent.

這觀念想表達 python 的「everything is a object」思想，舉凡: number, string, list, and ranges 等都是 object

>[!note] 概念四
>**Sharing and Identity.** Because we have been changing a single list rather than creating new lists, the object bound to the name chinese has also changed, because it is the same list object that was bound to suits!

這觀念是在描述以下程式
```python
chinese = ['coin', 'string', 'myriad']
suits = chinese
suits.pop()
suits.remove('string')
suits.append('cup')
suits.extend(['sword', 'club'])
suits[2] = 'spade'
suits[0:2] = ['heart', 'diamond']
print(chinese)   # ['heart', 'diamond', 'spade', 'club']
```

當 `suits = chinese` 時，`suits` 這個名字就 refer to `chinese` 所 refer to 的那一個物件 (`x`)，又 list 是 mutable data，所以後續對於 `suits` 的操作都是對 `x`，因此 `print(chinese)` 會是 `['heart', 'diamond', 'spade', 'club']` 而不是原來的 `['coin', 'string', 'myriad']`。
用 python reference 的原文來解釋
- "Names refer to objects. Names are introduced by name binding operations": 名字只是用來指向物件的，不像 C/C++ 會分配記憶體 
- Objects whose value can change are said to be _mutable_; objects whose value is unchangeable once they are created are called _immutable_. (The value of an immutable container object that contains a reference to a mutable object can change when the latter’s value is changed; however the container is still considered immutable, because the collection of objects it contains cannot be changed.)
	-  numbers, strings and tuples are immutable
	- dictionaries and lists are mutable
- An assignment statement evaluates the expression list (remember that this can be a single expression or a comma-separated list, the latter yielding a tuple) and assigns the single resulting object to each of the target lists, from left to right. 
最後一點說明 `=` 只是 assign 右側的 object 給左側，而在第一點說明名字只是用來指向物件，所以 `=` 只是讓名字指向物件。第二點則說 list 是 mutable 所以可以直接改變其值，如果是 immutable 那需要造出一個新物件，來表達更改後的值，所以如果是以下情況
```python
chinese = ('coin', 'string', 'myriad')
suits = chinese
```
任何改變 `suits` 的操作不影響 `chinese` 只的物件，因為 tuple 是 immutable。

>[!question] 問題一
>如何看兩物件是不是同一個 ? 既然 `=` 不能複製出一個新物件，那如果我想要在 python 中複製一個 list 物件如何做 ?

對於第一個問題可以用 `is` 、`is not` 或是 `id()` 來判斷，因為 python 保證物件的 id 唯一。第二個問題可以用 list constructor 來處理，看以下例子
```python
suits = ['heart', 'diamond', 'spade', 'club']
nest = list(suits)
nest[0] = suits
suits.insert(2, 'Joker')

print(suits) # ['heart', 'diamond', 'Joker', 'spade', 'club']
print(nest) # [['heart', 'diamond', 'Joker', 'spade', 'club'], 'diamond', 'spade', 'club']

print(suits is nest) # False
print(id(suits) == id(nest)) # False
print(suits is nest[0]) # True
```

>[!note] 概念五
>Tuple 的概念
>- Tuples are created using a tuple literal that separates element expressions by commas. Parentheses are optional but used commonly in practice. 
>- Tuples have a finite length and support element selection. 
>- While it is not possible to change which elements are in a tuple, it is possible to change the value of a mutable element contained within a tuple.
>- Tuples are used implicitly in multiple assignment. An assignment of two values to two names creates a two-element tuple and then unpacks it.

第一點說明 tuple 的重點是逗號，符合 python reference 說的 "Note that tuples are not formed by the parentheses, but rather by use of the comma."，所以只有一個元素的 tuple 也要 comma (空 tuple 是例外)
```txt
>>> 1, 2 + 3
(1, 5)
>>> ()    # 0 elements
()
>>> (10,) # 1 element
(10,)
```

第二點說合 tuple 常見操作
```txt
>>> code = ("up", "up", "down", "down") + ("left", "right") * 2
>>> len(code)
8
>>> code[3]
'down'
>>> code.count("down")
2
>>> code.index("left")
4
```

第三點說明 tuple 是只其內的物件不變，但如果在 tuple 內的物件 (x) 是 mutable 則 x 內的物件是可變， tuple 只保證 x 不被掉包。

>[!note] 概念五
>Dict 基本概念
>- The purpose of a dictionary is to provide an abstraction for storing and retrieving values that are indexed not by consecutive integers, but by descriptive keys
>- Dictionaries are unordered collections of key-value pairs.
>- The dictionary type also supports various methods of iterating over the contents of the dictionary as a whole. The methods keys, values, and items all return iterable values.
>- dictionary constructor
>- restriction:
>	- A key of a dictionary cannot be or contain a mutable value.
>	- There can be at most one value for a given key
>- A useful method implemented by dictionaries is get, which returns either the value for a key, if the key is present, or a default value.
>- Dictionaries also have a comprehension syntax analogous to those of lists.

先看一下 dictionary 常規操作 (注意 key 的順序是實作決定的，可能不是你心裡想的那樣。前句在 python 3.7 後是錯的 "Dictionaries preserve insertion order. Note that updating a key does not affect the order. Keys added after deletion are inserted at the end." key 會依照插入順序處理)
```python
numerals = {'I': 1.0, 'V': 5, 'X': 10}
print(numerals['X'])     # 10

numerals['I'] = 1
numerals['L'] = 50
print(numerals)          # {'I': 1, 'V': 5, 'X': 10, 'L': 50}
```

dict 中的 keys、values、items 都是可以迭代的，所以在使用會迭代 dict 的函數前要指定是誰被迭代
```python
numerals = {'I': 1, 'V': 5, 'X': 10, 'L': 50}
print(sum(numerals.values()))      # 66
```

可以用 dict constructor 來創造 dict 物件
```python
print(dict([(3, 9), (4, 16), (5, 25)]))   # {3: 9, 4: 16, 5: 25}
```
其中的 tuple 提供了 key-value 的結構。

在 dict 中常見的 `get(key, default value)` ，如果找到該 key 則回傳其值，否則回傳 default value (如果不寫 default value 則默認是 None)
```python
numerals = {'I': 1, 'V': 5, 'X': 10, 'L': 50}
print(numerals.get('A', 0))    # 0
print(numerals.get('A'))       # None
print(numerals.get('V', 0))    # 5
```

講一下 dict 的 comprehension syntax
```python
print({x: x*x for x in range(3,6)})   # {3: 9, 4: 16, 5: 25}

def divide(quotients, divisors):
	return {q: [d for d in divisors if d % q == 0] for q in quotients}
```

>[!note] 概念六
>The nonlocal statement declares that whenever we change the binding of the name `balance`, the binding is changed in the first frame in which balance is already bound. The nonlocal statement indicates that the name appears somewhere in the environment other than the first (local) frame or the last (global) frame.

這概念在說明以下程式
```python
def make_withdraw(balance):
	def withdraw(amount):
		nonlocal balance
		if amount > balance:
			return 'Insufficient funds'
		balance = balance - amount
		return balance
	return withdraw
```
`withdraw()` 需要在外層的變數 `balance`，所以需要關鍵字 `nonlocal` 跟程式說該變數外層去外層找。`nonlocal` 的變數不能在 Local frame 或是 global frame，不然會出現 syntaxerror。

>[!note] 概念七
>Assignment statements already had a dual role: they either created new bindings or re-bound existing names.

引入 nonlocal 關鍵字後，我們可以讓 inner 的賦值 (`x = ...`) 可以 re-bind enclosing frame 裡的那個名子，local frame 不會多出一個自己的 `x`，如此該函式可能在不經意間改動 enclosing 的變數導致該函式變為 non-pure function，但這讓我們的操作變得更加有彈性。
在 python lookup of names 有一個限制 "within the body of a function, all instances of a name must refer to the same frame." 所以不能在沒用 nonlocal 的情況下，對於 enclosing frame的名子在 local frame 賦值。
**注: enclosing frame 本身不包括 local frame**

>[!important] Closure 概念重溫
>在 nested function 內的函式是可以藉由名稱存取 closure 範圍內的物件並進行操作，如下例子:
>```python
>def outer():
 >    data = []
>    def inner():
>        data.append(1)
>        print(data)
 >   return inner
>f = outer()
>f()    # [1]
>f()    # [1,1]
>```
>但在 closure 範圍內如果沒用 `nonlocal` 不能 re-bound 現有名子


>[!note] 概念八
>Non-local assignment is an important step on our path to viewing a program as a collection of independent and autonomous _objects_, which interact with each other but each manage their own internal state.

這由 non-local 我們可以讓 inner function 可以改動外層 frame 裡的變數，如此 function 之間可以用這些變數維護抽象的訊息，同時各自內部的變數，可以保留本地的狀態。需小心 nonlocal 拿到的 enclosing frame 的變數是所有定義在同一個 enclosing frame 裡且指涉同一變數的 nested function 共享的，會讓程序變成順序相依 (order-dependence) 跟失去 referential transparency ( its value does not change if we substitute one of its subexpression with the value of that subexpression)，因為 non-local 變數可能隨著呼叫而改變。


>[!note] 概念十
>The key to correctly analyzing code with non-local assignment is to remember that only function calls can introduce new frames. Assignment statements always change bindings in existing frames.

這觀念是在說明這程式
```python
def make_withdraw(balance):
	def withdraw(amount):
		nonlocal balance
		if amount > balance:
			return 'Insufficient funds'
		balance = balance - amount
		return balance
	return withdraw

wd = make_withdraw(12)
wd2 = wd
print(wd2(1))    # output: 11
print(wd(1))     # output: 10
```

這程式只有在 `nake_withdraw()` 函式呼叫時才創造出存放 balance 的那個 frame (f1)，之後每一次 `withdraw` 也會產生各自的 frame，但他們透過 `nonlocal` 去改的是 f1 裡的 balance。 `wd2 = wd` 只是把 `wd2` 這名子 bind 到 `wd` 那個函式物件而已，所以之後對 `wd2` 的操作都是對 f1 的。

>[!note] 概念十一
>_Constraint-based system_: It supports computation in multiple directions
>_Declarative programming_:   programmer declares the structure of a problem to be solved, but abstracts away the details of exactly how the solution to the problem is computed.

介紹兩個名詞。_Constraint-based system_ 是想跟一般命令 (imperative)程式做區分，他不一定要按照一定的輸入輸出順序去做運算，只要計算結果符合規範就行，例如: `a + b = c` 只要給出兩個，他就算出剩下那一個。_Declarative programming_ 給定一限制和充足的條件後，他可以自己計算出剩下的未知數，他指宣告「問題的結構、有哪些關係等」不去管「具體怎樣去算」。

>[!question] 問題二
>為啥 python 要設計 `list` 、`dict` 等 mutable 的物件? 這在函式間傳參數時，要時刻擔心該物件被修改?

這些 mutable 的想法是想面對概念一說的「資料會隨著程式的執行而改變其狀態」，而在函式間傳遞物件會被修改是因為，Python 中變數是綁在物件上的名子，賦值從不複製物件。這設計有以下兩個常見盲區:
第一個盲區: 
```python
def f(lst):
    lst.append(4)      # 修改原物件 → 外部看得到
    lst = [9, 9]       # 重新綁定名字 → 外部看不到

x = [1, 2, 3]
f(x)
print(x)  # [1, 2, 3, 4]
```
在 `f()` 內第二行的賦值，會把 `lst` 從新綁訂到 `[9, 9]` 這物件，而原始 `lst` refer to 的物件還在，綁訂到 `x` ，所以 `print(x)` 會輸出 `[1, 2, 3, 4]`。
第二個盲區:
```python
def f(x, acc=[]):   # 危險!預設值只建立一次
    acc.append(x)
    return acc
```
這個 `acc=[]` 是 default assignment在**函式定義時**(也就是 `def` 那一行被執行時)就建立好，並存在函式物件 (可以用 `f.__defaults__` 看到)，所以如果後續在用 default assignment 時會用同一個 list，除非你傳自己的 list

>[!question] 問題三
>對於以下程式
>```python
>def f(lst):
>    lst.append(4)      # 修改原物件 → 外部看得到
>    lst = [9, 9]       # 重新綁定名字 → 外部看不到
>x = [1, 2, 3]
>f(x)
>```
>為何不會出現 unbounderror? 我記得在函式內賦值時，會讓該變數綁定在 local frame

你記得規則是對的，但函式在呼叫 (`f(x)`) 時，就已經把 `lst` 綁到 `x` 所指的物件上，所以 `lst.append(4)` 是對 `x` 所指物件的操作，`lst = [9, 9]` 是把 `lst` 重新綁到 `[9, 9]` 這物件上。



---
# Section 2.5 (Object-Oriented Programming)

>[!note] 概念一
>class 常見名詞如下
>- The act of creating a new object instance is known as _instantiating_ the class
>- An _attribute_ of an object is a name-value pair associated with the object, which is accessible via dot notation.
>	- In the broader programming community, instance attributes may also be called _fields_, _properties_, or _instance variables_.
>- Functions that operate on the object or perform object-specific computations are called methods.
>	- We say that methods are _invoked_ on a particular object.

這邊單純在介紹 class 的名詞，例子如下
```python
a = Account('Kirk')   # instantiating
a.holder              # attribute/field/property/instance variable
a.deposit(15)         # invoke the method
```

>[!note] 概念二
>When a class statement is executed, a new class is created and bound to \<name\> in the first frame of the current environment.

class 的模板如下
```txt
class <name>:
    <suite>
```
這觀念想說的是，當這 class 被執行時 (程序碰到)，一個新的 class 物件便被造出來，綁定的名稱便是 `<name>` ，而綁定的位置便是當下執行的那個 frame。

>[!note] 概念三
>The method that initializes objects has a special name in Python, `__init__` (two underscores on each side of the word "init"), and is called the _constructor_ for the class.

這說明 class 中的 `__init__()` 函式，他用來初始化一個剛建立的 object/instance，舉個例子
```python
class Account:
   def __init__(self, account_holder):
       self.balance = 0
       self.holder = account_holder
```
- `self`:  bound to the newly created Account object
- second parameter, `account_holder`: bound to the argument passed to the class when it is called to be instantiated.
要呼叫這 class 來 instantiate 這物件便用 `a = Account('Kirk')`。

>[!note] 概念四
>Object methods are also defined by a def statement in the suite of a class statement.

這概念說明 class 的 method 可以用 def 來寫，舉個例子
```python
 class Account:
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        def deposit(self, amount):
            self.balance = self.balance + amount
            return self.balance
        def withdraw(self, amount):
            if amount > self.balance:
                return 'Insufficient funds'
            self.balance = self.balance - amount
            return self.balance
```

>[!note] 概念五
>When a method is invoked via dot notation, the object itself plays a dual role. First, it determines what the name withdraw means; withdraw is not a name in the environment, but instead a name that is local to the Account class. Second, it is bound to the first parameter self when the withdraw method is invoked.

這邊說了 class method invoke 時的 dot notation，以這例子舉例: 
```python
spock_account = Account('Spock')
spock_account.withdraw(90)
```
這邊的 dot notation 有兩個作用
1. 確定 `withdraw` 這 method 是屬於 Account 這個 class 的。
2. 把 spock_account 這物件當參數傳進 withdraw 中。
由第二點可以知在 class 中用 def 造出來的 method 第一個參數 `self` 不能省略。

>[!note] 概念六
>To achieve automatic self binding, Python distinguishes between _functions_, which we have been creating since the beginning of the text, and _bound methods_, which couple together a function and the object on which that method will be invoked. A bound method value is already associated with its first argument, the instance on which it was invoked.

因為透過 dot operation 得到的 method 會自動把該物件當第一個參數傳給該 method，跟一般的 function 不同。這時引入一個新概念「bound method」這讓函式和物件在 dot 取值時被綁在一起。我們可以用一般 function (呼叫時要填上 self) 或是 bound method (呼叫時不用填上 self) 來呼叫該物件的函式
```python
Account.deposit(spock_account, 1001)
spock_account.deposit(1000)
```

>[!note] 概念七
>Class attributes are created by assignment statements in the suite of a class statement, outside of any method definition.

介紹 class attribute (也叫 class variables 或是 static variables)，它是用來表示一個所有用該 class 的 object 所共同擁有的變數，舉個例子
```python
class Account:
        interest = 0.02            # A class attribute
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
            
spock_account = Account('Spock')
kirk_account = Account('Kirk')
print(spock_account.interest)    # 0.02
print(kirk_account.interest)     # 0.02
            
Account.interest = 0.04
print(spock_account.interest)    # 0.04
print(kirk_account.interest)     # 0.04
```


>[!note] 概念八
>We could easily have a class attribute and an instance attribute with the same name.

這想說 attribute 可以分為 instance attribute 跟 class attribute，這兩個 attribute 對應到相同的名子，那我們要怎樣區分優先呢? Evaluate a dot expression 的過程如下
```txt
<expression> . <name>
```
1. 先看 expresion 對應到的 object 是啥
2. 看 instance attribute 存不存在，存在就用它的
3. 沒 instance attribute 就用 class attribute
4. 如果是值就直接回傳；如果在 class 找到函式回傳 bound method (自動把 instance 塞進去)，如果是在 instance 找到就直接回傳原函式，不用 bound method
看個例子:
```python
kirk_account.interest = 0.08
print(kirk_account.interest)     # 0.08
print(spock_account.interest)    # 0.04

Account.interest = 0.05
print(kirk_account.interest)     # 0.08
print(spock_account.interest)    # 0.05
```

在 `kirk_account.interest = 0.08` 時就透過 assignment 讓 `kirk_account` 有 instance attribute，所以 `Account.interrest = 0.05` 不會影響 `kirk_account.interest`。`kirk_account.interest = 0.08` 為 instance attribute 所以只會影響 `kirk_account` 這一個 object 不會影響其他 object，所以 `spock_account.interest` 不變，應該說透過 instance 做 assignment 不會影響 class attribute。

>[!question] 問題一
>為啥要設計 instance attribute 不受 class attribute 影響? 設計 attribute 的目的不是為了多個由相同 class 形成的 object 可以有同樣的變數?  

這樣設計是為了提供 attribute 更多的彈性，如果限制所有 instance 不能有自己的 instance attribute 那麼我們便不能提供更細緻化的操作。以銀行客戶為例，一般用戶的利率跟 VIP 不同，而多數用戶的都是一般用戶，且利率是所有用戶的必備變數，所以提供 interest 這變數是合理的，多數用戶都是用同樣利率，可以只設一遍就好，少部分族群特殊處理。

>[!note] 概念九
>Inheritance also has a role in our object metaphor, in addition to being a useful organizational feature. Inheritance is meant to represent _is-a_ relationships between classes, which contrast with _has-a_ relationships.

Inheritance 是種 _is-a_ 的關係。「A is-a B」代表 A 是 B 的一種特化，像是「dog is a animal」、「math is a subject」，所以 A 跟 B 之間是強耦合，通常除非 A 有特別去寫，不然 attribute 和 method 跟 B 一樣。「A has-a B」表示 A 包含 B，像是「car has a wheel」、「campus has a building」，所以 A 跟 B 之間是組合的關係，彼此之間的耦合較低。在 OOP 中我們比較傾向設計出低耦合的程式。具個 python 繼承的例子
```python
class Account:
        """A bank account that has a non-negative balance."""
        interest = 0.02
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        def deposit(self, amount):
            """Increase the account balance by amount and return the new balance."""
            self.balance = self.balance + amount
            return self.balance
        def withdraw(self, amount):
            """Decrease the account balance by amount and return the new balance."""
            if amount > self.balance:
                return 'Insufficient funds'
            self.balance = self.balance - amount
            return self.balance
            
class CheckingAccount(Account):
        """A bank account that charges for withdrawals."""
        withdraw_charge = 1
        interest = 0.01
        def withdraw(self, amount):
            return Account.withdraw(self, amount + self.withdraw_charge)
 
checking = CheckingAccount('Sam') 
```
這個 CheckingAccount 繼承自 Account 這個 base class，CheckingAccount 新增 `withdraw_charge` 這個 class attribute 並更換 `interest` 這 class attribute 跟 `withdraw()` 這函式，這函式實作用到 Account 的 method 這是可行的，應為 calling ancestors 會直接幫你定位到 Account 這個 class，而 Account 有實作 `withdraw()` 這函式 (calling ancestor 的那個 class 如果沒找著，會把他當 base class 往上找)。

>[!note] 概念十
>the act of "looking up" a name in a class tries to find that name in every base class in the inheritance chain for the original object's class.

在繼承的關係中，會從本身這個 class 依照 parent class 的關係一路往上找。

>[!note] 概念十一
>The class of an object stays constant throughout. Even though the deposit method was found in the Account class, deposit is called with self bound to an instance of CheckingAccount, not of Account.

在繼承中會一路往上找 method 或是 attribute，但是 class of an object 一直是自己，不會隨著找尋的過程而改變。會這樣設計是因為，該物件是描述自己，所以所有的變數應該要看自己才對。有這關係後`return Account.withdraw(self, amount + self.withdraw_charge)`中用 `self.withdraw_charge` 是合理的應為如果他還有繼承，例如
```python
class TestAccount(CheckingAccount):
	withdraw_charge = 0.5
```
這樣在呼叫 `withdraw()` 時 `withdraw_charge` 才會帶 0.5，如果 `return Account.withdraw(self, amount + self.withdraw_charge)` 寫成 `return Account.withdraw(self, amount + CheckingAccount.withdraw_charge)` 在那裏也會是對的，但一旦有 subclass 繼承你，那他就算把 attribute 該成 `withdraw_charge = 0.5` 他在呼叫 `withdraw()` 時會用 `CheckingAccount.withdraw_charge` 而不是你自己的 `withdraw_charge` 會這樣是因為 "Attributes that have been overridden are still accessible via class objects"。

>[!note] 概念十二
>Python supports the concept of a subclass inheriting attributes from multiple base classes, a language feature called _multiple inheritance_.

Python 允許一個 subclass 可以繼承自多個 parent clasee，舉個例子
```python
class SavingsAccount(Account):
        deposit_charge = 2
        def deposit(self, amount):
            return Account.deposit(self, amount - self.deposit_charge)
            
class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
        def __init__(self, account_holder):
            self.holder = account_holder
            self.balance = 1           # A free dollar!
```

這會遇到一個問題是「non-ambiguous reference」，這問題是說在同一層的 parent class 可能定義一樣的 method 或 attribute ，此時需要該 method 或是 attribute 的 child class 要看哪一個，準則是 "Python resolves names from left to right, then upwards"。

---
# Section 2.6 (Implementing Classes and Objects)

>[!note] 概念一
>We see that classes and objects can themselves be represented using just functions and dictionaries.

這觀念只想說只要用「函式」跟「字典」便可以做出物件，不需要程式語言特別去寫。物件是一個抽象出來的概念，他只是「資料」跟「行為 (狀態轉移、讀取、算完回傳值不碰資料)」的一個集合，資料可以由函式內的變數提供 (閉包)，行為可以透過字典找對應的 nested function 再交由該函式實作 (dispatch dictionary/ message passing)。

>[!note] 概念二
>We implemented dictionaries with lists, we implemented lists with pairs, and we implemented pairs with functions. As we implement an object system in terms of dictionaries, keep in mind that we could just as well be implementing objects using functions alone.

這觀念想說 dictionary 可以由 list 構成，list  可以由 pair 構成，pair 可以由 function 構成。Dictionary 由 $N$ 個 key-value 的元素購成，其中 key-value 可以由 list ($l^{\prime}$) 組成，而 $N$ 個 key-value 的元素，可以由一個 list 把這些 $l^{\prime}$ 包進去。list 可以用遞迴的 pair 去定義，這邊的 pair 結構是 (現在的值, 指向下一個 pair)，結尾是用指向一個獨立存在的空 list 定義。函式仰賴著 closure 做出 pair，看以下例子
```python
def cons(x, y):
    def dispatch(m):
        if m == 0:
            return x
        elif m == 1:
            return y
    return dispatch

def car(z):
    return z(0)

def cdr(z):
    return z(1)
```
Dictionary 可以單純依靠 function 來實作。

>[!question] 問題一
>如何說明物件導向不是有 class 跟 `.` 操作符的語言的特權

如果可以用函式 (這邊的函式有 closure 的概念)跟字典 (字典也可以用函式搞出來)，做出物件導向程式的功能和觀念，那只要簡簡單單的函式就可以實作出物件導向程式。物件導向程式有以下功能和觀念
1. instance 跟 class 層級的觀念
2. bound method
3. 繼承的想法。child class 如何複寫 parent class、如何向 parent class 查找、instance 屬性不改到 class 屬性。
注: 上述結構（instance/class/繼承）是用函式＋closure＋字典搭出來的，取代了內建 class 語法；而 `.` 操作符則由字典的 message passing（`['get']('name')`）取代。

>[!question] 問題二
>如何做出 「instance 跟 class 層級的概念」跟 「bound method」?

在造 instance 時做出「instance 跟 class 層級的概念」跟 「造 bound method 的機制」，這時才是 instance 跟 class 真正的分界線，所以適合做出「instance 跟 class 層級的概念」，而 「bound method」傳入的 `self` 是 instance 自己，因此在做 instance 的方法內實做 bound method 是合理的。具體做法看如下程式，如何在給定 class (`cls`) 的情況下做出 instance 
```python
def make_instance(cls):
	def get_value(name):
		if name in attributes:
			return attributes[name]
		else:
			value = cls['get'](name)
			return bind_method(value, instance)
	def set_value(name, value):
		attributes[name] = value
	attributes = {}
	instance = {'get': get_value, 'set': set_value}
	return instance		
```

instance 透過 dispatch dictionary (`{'get': get_value, 'set': set_value}`) 的方式讓 instance 可以由 `get` 得到 attribute 跟 `set` 設定 attribute (attributee 包括值跟函式)。在 `get_value()` 中做出從 instance 向上找的關係。BTW, `attributes[name] = value` 這是對 dict 的 mutation 不是 assignment 所以沒有 unbinderror 的問題。
bound method 是由 `bind_method()` 做出來
```python
def bind_method(value, instance):
	if callable(value):
		def method(*args):
			return value(instance, *args)
		return method
	else:
		return value
```
這邊先判斷該 `value` 是不是 callable，如果是的話造一個 `method()` 函式，在該函式內呼叫 `value()`，並預先記住 instance ，等到真正呼叫時才把 instance 塞進去並正式呼叫 (只有在正式呼叫時，才把物件綁定在引數上)。在 python Built-in Function 中對於 Built-in Function 的描述為 "Return `True` if the _object_ argument appears callable, `False` if not." ，所以 `callable(value)` 是用來判斷該 `value` 使否 callable，python reference 的 3.2.8 中對於 callable types 的說明為 "These are the types to which the function call operation (see section [Calls](https://docs.python.org/3/reference/expressions.html#calls)) can be applied:"，所以包括 class 或是帶有 `__call__()` method 的 instance 都是 callable。


>[!question] 問題三
>如何做出繼承的想法?

繼承主要是做 class 的時候才要面對的問題，所以是在 `make_class()` 處理。`make_class()` 的程式如下，給定 `attributes, base_class` 來產出一個 class，`attributes` 表想生成 class 的 attribute，`base_class` 是指產出 class 的 parent class。
```python
def make_class(attributes, base_class=None):
	def get_value(name):
		if name in attributes:
			return attributes[name]
		elif base_class is not None:
			return base_class['get'](name)
	def set_value(name, value):
		attributes[name] = value
	def new(*args):
		return init_instance(cls, *args)
	cls = {'get': get_value, 'set': set_value, 'new': new}
	return cls
```
做 class 時要有跟 instance 相似的 `get_value()`、`set_value()` 函式，此外 `new()` 為新造一個 instance 的函式依舊是必不可少的。`get_value()` 經過 `base_class` 一層層向上查詢，達成 calling ancestor 的想法，這邊不像 instance 要 bound method，單存做查找；動態覆寫 parent class 是由 `set_value` 把 `name, value` 直接寫進本地的 `attributes` 而且 `get_value()` 事先找本地的 attribute，其實在 attributes 傳進這函式時已經先做一次覆寫 parent class。

`new()` 中 `init_instance()` 的函式如下 (該函式負責 instance 的初始化)
```python
def init_instance(cls, *args):
	instance = make_instance(cls)
	init = cls['get']('__init__')
	
	if init:
		init(instance, *args)
	return instance
```
Instance 初始化主要是看該 instance 對應的 class (整條繼承鏈) 有無 `__init__()`，如果有的話按照 `__init__()` 來做初始化，沒的話不做任何初始化，直接回傳一個只有基本結構的空實例。注意這邊的 `init` 沒有 bound method，所以要把 instance 加進引數內。

>[!note] 概念三
>舉個例子

下面這程式是用來造一個 Account class 和使用該 class 的程式
```python
def make_account_class():
	interest = 0.02
	def __init__(self, account_holder):
		self['set']('holder', account_holder)
		self['set']('balance', 0)
	def deposit(self, amount):
		new_balance = self['get']('balance') + amount
		self['set']('balance', new_balance)
		return self['get']('balance')
	def withdraw(self, amount):
		balance = self['get']('balance')
		if amount > balance:
			return 'Insufficient funds'
		self['set']('balance', balance - amount)
		return self['get']('balance')
	return make_class(locals())
	
Account = make_account_class()
kirk_account = Account['new']('Kirk')

print(kirk_account['get']('holder'))        # Kirk
print(kirk_account['get']('deposit')(20))   # 20

kirk_account['set']('interest', 0.04)
print(Account['get']('interest'))           # 0.02
```
這函式用到 `locals` 這函式，該函式回傳一個 dict，這 dict 存 local frame 的 name 跟他們的值，所以這邊的 `locals()` 把 `make_account_class 的 attribute` 和對應的值跟函式存下來，接著傳給 `make_class` 這函式。`make_class`  由 `cls` 這個 dispatch dict 做好 message passing 並提供 `new` 來初始化這個 class。`make_account_class` 內 nested function 的 `self` 都是呼叫該 function 的 instance，別忘了我們前面有座 bound method。

以下程式是在展示繼承的操做
```python
def make_checking_account_class():
	interest = 0.01
	withdraw_fee = 1
	def withdraw(self, amount):
		fee = self['get']('withdraw_fee')
		return Account['get']('withdraw')(self, amount + fee)
	return make_class(locals(), Account)

CheckingAccount = make_checking_account_class()
jack_acct = CheckingAccount['new']('Spock')

print(jack_acct['get']('interest'))      # 0.01
print(jack_acct['get']('deposit')(20))   # 20
```
跟前一個程式類似，基本上只是多傳一個 `base_class` 。一個小細節是 `return Account['get']('withdraw')(self, amount + fee)` 透過類別的 `get` 取出未綁定的 `withdraw` 函式，在手動把 self 傳進去 (instance 才有 bound method，class 只能老老實實的把 instance 自己傳進去)，可使得程式直接套用 parent class 舊有函式。

---
# Section 2.7 (Object Abstraction)

>[!note] 概念一
>A central concept in object abstraction is a _generic function_, which is a function that can accept values of multiple different types.

抽象調物件的型別，讓呼叫者可專注在物件的值上而非型別，函式內部再自己處理。造出一個可自適應多種輸入物件的函式，我們叫他 _generic function_。**關鍵在於降低呼叫者的認知負擔**。






---
# Section 2.8 (Efficiency)





---
# Section 2.9 (Recursive Objects)













