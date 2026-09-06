# HW01

>[!question] 問題一
>為啥 python 的函式沒有返回值得型別？

Python 的設計型別是動態型別 (dynamic typing)，型別綁在值（物件）上而非變數上 (物件的型別在它被建立時就固定了;不確定的是某個名字/變數在執行時會綁到哪個物件,所以要執行到那一行,才知道這個名字「當下是什麼型別」)，在執行期才會知道型別，所以還傳值在回傳才知道型別。語言不要求、也不強制你標註型別。不過 Python 提供了 type hints（型別提示），參數和回傳值都可以寫，例如 `def f(x: int) -> int:`，但直譯器 (intepreter) 執行時不會主動檢查這些註解，它只會把這些型別提示存起來，要做檢查得用 mypy、pyright 等靜態檢查工具。同理，函式的 paramter 也沒有寫型別。
注: 靜態型別 (Static Typing) 的語言會在編譯期 (Compile-time) 就檢查型別是否正確。雖然傳統上這類語言常需要手動標註型別（如 Java, C），但現代許多靜態語言也有提供型別推斷（Type Inference）的功能，由編譯器自動判斷。

>[!question] 問題二
>這程式在幹嘛
>```python=
>from operator import add, sub
>
>def a_plus_abs_b(a, b):
>	"""Return a+abs(b), but without calling abs.
>	
>      >>>a_plus_abs_b(2, 3)
>	5
>      >>> a_plus_abs_b(2, -3)
>	5
>	"""
>	if b < 0:
>		f = sub
>	else:
>		f = add
>	return f(a, b)
>```

這程式可以看出 python 中「一級函數」的概念，函數就是物件,而物件本來就能被指派給變數。一級函數告訴我們，函數可以被指派給變數(綁定到名稱)、當作參數傳給函式、當函式的回傳值和封裝在一個資料結構內。

>[!question] 問題三
>`％` 在 C/C++ 和 Python 有甚麼差別 ? 請詳述背後的設計想法

現在的 `/, %` 都是代表程式語言的符號，而不是數學上的符號 (除非特別強調)。以下等式是所有程式都必須遵守的規則$$ (a / b) * b + (a \% b) = a \tag{1} $$小心 python 的 `/` 跟 `//` 不一樣。從上面的等式可以知道 `a / b` 的值決定了 `a % b` 是多少，對於 `a / b` 有兩個派別:
1. 硬體直覺派 (Truncated Division): 利用硬體截斷的特性，直接定義 `a / b`。弟子有 C/C++、Rust、Jave。
2. 數學直覺派 (Floored Division): 在 `b > 0` 的情況下，希望餘數 (`a % b`) 的範圍是 `[0, b)` 。弟子有 Python。
想理解硬體截斷的特性，必須先看硬體如何處理 `a / b`。在 `a / b` 時，可以理解為硬體會先對 `a, b` 取絕對值，然後作整數除法的操作，之後自然地得到不考慮正負號的商 (`a / b`) 和餘數 (`a % b`) ，最後商的正負號對 `a,b` 做一次 xor 便結束。這設定下有一顯然事實是「不考慮正負號的商 (`a / b` ) 的結果是對數學上不考慮正負號的 `a / b` 真正結果取下高斯」，使得在 $a < 0, b > 0$ 時，為了滿足式 (1) 的限制，`a % b`必非正 ，而在 $a > 0, b < 0$ 時`a % b`必非負。
數學直覺派希望在 `b > 0` 的情況下，餘數 (`a % b`) 的範圍是 `[0, b)`，所以在 $a < 0, b > 0$ 時必須讓 `a / b` 向負無窮取整，這表示 `a,b` 有一為負時 `a / b` 向負無窮取整，導致 $a > 0, b < 0$ 時`a % b`必非正。在 [Why Python's Integer Division Floors](https://python-history.blogspot.com/2010/08/why-pythons-integer-division-floors.html)舉了個 python 的例子說明這方法的好處，另外在 cycle 或是其他 index 需要為正的資料結構時有益處。
例子: 
- 硬體直覺派: `-7 / 3 == -2`、`-7 % 3 == -1`；`7 / -3 == -2`、`7 % -3 == 1` ；`-7 / -3 == 2`、`-7 % -3 == -1`
-  數學直覺派: `-7 / 3 == -3`、`-7 % 3 == 2`；`7 / -3 == -3`、`7 % -3 == -2` ；`-7 / -3 == 2`、`-7 % -3 == -1`
注: 在早期（C89 / C++98 時代），當有負數參與除法時，商是向上還是向下取整，是**交由編譯器（Implementation-defined）決定**的。直到 **C99** 與 **C++11** 標準頒布後，才強制規定必須「向零取整（Truncated Division）」。

---
# LAB01

>[!question] 問題一
>在 python3 中為何需要定義 `//` ? 不能由 `a // b` 的 `a,b` 型別自動推理嗎 ?

在 python 中為了使 `/` 符合數學的直覺，而讓 `/` 出來的值為浮點數，如此避免了在 C/C++ 中 `5/2` 為 2 不符合數學直覺的問題。 在`//` 的語意永遠是 floor division，結果的型別則依 `a, b` 的型別推理——皆為 int 則回傳 int，含 float 則回傳 float。須小心 `//` 無法對標 C/C++ 的 `/`，前者是 floored division 後者是 truncated division，而且 C/C++ 的 `/` 會依照`a,b` 型別自動更換語意和運算結果，`5.0 / 2` 會輸出 `5.0 / 2 = 2.500000` 而不是 python 在 `5.0 // 2` 的 `2.0`。
```python
5 // 2 # 輸出為 2
5.0 // 2 # 輸出為 2.0
```
C/C++ 跟 Python 對於 `/` 的想法不同，根源在於 C/C++ 的型別是 static type 而 Python 的型別是 dynamic，所以 C/C++ 可以依照運算元的型別自動推測運算子的語意，而 Python3 選擇禁止做到這件事 (運算元的型別在執行期才知道，無法在程式撰寫時知道，所以讓運算子的語意隨著運算元走是危險的)，因此需把運算子和運算元分開處理，運算元決定運算結果的型別，運算子的語意是固定的，確保不會隨執行期輸入的型別不同而浮動。具體說明可以看這一篇 [PEP 238 – Changing the Division Operator](https://peps.python.org/pep-0238/)。把運算子和運算元分開，頗有代數風範。

>[!question] 問題二
>請根據以下程式回答問題
>```python
>def welcome():
>     print('Go')
>     return 'hello'
> def cal():
>     print('Bears')
>     return 'world'
> print(welcome(), cal())
>```
>1. 輸出是啥?
>2. python 對於函式引數 (argument) 呼叫的想法和 C 跟 Rust 有何異同?

理解第一小題的輸出要有兩概念。第一個是 python 引數呼叫的順序是由左到右的；第二個是 `print()` 中用 `,` 分割的字串，默認是用空格連接還有 `print()` 默認結尾符是 `\n`。有這兩觀念後，不難推出 `print(welcome(), cal())` 的輸出是
```txt
Go
Bears
hello world
```
關於第一點的說明在 python reference 6.16 中 "Python evaluates expressions from left to right. Notice that while evaluating an assignment, the right-hand side is evaluated before the left-hand side."。第二點是看 python standard library 中 print 函式簽名便可知
```python
print(_*objects_, _sep=' '_, _end='\n'_, _file=None_, _flush=False_)
```

在 C 中對於引數順序的描述在 C99 standard 的 6.5.2.2 中 "The order of evaluation of the function designator, the actual arguments, and subexpressions within the actual arguments is unspecified, but there is a sequence point before the actual call."，表示 C 函式引數的順序不固定，這是為了符合 C 的宗旨「越貼近硬體越好」，與其人為規定順序，不如讓編譯器、硬體自己去優化。在 Rust Reference 的 8.2 [Evaluation order of operands](https://doc.rust-lang.org/reference/expressions.html#evaluation-order-of-operands) 中說 "The operands of these expressions are evaluated prior to applying the effects of the expression. Expressions taking multiple operands are evaluated left to right as written in the source code." 而 Call expression 正在 these expressions 的範圍內，以下例子可以看出來
```Rust
let mut one_two = vec![1, 2].into_iter(); 
assert_eq!( (1, 2), (one_two.next().unwrap(), one_two.next().unwrap()) );
```

---
# HW02
---
# LAB02

>[!question] 問題一
>以下兩句話如何理解?
>
>0) When the function returned by the lambda expression is called.
>1) When the lambda expression is evaluated.

"When the function returned by the lambda expression is called." 這句話中的 "function returned by the lambda expression" 表示該函式是由 lambda expresion 所回傳出來的，而最後的 "is called" 是說該 function 被呼叫。"When the lambda expression is evaluated." 在 python 中對於 expression 會 evaluate 像是 `3+5` ，而 lambda 函式是 expression，所以 python 會自己 evaluate，evaluate  的結果變是函式物件。這邊關鍵在區別函式的兩個時間點:
- 函式被建立的時間: lambda expression is evaluated，這時只是把函式的 parent frame 記錄好，把每個變數的尋找關係建立好，尚未真正的求值。
- 函式真正被呼叫的時間: the function ... is called，真正的新增一個 frame，沿著 parent frame 把變數值帶入進去。
例子:
```python
n = 1
f = lambda x: x + n

n = 100
print(f(5))
```
輸出是 105。

>[!important] 函式變數的找尋方式
>在函式定義的地方，會知道當地的 environment (parent frame 等) 並記下來 (用指標指著)。這 environment 所導出的 parent chain 決定了日後自由變數的**查找路徑**。在該函式被呼叫時，新增了一個 frame，記錄下參數和函式執行過程中在本地建立的名稱等，而其中的自由變數在此時才真正的沿 parent chain 找值帶入。


>[!question] 問題二
>以下在 python 的 interactive mode 下的 `c`、`c()` 指令輸出是啥
>```txt
>>>> # If Python displays <function...>, type Function, if it errors type Error, if it displays nothing type Nothing
>>>> b = lambda x, y: lambda: x + y # Lambdas can return other lambdas!
>>>> c = b(8, 4)
>>>> c
>>>> c()
>```

先觀察以下程式
```python
b = lambda x, y: lambda: x + y
```
這是兩個函式構成的 nested function，這兩函式分別是 `lambda x,y: ...` (f1) 跟 `lambda: x+y` (f2)。f1 需要兩參數 (`x`、`y`)  並回傳  f2，f2 不需要參數，回傳 `x+y` 其中 `x,y` 由於 closure 的關係來自 f1。 `c = b(8, 4)` 表示讓變數 `c` refer to `b(8,4)` ，而 `b(8,4)` 會回傳 f2，因此 `c` 會印出 Function。`c()` 表示呼叫 (call) f2 (也就是 `b(8,4)` 回傳的那個 lambda)，此時才計算 `x+y` 並回傳 12。

>[!question] 問題三
>以下在 python 的 interactive mode 下的 `one_thousand` 指令輸出是啥
>```txt
>>>> print_lambda = lambda z: print(z) # When is the return expression of a lambda expression executed?
>>>> one_thousand = print_lambda(1000)
>>>> one_thousand # What did the call to print_lambda return? If it displays nothing, write Nothing
>```

這題的問題是 `print(1000)` 後，`print()` 會回傳什麼? 在 python tutorial 4.8 (Defining Function) 中說到 "- The `return` statement returns with a value from a function. `return` without an expression argument returns `None`. Falling off the end of a function also returns `None`."，而 `print()` 的作用是副作用(往螢幕寫東西)，不回傳有意義的值，故回傳 `None`。

>[!question] 問題四
>以下函式輸出為何?
>```python
>print(not 10)
>print(not None)
>```

需要先知道 python 中 build-in 物件哪些是 False，依據 python standard library 的 built-in type，有以下物件
- constants defined to be false: `None` and `False`
- zero of any numeric type: `0`, `0.0`, `0j`, `Decimal(0)`, `Fraction(0, 1)`
- empty sequences and collections: `''`, `()`, `[]`, `{}`, `set()`, `range(0)`
所以輸出為
```txt
False
True
```

>[!question] 問題五
>請依照以下程式回答
>```python
>x = (1+1) and 5
>print(x)
>x = -1 or 5
>print(x)
>```
>1. 輸出是啥? 請給出理由
>2. C/C++ 跟 Rust 也用一樣的規則嗎? 

依照 python reference 6.11 (boolean operations) 的說明
- The expression `x and y` first evaluates _x_; if _x_ is false, its value is returned; otherwise, _y_ is evaluated and the resulting value is returned.
- The expression `x or y` first evaluates _x_; if _x_ is true, its value is returned; otherwise, _y_ is evaluated and the resulting value is returned.
可以看到短路求值 (short-circuit evaluation)，且可以回傳運算元，所以輸出為
```txt
5
-1
```
C/C++ 跟 Rust 都有短路求值，但他們都不能回傳運算元。會有這樣的差別是 python 是動態語言，而動態語言對於型別的要求不向靜態語言那麼嚴苛 (編譯時必須單一型別，所以對於 `x and y` 如果能回傳 `x` 或是 `y`，那遇到 `x,y` 型別不同時該回傳值會是兩型別聯集，靜態語言不接受)，在動態語言中有一哲學是「每個物件本身都帶有真假性（truthiness)」，正如 python standard library 的 built-in type 說的 "Any object can be tested for truth value"，所以動態語言可以直接用物件判斷真假。判斷完真假後回傳物件，對於直譯器沒太麻煩，同時可以簡化語法，所以在動態語言 (Lisp、Perl、Python、Ruby、JavaScript) 常常讓真假判斷完後，回傳運算元。

---
# Hog

>[!question] 問題一
>依據以下程式回答
>```python
>def roll_dice(num_rolls, dice=six_sided):
>	"""Simulate rolling the DICE exactly NUM_ROLLS > 0 times. Return the sum of the outcomes unless any of the outcomes is 1. In that case, return 1.
>
>	num_rolls: The number of dice rolls that will be made.
>	dice: A function that simulates a single dice roll outcome. Defaults to the six sided dice.
>	"""
>	# These assert statements ensure that num_rolls is a positive integer.
>	assert type(num_rolls) == int, 'num_rolls must be an integer.'
>	assert num_rolls > 0, 'Must roll at least once.'
>	# BEGIN PROBLEM 1
>	"*** YOUR CODE HERE ***"
>	# END PROBLEM 1
>	answer = 0
>	Sow_Sad_happened = False
>	while(num_rolls > 0):
>		add_value = dice()
>		if(add_value == 1):
>			Sow_Sad_happened = True
>		answer += add_value
>		num_rolls -= 1
>		
>	return 1 if Sow_Sad_happened else answer
>```
>1. `assert` 在幹嘛 ?
>2.  `return 1 if Sow_Sad_happened else answer` 這行在幹嘛?

先回答第一個問題，`assert` 是在做檢查，如果 `assert` 後接的條件不符合，會拋出 (raise) 一個 `AssertionError` 例外，如果沒有 `try/except` 接住，整個程式會中斷並印出錯誤。`return 1 if Sow_Sad_happened else answer` 是 python 的條件運算式(conditional expression)，有時也叫三元運算子，其表達格式是 `符合條件時的值 if 條件 else 未符合條件時的值` 。

>[!question] 問題二
>依據以下程式回答
>```python
>def sus_points(score):
>	"""Return the new score of a player taking into account the Sus Fuss rule."""
>	# BEGIN PROBLEM 4
>	"*** YOUR CODE HERE ***"
>	# END PROBLEM 4
>	if(num_factors(score) != 3 and num_factors(score) != 4):
>		return score
>	score += 1
>	while(not is_prime(score)):
>		score += 1
>	return score
>```
>1. 為啥 python 中沒有 `!`、`&&`、`||` 而是用 not、and、or 來表達 ?

因為 python 的設計哲學是「讓語言易懂」所以把 C 中的 `!`、`&&`、`||` 換成 not、and、or，而會選這些符號做替換是因為 python 的設計者受 ABC 語言的影響。另外和 `!`、`&&`、`||` 相似 `~`、`&`、`|` 跟 C 一樣被用來做位元運算。

>[!question] 問題三
>依據以下程式回答
>```python
>def play(strategy0, strategy1, update,score0=0, score1=0, dice=six_sided, goal=GOAL):
>	"""Simulate a game and return the final scores of both players, with
>	Player 0's score first and Player 1's score second.
>	"""
>	who = 0 # Who is about to take a turn, 0 (first) or 1 (second)
>	# BEGIN PROBLEM 5
>	"*** YOUR CODE HERE ***"
>	# END PROBLEM 5
>	while(score0 < goal and score1 < goal):
>		if(who == 0):
>			score0 = update(strategy0(score0, score1), score0, score1, dice)
>		else:
>			score1 = update(strategy1(score1, score0), score1, score0, dice)
>		who = not who
>		
>	return score0, score1
>```
>1. `who` 原本指定為 `0` 為 int 為啥可以用 `not` 來操作? bool 的 `who` 為啥可以直接跟 `1` 比 ?
>2. 為啥這函式可以一次回傳兩個值 ?

在回答第一個問題前，需要了解兩個 python 的概念。第一個是使用 `not` 後會回傳一個 `bool`；第二個概念是 `bool` 是 `int` 的子類別，bool 的 True 為 1 而 False 為 0。在 `not who` 是 `int` 藉由 `not` 回傳一個 `bool` ，而 `who == 0` 是因為 bool 是 int 的 subclass 所以比較結果符合預期。

>[!note] 在 C 中 `!` 也是回傳 bool 來實現 int 變 bool 嗎?
>不是。C 的 `!` 回傳的是 `int`（0 或 1），不是 bool。在 C 中`!E` 等價於 `(0 == E)`，結果的型別就是 int。

第二個問題是 python 給的一給 tuple packing，直接寫 `a,b,c` 在表達式 (expression) 會被當成一個 tuple 變成 `(a,b,c)` 所以 `return score0, score1` 不是分別回傳兩個值，而是回傳一個 tuple 物件，這 tuple 物件有兩個值，第一個為 score0 第二個為 score1。會選 tuple 而不是選 list，是因為這兩物件的設計哲學不同
- tuple: 一組異質的、位置有意義的、長度固定的資料(像一筆記錄)。  
- list: 一堆同質的、可增減的、長度會變的元素(像一個清單)。
還有一個原因是 tuple 是 immutable 的物件，而 list 是 mutable 的物件。

>[!note] 逗號變 tuple 的規範
>Except when part of a list or set display, an expression list containing at least one comma yields a tuple. The length of the tuple is the number of expressions in the list. The expressions are evaluated from left to right.

在規範中對於 display 的敘述如下
- For constructing a list, a set or a dictionary Python provides special syntax called “displays”
所以 tuple 的規範是除了在造 list 或是 set 外，只要在 expression list 加上逗號就會變成 tuple，而 expression list 的規定如下:
- `expression_list::=  expression("," expression)* [","]`
這是**BNF / EBNF**(巴科斯範式),是一種用來「描述某個東西的合法長相」的記號法，意思是: 一個 expression_list,是由
1. `expression`: 先放**一個**運算式(必要)
2. `("," expression)*`: 後面可以接「**逗號 + 一個運算式**」這組,重複 **0 到多次**
3. `[","]`: 最後可以再放**一個逗號**(可選)
所組成的。BNF 的符號對照表

| 符號                | 意思                             |
| ----------------- | ------------------------------ |
| `::=`             | 「**定義為**」。左邊的名字,就是右邊描述的這個結構    |
| `expression`(沒引號) | 一個**規則名稱**,代表「這裡要放一個運算式」(像填空格) |
| `","`(**有引號**)    | **字面上的逗號**,必須原封不動照打            |
| `( )`             | 純粹用來**框住一組東西**(分組),本身沒意義       |
| `*`               | 前面那組「**重複 0 次或多次**」            |
| `[ ]`             | 裡面的東西「**可有可無**」(出現 0 次或 1 次)   |
注: expression 的意思如下:
- **expression(運算式)**:任何「會算出一個值」的東西。`1`、`a + b`、`f(x)`、`score0`、`not who`、`len("abc")`——這些全都是 expression。

>[!question] 問題四
>依據以下程式回答
>```python
>def make_averaged(original_function, times_called=1000):
># BEGIN PROBLEM 8
>"*** YOUR CODE HERE ***"
># END PROBLEM 8
>	# answer = 0
>	def specific_original_function(*args):
>		answer = 0
>		for i in range(times_called):
>			answer += original_function(*args)
>		return answer / times_called
>	return specific_original_function
>```
>1. 為啥 `answer` 不能寫在 `def specific_original_function()` 那一層 ?
>2. `*args` 是啥 ?

在內層的 `answer += original_function` 對於 `answer` 賦值，所以 python 會把 `answer` 這變數加入 local 的 frame 中，而這之後的 `answer + original_function` 程式不知道 `answer` 是啥，因為沒賦值過，所以報錯。前述事情可在文法書的 4.2.2 中找到 "If a name binding operation occurs anywhere within a code block, all uses of the name within the block are treated as references to the current block."。不管把 answer 寫在外層的函式或是餐數列都沒用，除非使用 `nonlocal` 或是 `global`，
- `nonlocal`：指向最近一層「外圍函式」的變數（但不是 global）
- `global`：指向模組層級（最外層）的變數
另外你用外層的變數 (`answer`)，每次呼叫時記得歸零。
`*args` 是傳不定長度的參數的手段，還有一個方法是用 `**kwargs`。兩者的差別是 `*args` 需要參數的順序固定 (位置參數)，而 `**kwargs` 可以透過關鍵字的方式指定參數 (關鍵字參數)，在底層細節上 `*args` 傳的是 tuple `**kwargs` 傳的是 dictionary。看幾個例子:
```python
>>> list(range(3, 6))        # 一般呼叫,引數分開寫
[3, 4, 5]
>>> args = [3, 6]
>>> list(range(*args))       # 用 * 從 list 拆出引數
[3, 4, 5]
```

```python
>>> def parrot(voltage, state='a stiff', action='voom'):
...     print("-- This parrot wouldn't", action, end=' ')
...     print("if you put", voltage, "volts through it.")
...
>>> d = {"voltage": "four million", "action": "VOOM"}
>>> parrot(**d)
```

```python
def add(a, b, c):
    return a + b + c

nums = [1, 2, 3]
add(nums)      # ❌ TypeError:只傳了「一個 list」當 a,b、c 沒值
add(*nums)     # ✅ 等同 add(1, 2, 3) → 6
```

```python
def logger(*args, **kwargs):   # 定義:打包,什麼都收
    print("收到:", args, kwargs)
    return real_func(*args, **kwargs)  # 呼叫:拆包,原封轉交
```
`args` 進來是 tuple、`kwargs` 是 dict;轉交時 `*`、`**` 又把它們攤平還原。所以
- **函式定義的參數列**(`def f(*args, **kwargs)`)→ 打包(收集)
- **呼叫端**(`f(*args, **kwargs)`)→ 攤平(展開)






---





