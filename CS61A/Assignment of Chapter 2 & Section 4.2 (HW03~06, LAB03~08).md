# HW03

>[!question] 問題一
>以下用來算總共有幾種找零方式的程式，有沒有可取的地方 ?
>```python
> def next_smaller_dollar(bill):
>"""Returns the next smaller bill in order."""
>	if bill == 100:
>		return 50
>	if bill == 50:
>		return 20
>	if bill == 20:
>		return 10
>	elif bill == 10:
>		return 5
>	elif bill == 5:
>		return 1
>
>def count_dollars(total):
>	@lru_cache(maxsize=None)     # 等同於 @cache
>	def count_helper(bill, current_value):
>		if(current_value == 0):
>			return 1
>		elif(current_value < 0):
>			return 0
>		if(bill == 1):
>			return 1
>		return count_helper(bill, current_value - bill) + count_helper(next_smaller_dollar(bill), current_value) 
>	return count_helper(100, total)		
>```

這程式有三個巧思
1. 定義一個順序，讓多個相同的找零組合只計算一次 (EX: (5,1,1),  (1,5,1), (1,1,5)，只能算一次)。
2. 用 nested function 讓 API 接口更乾淨。
3. 用 memoization (`@lru_cache(maxsize=None)`) 使得遞迴呼叫的過程，程式會把函式的引數 (argument) 跟其對應結果記在記憶體中，讓後續調用時不用建立函式 frame 直接拿結果。

先說明一下第一個巧思，在思考多個代表相同意思的組合只能算一次的問題時，主要有兩種常見思路
3. 一次找完再去重
4. 定義一個順序，使的在找的過程中多個代表相同意思的組合只會出現一次
第一個思路類似於資料後處理，適合用在找的過程的算法無法更動，這時只好在原本算法的結果上做處理。第二個思路是基於一個觀察「多個代表相同意思的有序序列常常只是一個組合的不同排列」，所以只要在組合時強制規定一個順序，使其組合唯一，那結果就符合要求。
第二個巧思是來自於一個直覺的想法「只想知道一個金額的找零組合，我在呼叫 API 時應該只需要輸入金額」，所以 `count_dollars()` 只有一個金額的參數，至於在實作時需要知道的目前幣值，則是在呼叫 nested function 中內部的函式時來處理，外層看不到。

>[!question] 問題二
>請依據以下程式回答問題
>```python
>def make_anonymous_factorial():
>	return (lambda f: lambda x: x if x == 1 else x * f(f)(x - 1))(lambda f: lambda x: x if x == 1 else x * f(f)(x - 1))
>	# return (lambda f: f(f))(lambda f: lambda x: 1 if x==1 else x * f(f)(x-1))
>```
>1. `return (lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))(lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))` 跟 `return (lambda f: f(f))(lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))` 在幹麻？他兩設計理念的差別在哪？
>2. `return (lambda f: lambda x: x if x == 1 else x * f(f(x - 1)))(lambda f: lambda x: x if x == 1 else x * f(f(x - 1)))`　這樣寫是對的嗎？
>3. 如果寫 `return (lambda f: lambda x: 1 if x == 1 else x * f(x - 1))(lambda f: lambda x: 1 if x == 1 else x * f(x - 1))` 錯在哪？
>4. 解釋一下 U Combinator 跟 Y Combinator 的想法？

這兩程式都是在計算給定 `x` 的階層值，特點都是在不聲明函式名稱的情況下實現遞迴呼叫。先說明以下程式
```python
return (lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))(lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))
```
要理解這函式，先做一個 name binding
```python
g1 = lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1)
```
這個函式 `g1` 可以作如下翻譯
```python
def g1(f):
    def h(x):                      
        if x == 1:
            return 1
        else:
            return x * f(f)(x - 1)  
    return h
```
`g1` 是一個吃 `f` 的函式並吐出 `h` 的函式，`h` 本身是一個吃 `x` 的函式，該函式依靠 `f` 做遞迴處理。接下來可以令
```python
g2 = lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1)
```
所以
```python
return (lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))(lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))
```
可以寫成
```python
return g1(g2)
```
剖析 `g1(g2)(3)` 之前又有一個概念:「C/C++、Rust、Python 是嚴格語言 (strict evaluation)」，嚴格語言代表一個函式被呼叫前只做兩件事
1. 把引數 (argument) 的值算出來。
2. 把引數的值綁到參數 (parameter) 上並運行函數本體。
而目前 `g1` 的引數 `g2` 的值就是 lambda 函式本身 (其本身就代表一個運算過程)，所以直接把 `g2` 灌進 `g1` 內，即 `g1` 的 `f` 被 `g2` 替代。目前整個函示變為
```python
def g1(f):
    def h(x):                      
        if x == 1:
            return 1
        else:
            return x * g2(g2)(x - 1)  
    return h
```
接下來把 `x=3` 帶入，求 `3 * g2(g2)(2)` ，現在 `g2` 的 `f` 便是他自己，如此完成自我的呼叫，並把 `x=2` 帶入並自我呼叫一下，再來變到達終止處 `x=1`。整個過程如下
```text
g1(g2)(3)
= 3 * g2(g2)(2)
= 3 * 2 * g2(g2)(1)
= 3 * 2 * 1
```
這邊 `g2` 的自我呼叫，詳情如下 (`f` 為 `g2`)
```python
def g2(f):
    def h(x):                      
        if x == 1:
            return 1
        else:
            return x * f(f)(x - 1)  # return x * g2(g2)(x-1)
    return h
```
可以看到終止條件是 `x == 1`，而在遞迴時是靠自己呼叫自己達成。跟一般用函式名字做遞歸不同，這邊靠的是 `g2` 吃進自己 (`g2`) 並用 `f(f)` 來使得下一層呼叫存在。
接下來考慮以下程式
```python
return (lambda f: f(f))(lambda f: lambda x: 1 if x == 1 else x * f(f)(x - 1))
```
依舊做一下 name binding
```python
g1 = lambda f: f(f)
```
`g1` 可以寫成
```python
def g1(f):
	return f(f)
```
依舊只要把前面的 `g2` 傳入這邊的 `f` 問題變結束。

>[!note] 前面說明的兩行程式的思想與差別
>這題想表達得是，我們可以不靠函式名字來呼叫函式來達到遞迴的過程。為了達到遞迴，我們必須要出現一個在不符合終止條件時，依舊能繼續創造一個新的函式 (該函式和原函式一樣，跟一般遞迴呼叫函示名字一樣)，來表達新的轉移狀態。前述的創造一個新函式便是用 `f(f)` 來做到，而你要用哪個遞回函式變是指定 `f` 為你要的函式，這函式變是用第二個 lambda 來傳遞。所以第一個 lambda 變是提供第一個開始互相自我應用的地方，而接下來的遞迴函式變由第二個 lambda 提供。
>兩個寫法其實都一樣，只是第二個寫法把第一個開始互相引用寫得更乾淨。

以下寫法是有問題的
```python
return (lambda f: lambda x: x if x == 1 else x * f(f(x - 1)))(lambda f: lambda x: x if x == 1 else x * f(f(x - 1)))
```
第一個是程式呼叫過程會導致錯誤。先做一下 name binding
```
g = lambda f: lambda x: x if x == 1 else x * f(f(x - 1))
```
取 `x=2` 會有以下結果
```python
g(g)(2)
= 2 * g(g(1))
```
錯誤是 `2` 是 int 而 `g(g(1))` 是一個函式物件 (`g(g(1))` 是函式,要再 apply 一個引數才可能變成數值)，相乘會 typeerror。第二個問題是 `f(f(x - 1))` 內的 `f(x - 1)` 把要傳遞的函式破壞掉，提前把引數放入，而不是把函式放入，不是遞迴所想的是。


---
# LAB03

>[!question] 問題一
>依下面程式回答問題
>```python
>s = [7//3, 5, [4, 0, 1], 2]
>print(s[-1])
>```
>1. 輸出是啥 ?
>2. 結果跟 C/C++ 一樣嗎 ? 為何 ?

這會輸出在 `s` 的最後一個元素。
C/C++ 跟 Python 在設計此類資料結構的思路本就不同。C/C++ 索引時陣列退化成指標,而那根指標不帶長度，所以 (`a` 是一個陣列) `a[1]` 會索引到 `a` 後第一個物件，`a[-1]` 會是 undefined behaviour；Python 的 list 本身是一個物件，當你綁定一個名字到該物件時，可以透過名字操作該物件，而這序列物件本身就包含長度訊息，所以在規格書可以直接說

>1. If _i_ or _j_ is negative, the index is relative to the end of sequence _s_: `len(s) + i` or `len(s) + j` is substituted. But note that `-0` is still `0`.

整體思路是用負號代表從尾巴往回走。

>[!question] 問題二
>以下程式問題在哪
>```python
>def double_eights(n):
>"""Returns whether or not n has two digits in row that are the number 8."""
>	def recursive_find(is_previous_satisfy):
>		if n == 0:
>			return False
>		is_current_satisfy = n % 10 == 8
>		if is_previous_satisfy and is_current_satisfy:
>			return True
>			
>		n //= 10
>		return recursive_find(n % 10 == 8)
>return recursive_find(False)
>```

問題出在 `n //= 10`，當變數賦值時，該變數就被視為 local frame 的變數，而不會透過 closure 去看外層的變數。正如規格書 4.2.2 說的這句話 "If a name binding operation occurs anywhere within a code block, all uses of the name within the block are treated as references to the current block. This can lead to errors when a name is used within a block before it is bound."。因此 `if n == 0` 在 `n` 尚未綁定前讀取它,會觸發 `UnboundLocalError`。修法是讓 `recursive_find` 把 `n` 當成自己的參數(`def recursive_find(n, is_previous_satisfy)`)。

---
# HW04

>[!question] 問題一
>以下程式問題出在哪裡
>```python
>def deep_map(f, s):
>"""Replace all non-list elements x with f(x) in the nested list s."""
>	for item in s:
>		if type(item) == type([]):
>			deep_map(f, item)
>		else:
>			item = f(item)
>	return
>```

有一個觀念是「賦值時只是把名稱 bind 物件上」。我們確實可以透過名稱對物件操作，但賦值 (`=`) 時只是把名稱綁到其他物件上，所以這邊的 `item = f(item)` 只是把 item 從新綁到 `f(item)` 回傳的物件。如果要改動 `s` 內的元素，得在賦值左邊用 `s[index] = f(item)`，叫 list 就地把第 index 格改綁到 `f(item)`，這一步可以在 python refernce 7.2 (Assignment Statement) 中看到 "If the primary is a mutable sequence object (such as a list), the subscript must yield an integer. ... the sequence is asked to assign the assigned object to its item with that index"。

>[!question] 問題二
>在 python 中整數沒有限制長度，那如何表示數字的最小值呢?

可以直接用 `float('-inf')` 表達最小值。python 中的 float 用 IEEE754 的 64 位元版 (python standard library 的 built-in types 說 "Floating-point numbers are usually implemented using double in C")，所以可以表達負無窮 (位元為 `0xFFF0000000000000`)，而負無窮比所有整數都小，所以可以做為數字值的下界。前述有一問題「int 跟 float 比較，如果把 int 轉乘 float ，大數可能 overflow ，因為 int 沒限制長度；另一方面，如果數字超出 $2^{53}$ 可能因四捨五入導致精度不對而估錯。」，所以在 python standard library 的 built-in types 有一說明 "A comparison between numbers of different types behaves as though the exact values of those numbers were being compared." 避免這一問題。


---
# LAB04

---
# HW05

>[!question] 問題一
>請根據以下程式回答問題
>```python
>def stair_ways(n):
>	"""
>	Yield all the ways to climb a set of n stairs taking
>	1 or 2 steps at a time.
>	"""
>	if n < 0:
>		return
>	if n == 0:
>		yield []
>		
>	for step in [1, 2]:
>		for solution in stair_ways(n - step):
>			yield [step] + solution
>```
>1. 詳細的解釋一下這程式在幹嘛?
>2. `yield` statement 時函式不是被停止了，為何他會向遞迴一樣向 caller 回傳值，該 caller 還可以向它的 caller 回傳?
>3. 跟一般的遞迴差在哪?
>4. 將 `yield []` 改成 `return []` 會怎樣?

這是個遞迴的函式，用來產生登上樓梯的方法。由 `yield` statement 可以知這程式是個 generator function，使用 `next` function 來遍歷所有可能的方法。整個程式關鍵點在於 `for solution in stair_ways(n - step):` 這一行，因為 `stair_ways` 這函式是個 generator function，所以呼叫 `stair_ways` 這函式時會回傳 generator，再用 `next` 遍歷所有可能。關於 `yield` statement 暫停程式的問題，確實在 `yield` 那行時該函式暫停，但 caller 跟 callee 之間是用 `next` function 取值，callee 用 `yield` 回傳值給 caller 後，callee 確實暫停了，但 caller 依舊會用自己的 `yield` 回傳，一路傳到遞迴的一開始，所以對整個遞迴的輸出沒問題。整體來說跟一般的遞迴差不多，重點在於求值的時間，這程式是 lazy 求值的只有在呼叫 `next` 時算值 (這邊的回傳是用 `next` function 一次蹦一個)，一般的遞迴是 eager 求值的，一次產生所有登上樓梯的方式。當一個程式因為 `yield` 變為 generator 時，return statement 只會~~呼叫~~ raise `StopIteration` ，caller 不會拿到 return 的那個物件，所以如果這邊把 `yield []` 改成 `return []` ，只會~~呼叫~~ raise `StopIteration([])` 而 `[]` 儲存在 `StopIteration.value` ，不會回傳 `[]` 給 caller。把 `yield []` 改成 `return []` 有一錯誤是 `stair_ways(n)` 一定沒有輸出，因為 `yield [step] + solution` 需要 `stair_ways` 要有輸出，而 `n == 0` 這 base case 沒輸出，所以整體沒有任何輸出。

---
# LAB05



>[!question] 問題七
>以下程式在底層在幹嘛
>```python
>map_iter = map(lambda x : x + 10, range(5))
>print(list(map_iter))
>```




>[!question] 問題八
>```python
>def insert_items(s, before, after):
>"""Insert after into s after each occurrence of before and then return s."""
>	inputPosition = [i for i in range(len(s)) if s[i] == before]
>	for pos in reversed(inputPosition): 
>		s.insert(pos + 1, after)
>	return s
>```
>1. `for` statement 的 `reversed` function 只會被呼叫一次嗎?
>2. 為啥要用 `reverse` function

在這行程式 `for pos in reversed(inputPosition):` 的 `reversed` function 會輸出一個 list_reverseiterator，這一個 list_reverseiterator 會從後向前迭代整個 list。而 `for` statement 在收到這個 list_reverseiterator 會用 `__iter__` method 把它變 iterator (iter() 一個 iterator 依舊輸出一個 iterator)，再用 `__next__` method 往前挺進。這邊會用 `reversed` function 是因為要用 `insert` method 往 `s` 內加東西，如果從後面的 index 向前面的 index 插入，插入時並不候影響到之後要插入的位置，因為插入時只會影響該 index 之後的元素，而在後 index 向前 index 插入的設定下，在插入 index 為 i 的元素時，i 後面的元素已經處理完就算往後移也沒差。

>[!question] 問題九
>請根據以下程式回答問題
>```python
>def group_by(s, fn):
>	grouped = {}
>	for item in s:
>		key = fn(item)
>		if key in grouped:
>			# grouped[key] = grouped[key].append(item)
>			grouped[key].append(item)
>		else:
>			# grouped[key] = list(item)
>			grouped[key] = [item]
>	return grouped
>```
>1. 為何不能用 `grouped[key] = grouped[key].append(item)` 取代 `grouped[key].append(item)`?
>2. 為何不能用 `grouped[key] = list(item)` 取代 `grouped[key] = [item]`?

`grouped[key].append(item)` 會回傳 `None` 所以 `grouped[key] = grouped[key].append(item)` 會把 `grouped[key]` 賦值為 `None`。在 `grouped[key].append(item)` 會對於 `grouped[key]` 的 list in-place 修改，所以直接寫 `grouped[key].append(item)` 就行。想用 `list(item)` 該 `item` 必需是 iterable，而這邊的 `item` 不是 iterable，所以直接用是不行的。 

>[!note] 返回 None 的設計理念
>看 python 設計者 Guido van Rossum 的 mail [sort() return value](https://mail.python.org/pipermail/python-dev/2003-October/038855.html)，這篇 mail 說明了作者的想法「對於有回傳新值的函式，他是支持 chaining 的想法，但對於 in-place 修改的物件，他是不支持 chainning，因為如果支持 chainning 的話，你會不知道你現在是 in-place 修改函式在新物件上修改，所以  Guido van Rossum 把 in-place 跟回傳新值的函式分開」。舉些例子，考慮以下程式
>```python
># 寫法一
>x.compress().chop(y).sort(z)
>
># 寫法二
>x.compress()
>x.chop(y)
>x.sort(z)
>```
>寫法二顯然更有可讀性，一看就知道是對 `x` 做了三個 in-place 的修改 (`compress`、`chop(y)`、`sort(z)`)，而不會去往造一個新物件的方向想。再看一個程式
>```python
> y = x.rstrip("\n").split(":").lower()
>```
>Guido van Rossum 對於以上程式是支持的，因為 `rstrip()`、`split()`、`lower()` 這三個函式都會回傳新值。常見的 in-place 方法有，list: `a.append(x)`、`a.extend(iter)`、`a.insert(i, x)`、`a.remove(x)`、`a.clear()`、`a.sort()`、`a.reverse()` 以上方法都是 in-place 方法回傳都是 `None` (例外: `a.pop()`、`a.pop(0)` 移除並回傳元素)；dict: `d.update(other)`、`d.clear()` (`d.pop(k)` 移除並回傳對應的值)；set: `s.add(x)`、`s.remove(x)`、`s.discard(x)`、`s.update(other)`、`s.clear()` (`s.pop()` 移除並回傳元素)。在 list 中回傳新物件的寫法: `sorted(a)`、`reversed(a)` (`reversed(a)` 是回傳一個迭代器 list_reverseiterator)。注意 `a += [5]` 是 in-place 修改，而 `a = a + [5]` 是建新物件，這點在 [PEP 203 Augmented Assignments](https://peps.python.org/pep-0203/#rationale) 中有說明 `x += y` 是先調用 `x.__iadd__(y)` 不行的話再用 `x.__add__(y)`，這好處在於 "The new in-place operations are especially useful to matrix calculation and other applications that require large objects. In order to efficiently deal with the available program memory, such packages cannot blindly use the current binary operations."

>[!note] `list()` 的觀念
>在 python built-in type 中對於 `list()` 的說明是 `class list(iterable=(), /)` 所以 `list()` 是個 class。"The constructor builds a list whose items are the same and in the same order as _iterable_’s items. _iterable_ may be either a sequence, a container that supports iteration, or an iterator object." list 可以讓所有 iterable 物件變成 list，`list('abc')` 會回傳 `['a', 'b', 'c']`，`list((1, 2, 3))` 會回傳 `[1, 2, 3]`。list 有一個常見觀念如下 (`list(a)` 建立新的 list 物件，但元素共用參照；改 b 的結構不影響 a，透過 b 修改共用元素的內部則 a 也會變，shallow copy)
>```python
>a = [[1, 2], [3, 4]]
>b = list(a)
>
>b.append([5, 6])
>print(a)     # [[1, 2], [3, 4]]
>print(b)     # [[1, 2], [3, 4], [5, 6]]
>
>b[0].append(99)
>print(a)     # [[1, 2, 99], [3, 4]]
>print(b)     # [[1, 2, 99], [3, 4], [5, 6]]
>```


---
# HW06

>[!Question] 問題一
>考慮以下程式並試回答問題
>```python
>def deep_map_mut(func, s):
>"""Mutates a deep link s by replacing each item found with the
>result of calling func on the item. Does NOT create new Links (so
>no use of Link's constructor).
>Does not return the modified Link object.
>>>> link1 = Link(3, Link(Link(4), Link(5, Link(6))))
>>>> print(link1)
><3 <4> 5 6>"""
>	if s is Link.empty:
>		return
>	if isinstance(s.first, Link):
>		deep_map_mut(func, s.first)
>	else:
>		s.first = func(s.first)
>		
>	deep_map_mut(func, s.rest)
>	return
>```
>1. 這邊的 `isinstance(s.first, Link)` 在解決甚麼問題 ?
>2. 為什麼要用遞迴處理這程式 ?

用 `isinstance(s.first, Link)` 是因為 `s` 是一個 deep link 無法保證 `s.first` 是一個值，所以需要 `isinstance(s.first, Link)` 來處理 `s.first` 是一個 Link class 的情況。在 deep link 中我們無法知道要經過多少個 `s.first` 才能達到 `s.first` 是值的情況，同時 Link class 是用遞迴定義，所以用遞迴是一個很自然的選擇。

---
# LAB06

>[!question]
>請根據以下程式回答問題
>```python
>class Mint:
>"""A mint creates coins by stamping on years.  The update method sets the mint's stamp to Mint.present_year."""
>	present_year = 2024
>	def __init__(self):
>		self.update()
>	def create(self, coin):
>		"*** YOUR CODE HERE ***"
>		# self.coin = coin(self.year)
>		# return self.coin
>		return coin(self.year)
>	def update(self):
>		"*** YOUR CODE HERE ***"
>		# self.stamp = present_year
>		# self.year = self.present_year
>		self.year = Mint.present_year
>class Coin:
>	cents = None # will be provided by subclasses, but not by Coin itself
>	def __init__(self, year):
>		self.year = year
>	def worth(self):
>		"*** YOUR CODE HERE ***"
>		return self.cents + max(0, (Mint.present_year - self.year - 50))
>class Nickel(Coin):
>	cents = 5
>class Dime(Coin):
>	cents = 10
>```
>1. class 之間的分割邏輯是啥?
>2. 這邊為啥在 `return self.cents + max(0, (Mint.present_year - self.year - 50))` 用 `Mint.present_year`?
>3. 為何在 Mint class 的 `__init__` method 中呼叫 update method 要用 `self.update()` 不直接用 `update()` 就好

Mint 是鑄幣廠負責產出 Coin，所以有 create method 負責產出。Coin 的價值包含兩個指標，一個是本身價值由 `cents` 這 class attribute 表達，另一個歷史價值 (放得越久價格可能更高) 由鑄幣廠現在年份減去 Coin 鑄造年份再減去 50 的值體現，這值跟 0 取 max 即為歷史價值。需要注意這邊 Mint class 跟 Coin class 之間的關係是 create-a 而不是 is-a 或是 has-a，會這樣設計是因為 Coin 不是 Mint 的一個特化，Coin 也不是 Mint 組成的一部分，而是 Mint 造出來的一個 instance。Nickle 跟 Dime 才是 is-a 的關係，因為他倆是 Coin 的一個特例。這邊在 `return self.cents + max(0, (Mint.present_year - self.year - 50))` 用 `Mint.present_year` 是因為歷史價值要對其 Mint 的時間，所以直接用 `Mint.present_year` 把 Mint attribute 取出來是最合理的 (你直接用 `self.present_year` 也不行，因為 Coin 沒有繼承自 Mint)。對於第三題，在 python charpter 4 (Execution model) 中的描述已經回答 "Names in class scope are not accessible. Names are resolved in the innermost enclosing function scope. If a class definition occurs in a chain of nested scopes, the resolution process skips class definitions. This rule prevents odd interactions between class attributes and local variable access. If a name binding operation occurs in a class definition, it creates an attribute on the resulting class object. To access this variable in a method, or in a function nested within a method, an attribute reference must be used, either via self or via the class name." 重點在於 "If a class definition ..., the resolution process skips class definitions" ，method 裡的名字解析會跳過外層 class 的 scope，所以 class body 裡定義的名字（不管是 attribute 還是 method）在 method 內都不能用裸名存取，必須透過 attribute reference（`self.update()` 或 `Mint.update(self)`），這樣可以避免ㄧ些奇怪的狀態，如下面的例子
```python
class Counter:
    count = 0
    def increment(self):
        count = count + 1   # 這行該是什麼意思？ count 要看 local 的還是 class attribute 的

class C:
    x = 1
    def f(self):
        return x   # 假設這能看到 class scope，回傳 1？

del C.x            # 執行期把 class attribute 刪掉
C().f()            # 現在這個 x 是什麼？NameError？還是 fallback 到 global？

c = C()
c.x = 99
c.f()   # 裸名 x 該回傳 1（class）還是 99（instance）？
```
如果允許 method 內的裸名解析到 class scope，同一個名字會同時受「scope 鏈查找」和「attribute 查找」兩套規則影響（後者還是動態的，會被 `del`、instance 遮蔽改變），語意無法確定。強制用 `self` 或 class name 存取，讓兩套查找系統各自獨立，這就是 "prevents odd interactions" 的意思。

---
# LAB07

>[!Question] 問題一
>```python
>class FreeChecking(Account):
>	withdraw_fee = 1
>	free_withdrawals = 2
>	"*** YOUR CODE HERE ***"
>	def withdraw(self, amount):
>		fee = 0
>		if self.free_withdrawals != 0:
>			self.free_withdrawals -= 1
>		else:
>			fee = self.withdraw_fee
>		return super().withdraw(amount + fee)
>```
>1. 為何這邊用 `super()` ?
>2. 為何不用 `Account.withdraw(self, amount + fee)` 而用 `super().withdraw(amount + fee)` ?

這邊的 `super()` 是想直接用父類別的函式來處理，這符合繼承的想法「能調用父類別的就調用」。直接使用 `super()` 可以讓直譯器依照 MRO (Method Resolution Order)，去找該調用哪個父類別，還有一個好處是 `super()` 會用 bound method 讓傳參時少 `self` 這參數。另一方面，如果使用 `Account.withdraw(self, amount + fee)` 相比用 `super()` 要多傳一個 `self` 這參數，還有一個問題是如果 `FreeChecking(Account)` 中的 `Account` 被替換成其他父類別，那 `Account.withdraw(self, amount + fee)` 要換成替換的那個父類別。

---
# LAB08