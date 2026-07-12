# Section 4.1 (Introduction)

>[!question] 問題一
>`list` 跟 `range` 這類的資料結構，有啥短版?

在真實世界的資料常常是 infinite 或是 unbounded (unbounded or even infinite size)，像是汽車行進的軌跡、鍵盤的輸入、用戶的行為等，因此資料結構需要有處理未知或無限長輸入的能力，不需要紀錄全部的輸入，但至少要記錄最近 $N$ 筆輸入的能力。 list 受限於記憶體長度的限制無法存入源源不絕進入的資料，如果用覆寫的方式存入最進 $N$ 筆輸入，整個操作變得不直覺，因為難以確定起始點在哪；range 有開頭和結束，需要一開始就輸入終點，無法表達未知結束點或是沒有結束點這件事。


---
# Section 4.2 (Implicit Sequences)

>[!note] 概念一
>_lazy computation_ describes any program that delays the computation of a value until that value is needed.

這個 lazy 的思想蠻常見的，只在需要的時候計算需要的值，避免一次計算所有的值，這可以避免計算不必要的和大量記憶體的空間，因為不見得所有值都要用，同時這也是做出 implicit sequence 的必要手段，如果少了他 unbounded sequence 你根本無窮處理起。這方法有一個缺點是如果一個值被多次使用，lazy 要重覆計算多次。

>[!note] 概念二
>An _iterator_ is an object that provides sequential access to values, one by one. The iterator abstraction has two components
>1. a mechanism for retrieving the next element in the sequence being processed
>2. a mechanism for signaling that the end of the sequence has been reached and no further elements remain

Iterator 的思想類似於 streaming，讓下一個元素依序流進來，一次只處理一個元素，唯一要注意的點是結束點要能被提醒以應對 bounded sequence 的情況。或是抽象一點看，sequence 本身存在的意義，便是為了我們能依序來處理資料，所以需要支持取下一個元素的操作 (`next`)，而有了取下一個元素的方法，我們便可以一直往下走 (iterator 一開始以設在起點，所以 base case 是好的)，問題在於如果遇到 finite sequence 他有一結束點，無法一直走下去，所以需要一個提醒走到底的機制 (StopIteration)。"For any container, such as a list or range, an iterator can be obtained by calling the built-in iter function" 常見操作如下
```python
primes = [2, 3, 5, 7]
iterator = iter(primes)
print(next(iterator))      # 2
print(next(iterator))      # 3
print(next(iterator))      # 5
print(next(iterator))      # 7
next(iterator)     # 拋出 StopIteration 例外
```

>[!note] 概念三
>Two separate iterators can track two different positions in the same sequence. However, two names for the same iterator will share a position, because they share the same value.

直接看例子吧
```python
r = range(3, 13)
s = iter(r)

print(next(s))   # 3
print(next(s))   # 4

t = iter(r)
print(next(t))   # 3
print(next(t))   # 4

u = t
print(next(u))   # 5
print(next(u))   # 6
```

>[!note] 概念四
>Calling `iter` on an iterator will return that iterator, not a copy.

這限制讓我們可以無腦的使用 `iter` 而不用擔心它是 iterator 還是 container。舉個例子
```python
r = range(3, 13)
t = iter(r)

print(next(t))   # 3
print(next(t))   # 4

v = iter(t)
print(next(v))   # 5
print(next(v))   # 6
```

>[!note] 概念四
>Any value that can produce iterators is called an _iterable_ value. In Python, an iterable value is anything that can be passed to the built-in iter function

Python 中 iterable 的型別包括: strings、tuples、set、dictioinary，iterator 等，只要能傳進 built-in iter 函式都算。看個例子
```python
d = {'one': 1, 'two': 2, 'three': 3}
k = iter(d)
print(next(k))   # one
print(next(k))   # two

v = iter(d.values())
print(next(v))   # 1
print(next(v))   # 2
```
要注意 dictionary 新增或刪減時 key 順序被調整，可能造成 iterator 變成 invalid。

>[!note] 概念五
>Several built-in functions take as arguments iterable values and return iterators. These functions are used extensively for lazy sequence processing.

這是想對於 iterable 多一些操作，我們想把一個函式作用在 iterable 的 value 上，並回傳一個 iterator。舉例 `map` 函式， `map` 函式只有在索取該元素時，才把對應的函式，作用在該元素上
```python
def double_and_print(x):
	print(x, '=>', 2*x)
	return 2*x

s = range(3, 7)
doubled = map(double_and_print, s)
print(next(doubled))
# 3 => 6
# 6
print(next(doubled))
# 4 => 8
# 8

print(list(doubled))
# 5 => 10
# 6 => 12
# [10, 12]
```
從輸出可以佐證 `map` 函式是 lazy 的，因為只有在 `next(doubled)` 才輸出 `print(x, '=>', 2*x)` 對應的內容，而且在建構時 (`doubled = map(double_and_print, s)`) 沒有輸出。

>[!note] 概念六
>The for statement in Python operates on iterators. Objects are _iterable_ (an interface) if they have an __iter__ method that returns an _iterator_. 

先看一下用 `for` iterate 整個 iterable object 物件的例子
```python
counts = [1, 2, 3]
for item in counts:
	print(item)
```
Python 中的 `for` statement 是作用在 iterator 上，用 `__iter__()` 把 iterable object 換成 iterator，接著用 `__next__()` 取得下一個元素，直到 StopIteration 這個例外 raise。須注意 iterator 本身也是 iterable 所以可以寫
```python
counts = [1, 2, 3]
for item in iter(counts):
	print(item)
```

>[!important] python 中 for statement 機制
>考慮以下程式
>```python
>for i in range(10):
>	print(i)
>	i += 2
>```
>輸出會是零到九全部輸出一遍，這邊的 `i += 2` 對於整個迴圈不會有任何影響，因為整個 `for` statement 會先用 `__iter__` 把 `range(10)` 這物件變成 iterator 再用 `__next__` 往下一個元素往前，而 `i` 是在每次 `__next__` 後把該元素賦值給 `i`。`i += 2` 不會影響 `__next__` method 因為 `iter(range(10))` 這物件不受 `i += 2` 影響。




>[!question] 問題一
>`iter()` 底層在幹嘛? 有哪些函式也是用相似的技巧?
>

`iter()` 底層是透過呼叫物件的 dunder (double underscore)方法 (`__iter__()` 來運作)，該方法是一類名稱前後有雙底線的特殊方法，讓物件可以呼應 Python 內建的語法和內建函式，這讓我們物件行為的實作更加容易。`iter(x)` 便是呼叫 `x.__iter__()` ，還有 `__getitem__` fallback (物件沒有 `__iter__` 時改用索引取值)等額外邏輯，相似的例子有
- `x + y`  <->  `x.__add__(y)`
- `len(x)`  <->  `x.__len__()`
- `str(x)`  <->  `x.__str__()`
- `next(x)`  <->  `x.__next__()`
- `pl[0]`  <->  `pl.__getitem__(0)`
 這些對應是多型 (polymorphism) 的基礎，由物件本身自行去定義各自的實作，但是有一個統一的接口。

>[!note] 概念七
>A _generator_ is an iterator returned by a special class of function called a _generator function_. Generator functions are distinguished from regular functions in that rather than containing return statements in their body, they use `yield` statement to return elements of a series.

引入 `yield` 的概念。不同於一般的 function，只要在函式 body 內出現 `yield` 就是 generator function，  `yield` statement 回傳 sequence 的元素。會想做這件事，是因為一般的函式 return 後所有的區域變數都消失，這對於需要用現有的值，推下一個值得 sequence 非常不友好，你需要用 nested function 或是 global variable 的方式記錄當前變數的值供下一次使用。`yield` 控制整個函式的執行，讓整個函示定在 `yield` 那一行，直到下一次呼叫 `next` (呼叫 generator function 時函式本體完全不執行,只回傳一個 generator 物件)。舉個例子 
```python
def letters_generator():        # 唯一的一個函式
    current = 'a'
    while current <= 'd':       # 這是迴圈,不是函式
        yield current
        current = chr(ord(current)+1)
        
for letter in letters_generator():
        print(letter)
# a
# b
# c
# d
        
# nested 的例子
def make_letters():
    current = 'a'               # 外層變數當作狀態

    def next_letter():          # ← 這才是 nested function
        nonlocal current
        letter = current
        current = chr(ord(current) + 1)
        return letter

    return next_letter

get = make_letters()
get()   # 'a'
get()   # 'b'
```
`yield` statement 是用來產出序列的下一個元素，所以函式內有 `yield` 自然的在呼叫它拾回傳的 generator 物件自動支援 `__iter__`、`__next__` method，舉個例子
```python
letters = letters_generator()

print(letters.__next__())   # a
print(letters.__next__())   # b
print(letters.__next__())   # c
print(letters.__next__())   # d

letters.__next__()   #  raise StopIteration exception
```
"The generator raises a StopIteration exception whenever its generator function returns." 所以函式結束時會 raise StopIteration exception。

>[!note] 概念八
>An object is iterable if it returns an iterator when its `__iter__` method is invoked. Iterable values represent data collections, and they provide a fixed representation that may produce more than one iterator.

只要一個物件支援 `__iter__` method，那他就是 iterable。用這一個統一的 `__iter__` method 我們便可以操作不同的 iterable 物件，將它們變為 iterator，再用 iterator 的操作 (`__next__` method)。舉個例子
```python
class LetterIter:
    def __init__(self, start='a', end='e'):
        self.next_letter = start
        self.end = end

    def __next__(self):
        if self.next_letter == self.end:
            raise StopIteration
        letter = self.next_letter
        self.next_letter = chr(ord(letter)+1)
        return letter

class Letters:
    def __init__(self, start='a', end='e'):
        self.start = start
        self.end = end
    def __iter__(self):
        return LetterIter(self.start, self.end)

b_to_k = Letters('b', 'k')
first_iterator = b_to_k.__iter__()
print(first_iterator.__next__())    # b
print(next(first_iterator))         # c

second_iterator = iter(b_to_k)
print(second_iterator.__next__())   # b
print(next(second_iterator))        # c
```
可以用 `iter` 或是 `__iter__` 來造出 iterator 再用 `next` 或是 `__next__` 往下一個元素走。

>[!important] class 內變數的聲明
>注意以下程式
>```python
>class LetterIter:
>    def __init__(self, start='a', end='e'):
>        self.next_letter = start
>        self.end = end
>    def __next__(self):
>        if self.next_letter == self.end:
>            raise StopIteration
>        letter = self.next_letter
>        self.next_letter = chr(ord(letter)+1)
>        return letter
>```
>可以注意到 `next_letter` 跟 `end` 這兩個變數沒在 `LetterIter` 這個 class 內賦值，而是在 `__init__` method 內做賦值。這是因為如果在 `LetterIter` 這個 class 內賦值，會變成 class attribute 而不是 instance attribute。可能會有一問題是「變數不用賦值嗎? 函式不會報錯嗎?」，回顧在 chapter 2 中我們用函式跟字典造出 class 時，我們是用字典建立 instance attribute，因此變數不用事先賦值，直接讓字典儲存就好。

>[!note] 概念九
>We may create iterable with `yield`

會想做這件事是因為我們想用 `yield` 儲存現在狀態供以後使用，例如
```python
def all_pairs(s):
   for item1 in s:
       for item2 in s:
           yield (item1, item2)
```
上面這程式會儲存現在函式狀態，讓雙層 `for` 迴圈順利迭代。我們在做一個 iterable 物件時可以用一樣的想法，如下程式
```python
class LettersWithYield:
    def __init__(self, start='a', end='e'):
        self.start = start
        self.end = end
    def __iter__(self):
        next_letter = self.start
        while next_letter < self.end:
            yield next_letter
            next_letter = chr(ord(next_letter) + 1)

letters = LettersWithYield()
print(list(all_pairs(letters))[:5]) 
```
藉由 `yield` 讓 `__iter__` method 回傳一個 generator，這個 generator 存儲函式狀態，特別是當前的 `next_letter` 的值。最後調用 `all_pair` 函式，裡面的 `for` statement 會自動使用這個 generator，因為 `for` 會呼叫 `__next__` 跟 `__iter__` 這些 generator 剛好都具備。兩層的 `for` loop 所以在內、外層的 for statement 造出各自不相干的 generator。 

>[!question] 問題二
>`print(list(all_pairs(letters))[:5])` 這邊造出了幾個 generator? 為啥 `print(list(all_pairs(letters))[:5])` 只傳了一個物件，為何可以造出多個 generator?

在 `all_pairs` 這函式內總共會產生 5 個 generator，外層的 for 迴圈會產生一個 generator，內層的 for 迴圈在外層的 for 迴圈每一次 `__next__` 後都會產生一個 generator。別忘記 `all_pair` 這函式本生有 `yield` 所以也是一個 generator function，所以在呼叫時會回傳一個 generator。因此總共有 6 個 generator。在 `all_pairs(letters)` 中確實只傳入了一個物件 (letters) 但該物件本身有 `__iter__` method 所以在每一次呼叫 `__iter__` method 時都會生成一個 generator。

>[!note] 概念十
>Iterators are mutable: they track the position in some underlying sequence of values as they progress.

Iterator 在 python glossary 的描述是 "An object representing a stream of data." ，在資料流時，需要有一個方法去描述流動的結果 (這邊是離散的，所以可以想成跳到下一筆資料)，而這方法便是 `__next__` method。既然可以用 `__next__` method 去改變 iterator 的狀態，所以它是 mutable。舉個物件中用 `__next__` 的例子
```python
class LetterIter:
    def __init__(self, start='a', end='e'):
        self.next_letter = start
        self.end = end
    def __next__(self):
        if self.next_letter == self.end:
            raise StopIteration
        letter = self.next_letter
        self.next_letter = chr(ord(letter)+1)
        return letter

letter_iter = LetterIter()
print(letter_iter.__next__())   # a
print(letter_iter.__next__())   # b
print(letter_iter.__next__())   # c
```


>[!question] 問題三
>`__iter__` 會造出 iterator ，而 `__iter__` 需要回傳一個帶有 `__next__` 的物件，為啥不把這兩個綁在一起?

關鍵點這是兩個不同的東西，`__iter__` 只管造出 iterator，而 `__next__` 決定下一個元素的選取。你可以在 `__iter__` 時就用現成的帶有 `__next__` 的物件，正如概念八所示，或是自己寫 `__next__` 的邏輯。把 `__iter__` 跟 `__next__` 分開實作，給了我們更多的自由度。迭代的進度(狀態)存在 iterator 身上,而不是存在資料本身，分開實作才能讓同一份資料同時產生多個獨立的 iterator，各自有各自的進度。如果寫在一起會讓所有 iterator 公用同一個進度，看如下例子
```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self        # 關鍵:永遠回傳同一個自己

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

c = Countdown(3)

for n in c:
    print(n)      # 3, 2, 1

for n in c:
    print(n)      # 什麼都不印!
```
除非 `__iter__` 每次都回傳一個新的 iterator (現成的或自己寫的都行，去用該 class 生成一個 instance)，然後把這 `class Countdown` 的 `__next__` 刪掉，統一由 `__iter__` 回傳的那個 iterator class 去處理。結論是把 `__iter__` 跟 `__next__` 分開在不同物件實作，讓所有 `__iter__` method 都可以生成一個新的 iterator，不至於大家都用都一個 iterator。









---
# Section 4.3 (Declarative Programming)






---
# Section 4.4 (Logic Programming)






---
# Section 4.5 (Unification)





---
# Section 4.6 (Distributed Computing)







---
# Section 4.7 (Distributed Data Processing)






---
# Section 4.8 (Parallel Computing)




