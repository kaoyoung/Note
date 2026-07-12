# HW03




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

這是個遞迴的函式，用來產生登上樓梯的方法。由 `yield` statement 可以知這程式是個 generator function，使用 `next` function 來遍歷所有可能的方法。整個程式關鍵點在於 `for solution in stair_ways(n - step):` 這一行，因為 `stair_way` 這函式是個 generator function，所以呼叫 `stair_way` 這函式時會回傳 generator，再用 `next` 遍歷所有可能。關於 `yield` statement 停止程式的問題，確實在 `yield` 那行時該函式停止，但 caller 跟 callee 之間是用 `next` function 取值，callee 用 `yield` 回傳值給 caller 後，callee 確實停止了，但 caller 依舊會用自己的 `yield` 回傳，一路傳到遞迴的一開始，所以對整個遞迴的輸出沒問題。整體來說跟一般的遞迴差不多，重點在於求值的時間，這程式是 lazy 求值的只有在呼叫 `next` 時算值 (這邊的回傳是用 `next` function 一次蹦一個)，一般的遞迴是 eager 求值的，一次產生所有登上樓梯的方式。

---
# LAB05

>[!question] 問題一
>根據以下程式回答問題
>```python
>s = [6, 7, 8]
>print(s.append(6))
>```
>1. 輸出是啥?
>2. 為啥要這樣設計? 時間複雜度是啥?






>[!question] 問題二
>根據以下程式回答問題
>```python
>s = [6, 7, 8, 6]
>print(s.insert(0, 9))
>print(s)
>```
>1. 輸出是啥
>2. 為啥要這樣設計? 時間複雜度是啥?






>[!question] 問題三
>根據以下程式回答問題
>```python
>s = [9, 6, 7, 8, 6]
>x = s.pop(1)
>print(x)
>print(s)
>```
>1. 輸出是啥?





>[!question] 問題四
>根據以下程式回答問題
>```python
>s = [9, 6, 7, 8, 6]
>x = s.pop(1)
>print(s.remove(x))
>print(s)
>```
>1. 輸出是啥? 時間複雜度是多少?







>[!question] 問題五
>根據以下程式回答問題
>```python
>a = [9, 7, 8]
>print(a.pop())
>```
>2. 輸出是啥?
>3. 為啥這樣設計?







>[!question] 問題六
>根據以下程式回答問題
>```python
>s = [3, 4, 5]
>s.extend([s.append(9), s.append(10)])
>print(s)
>```
>4. 輸出是啥?
>5. 為啥這樣設計?





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

Mint 是鑄幣廠負責產出 Coin，所以有 create method 負責產出。Coin 的價值包含兩個指標，一個是本身價值由 `cents` 這 class attribute 表達，另一個歷史價值 (放得越久價格可能更高) 由鑄幣廠現在年份減去 Coin 鑄造年份再減去 50 的值體現，這值跟 0 取 max 即為歷史價值。需要注意這邊 Mint class 跟 Coin class 之間的關係是 create-a 而不是 is-a 或是 has-a，會這樣設計是因為 Coin 不是 Mint 的一個特化，Coin 也不是 Mint 組成的一部分，而是 Mint 造出來的一個 instance。Nickle 跟 Dime 才是 is-a 的關係，因為他倆是 Coin 的一個特例。這邊在 `return self.cents + max(0, (Mint.present_year - self.year - 50))` 用 `Mint.present_year` 是因為歷史價值要對其 Mint 的時間，所以直接用 `Mint.present_year` 把 Mint attribute 取出來是最合理的 (你直接用 `self.present_year` 也不行，因為 Coin 沒有繼承自 Mint)。

