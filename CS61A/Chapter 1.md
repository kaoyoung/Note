# Section 1.1

---
# Section 1.2

>[!question] 問題一
>為啥作者說: "Function notation has three principal advantages over the mathematical convention of infix notation."

我只想其中最重要的想法，在數學的 infix notation 需要定義運算的順序，像是先乘除後加減等，而這邊直接使用 function notation 可以無腦的套括號從內到外的運算。

>[!note] 概念一
> Assignment is our simplest means of _abstraction_, for it allows us to use simple names to refer to the results of compound operations 

藉由 assignment 例如: `area = 5*4` 我們可以把多的運算整合成一個抽象的概念，並在這概念上進行操作，甚至在遞規的把概念組在一起。

>[!note] 概念二
>The possibility of binding names to values and later retrieving those values by name means that the interpreter must maintain some sort of memory that keeps track of the names, values, and bindings. This memory is called an _environment_.

這邊的 environment 跟作業系統層級的**環境變數(environment variables)**，例如 `PATH`、`HOME`、`USER` 這類，它們是 OS 維護的一組 key-value 設定，程式執行時可以讀取(像是 Python 的 `os.environ`、C 的 `getenv()`) ，類似都是「某段執行所處的上下文/周遭條件」。

>[!question] 問題二
>為啥可以寫 `print = 5` 沒 keyword 的概念嗎?

Python 中是有 keyword，例如: `if`、`for`、`def`，但 `max`、`print`、`abs` 是內建名稱，它們只是 Python 啟動時,預先塞進一個叫 `builtins` 的命名空間裡的普通名字。對直譯器來說,`max` 和你自己定義的 `my_var` 地位是一樣的,只是 `max` 剛好被預先綁定到「那個求最大值的函式」而已。**其實 `printf` 在 C/C++ 中也不是 keyword**，他是 `#include <stdio.h>` 宣告的一個**函式**而已，所以你在 global scope (main 之外) 做 `int printf = 5` 肯定炸，因為該 scope 已經定義過 `printf` 導致重複宣告(符號衝突)而失敗，另一方面 `printf` 指到執行該函數的程式碼，你無法赋值 (`printf = 5`)。如果硬要宣告叫 `printf` 的變數或是赋值可以用 shadowing 的概念，在某一個 scope (例如: `main()` 內) 內做 `int printf = 5`。


>[!question] 問題三
>`x, y = 5, 4` 之後用 `x, y = y ,x` ， `x,y` 的值分別是多少?

這想當於 `x,y` 做一個 swap，也就是 `x` 為 4 而 `y` 為 5。關鍵在於: "With multiple assignment, _all_ expressions to the right of = are evaluated before _any_ names to the left are bound to those values" 右邊先處理完，在赋值給左邊。在這邊意思是，右邊的 `y,x` 已知在一次性套到左邊的 `x,y` 也就不存在順序性的問題。

---
# Section 1.3 

>[!note] 概念一
>An import statement binds a name to a built-in function. A def statement binds a name to a user-defined function created by the definition.

想表達名字跟函式的 bind 關係。

>[!definition] 定義一 (Function Signature)
> A description of the formal parameters of a function is called the function's signature.

函式吃的變數 (argument) 數量跟型態不同，所以我們用 function signature 來表達這函式的一個特質。
- argument (引數): 實際傳入的值
- parameter (參數): 函式定義的變數名稱

>[!note] 概念二
>Regardless of the number of arguments taken, all built-in functions will be rendered as `<name>(...)`, because these primitive functions were never explicitly defined.

語言自帶的函示是系統最底層的內建函數，系統統一用 `...` 來代表它可以放參數而已。這些函式通常是用更底層的程式碼（例如 C 語言）直接寫死在系統核心裡的，所以它們並不是用該語言本身的標準語法寫出來的，所以系統無法（或不需要）給你一份詳細的參數清單。

>[!note] 概念
>Python provides two infix operators: `/` and `//`. The former is normal division, so that it results in a _floating point_, or decimal value, even if the divisor evenly divides the dividend:

須注意這邊跟 C/C++ 不同，`/` 表示真正的除法會自動切換到浮點數，例如: `8/4` 會是 `2.0`。在 `//` 結果為負時 C/C++ 跟 python 結果不同。在 C/C++ 時 `-7/2` 結果為 -3 而在 python 中 `-7/2` 結果為 -4。

---
# Section 1.4

>[!note] 概念一
>A function definition will often include documentation describing the function, called a _docstring_, which must be indented along with the function body.

這邊的想法是基於 "code is written only once, but often read many times." 所以我們應該多寫一點東西幫助人類理解。例子如下:
```python=
 def pressure(v, t, n):
        """Compute the pressure in pascals of an ideal gas.

        Applies the ideal gas law: http://en.wikipedia.org/wiki/Ideal_gas_law

        v -- volume of gas, in cubic meters
        t -- absolute temperature in degrees kelvin
        n -- particles of gas
        """
        k = 1.38e-23  # Boltzmann's constant
        return n * k * t / v
```

>[!note] 概念二
>In the `def` statement header, `=` does not perform assignment, but instead indicates a default value to use when the pressure function is called

在函數的 parameter 中用 `=` 是用來表示預設值而非赋值，預設值是在函數建立的那一瞬間就被建立好的，不會再改變。例子如下
```python
x = 5
def f(a=x)
	return a

x = 10
print(f()) # 會輸出 5 而非 10
```
這例子會輸出 5 而非 10



---
# Section 1.5 (Control)

>[!note] 概念一
>Each statement describes some change to the interpreter state, and executing a statement applies that change.

可以看以下例子
- Expressions, return statements, and assignment statements are simple statements.
- A def statement is a compound statement. The suite that follows the def header defines the function body.
舉個具體例子: `return x+y` 是 statement 其中的 `x+y` 是 expression，但如果是單純寫 `x+y` 為一行則為 expression statement，因為程式的執行單位是 statement，要讓一個 expression 出現在程式裡，就必須把它包成 expression statement。

>[!definition] 定義一 (statement, expression)
>- Statement: A **statement** is the smallest independent syntactic unit of an imperative programming language that expresses some **action** to be carried out. It is executed for its **side effects** rather than its value.
>- Expression: An **expression** is a phrase of computer code that can be **evaluated** according to the language's syntax and semantic rules to produce a **single value**.

statement 是被「執行 (execute)」的，expression 是被「求值 (evaluate)」的

>[!note] 概念二
>A while clause contains a header expression followed by a suite:
>```C
>while <expression>:
>    <suite>
>```

名詞解釋:
- suite 是指縮排的部分。
- clause 是指包含 header 跟 suite 的部分，所以 `if` 跟 `elif` 為兩個 clause。

>[!note] 概念三
>An `assert` statement has an expression in a boolean context, followed by a quoted line of text (single or double quotes are both fine, but be consistent) that will be displayed if the expression evaluates to a false value.

這邊說了 python 測試的其中一個語法: `assert`。可以看以下例子
```python=
def fib_test():
   assert fib(2) == 1, 'The 2nd Fibonacci number should be 1'
   assert fib(3) == 1, 'The 3rd Fibonacci number should be 1'
   assert fib(50) == 7778742049, 'Error at the 50th Fibonacci number'
```

>[!note] 概念四
> The first line of a docstring should contain a one-line description of the function, followed by a blank line. A detailed description of arguments and behavior may follow. In addition, the docstring may include a sample interactive session that calls the function

另一個 python 中的測試方法是 doctests，舉個例子
```python
def sum_naturals(n):
    """Return the sum of the first n natural numbers.

    >>> sum_naturals(10)
    55
    >>> sum_naturals(100)
    5050
    """
    total, k = 0, 1
    while k <= n:
        total, k = total + k, k + 1
    return total
```

如果要測試所有函式的 doctests 可以用以下方式
```python
from doctest import testmod
testmod()
```
如果是用單一函式，可以用以下函式
```python
from doctest import run_docstring_examples
run_docstring_examples(sum_naturals, globals(), True)
```
如果要在 CLI 使用，輸入以下指令
```shell
python3 -m doctest -v <python_source_file> # -v 表示想在全部執行結果，沒加時「預設失敗」才有輸出
```

---
# Section 1.6 (Higher-Order Functions)

>[!note] 概念一
>Functions that manipulate functions are called higher-order functions.

>[!note] 概念二
>some functions express general methods of computation, independent of the particular functions they call

抽象化函式內部的操作，只提供概念性的操作流程，具體細節再自行填上去。
```python
def improve(update, close, guess=1):
   while not close(guess):
       guess = update(guess)
   return guess
```

如上的函式，定義了一個逼近的操作過程，而具體方法需要另外去寫，並透過 `update`, `close`, `guess` 來操作。

>[!question] 問題一
>以上的寫法在 C/C++ 中如何做?

在 C 中要靠函式指針來傳遞實作的函數
```C
typedef double (*update_fn)(double);
typedef int    (*close_fn)(double);

double improve(update_fn update, close_fn close, double guess) {
    while (!close(guess)) guess = update(guess);
    return guess;
}

double golden_update(double g) { return 1.0/g + 1; }
int approx_eq(double x, double y) { return fabs(x - y) < 1e-3; }
int square_close_to_successor(double g) { return approx_eq(g*g, g + 1); }

double phi = improve(golden_update, square_close_to_successor, 1);
```

>[!question] 問題二
>上面的解偶方式有以下兩問題如何處理?
>1. 多個實現的函數，在 global frame 需要多個不重複的名字，導致混亂
>2. 函數的參數長度可能是多變的

這裡的思路是:「 improve 的架構是穩定的，不要再動，思考不讓函數名字汙染 global scope，同時讓函數所需的其他參數可以取得」。這邊使用到的方法叫 nested function 和 lexical scope 的概念。Lexical scope 讓我們可以把多餘的變數寫在外層的環境中，內層函式可以「讀取／捕獲」外層 frame 裡的自由變數，這解決參數長度多變的問題；另一方面，nested function，讓我們在外層函式之外，無法存取它內部定義的輔助函式，如此解決了多個不重複名字的問題。

>[!note] 概念三
>This discipline of sharing names among nested definitions is called _lexical scoping_. Critically, the inner functions have access to the names in the environment where they are defined

這邊在使用 lexical scope 時我們需要 environment model (環境模型是一種解釋程序執行時變量如何查找和綁定的抽象模型) 支援以下兩件事
1. Each user-defined function has a parent environment: the environment in which it was defined.
2. When a user-defined function is called, its local frame extends its parent environment.
這兩件事說明函式 paremt environemt 是何時決定和函式 local frame 跟 parent environment 的關係。

>[!question] 問題三
>namespace, frame, environment, closure 在 python 中代表啥？

這幾個名字橫跨了多個範圍，有的是資料結構 (ex: namespace)，有的是執行期的物件 (ex: frame, closure)，有的是程式語言理論的抽象 (environment)。逐一來說
- namespace: 一種紀錄名字對應到物件的資料結構，供程式之間互相引用名字使用。
- frame: 一個執行期的物件，每次呼叫函式時，CPython 便建立一個 frame，每一個 frame 包括: 好幾個 namespace、執行到第幾行、指向呼叫者等。
- environment: 一個程式語言理論的抽象，指當前 frame 依照 LEGB (Local -> enclosing -> global -> bulit-in) 順序可存取到的所有名字，其中 L / G / B 對應當前 frame 的三個 namespace，而 E 對應 closure 捕捉到的外層變數（語彙決定，定義時就固定）。
- closure: 一個執行期的物件，該物件是「函式物件 + 它捕捉到的外層變數」。

>[!note] 概念四
>An important feature of lexically scoped programming languages is that locally defined functions maintain their parent environment when they are returned.

在 lexical scope 中我們在內層函式可以獲取外成 frame 的變數，同時 lexical scope 的 closure 實作讓被回傳的內層函式記住它定義時的父層 frame，使得即使父函式已經結束，內層函式仍能繼續使用父層的變數。須注意 parent environment 是在定義時決定而不是執行中決定的。


>[!question] 問題四
>lexical scope 跟 closure 差在哪？

- lexical scope: 講的是規則，如果無法在當前 frame 找到該變數，可以到該 frame 定義的外層 (祖先 frame) 依順序去找。
- closure: 講的是一個「執行期的物件/機制」，它是「函式 + 它定義時所在的環境」綁在一起的那包東西,目的是讓上面那條規則在函式被回傳、離開原本的 frame 之後仍然成立。

>[!note] 概念五
>Python, we can create function values on the fly using lambda expressions, which evaluate to unnamed functions. A lambda expression evaluates to a function that has a single return expression as its body. Assignment and control statements are not allowed.

有 lambda 表達式，我們可以讓只有 single reture expression 作為 body 的函式不一定要有名子，可以直接被使用，相當於數字、字面值的物件也可以不需要名字直接使用，但需要注意 lambda 表達式預設沒有名字代表無法被 namespace 找到，所以只能作為當下的物件，用完即丟 (**前提是它沒被任何名字綁住、也沒被 return 或傳出去**——這樣的話，等到用它的那個暫時 frame 結束，就沒有任何路徑能再指到它，於是被回收，你之後再也叫不出來；**但只要它被某個名字接住，或被 return 出去由呼叫端接住，它就會繼續活著，不會被丟掉**)，除非你指定一個名字給他，例如: `square = lambda x: x * x`。再看個 lambda 的語法
```python
def compose1(f, g):
   return lambda x: f(g(x))
```
把 `lambda x: f(g(x))` 翻譯成自然語言是: "A function that    takes x    and returns     f(g(x))"，其中 "a function that" 對應 `lambda`，"takes" 對應 `x`，"and returns" 對應 `:`，" f(g(x))" 對應 ` f(g(x))`。


>[!note] 概念六
>Elements with the fewest restrictions are said to have first-class status. Some of the "rights and privileges" of first-class elements are:
>1. They may be bound to names.
>2. They may be passed as arguments to functions.
>3. They may be returned as the results of functions.
>4. They may be included in data structures.
>
>Python awards functions full first-class status, and the resulting gain in expressive power is enormous.

這邊在說 Python 中的函式是 first-class elements，代表 function 可以用一個名稱代表、傳進一格函式、當函式的回傳值、封裝在一個資料結構內。Python 中函式本身就是一種「值」、是一個物件,而不只是「可以被呼叫的特殊語法」，相當於函式跟數字、字串這些資料值地位相等，都是「一等公民」。

>[!note] 概念七
>Python provides special syntax to apply higher-order functions as part of executing a def statement, called a decorator.

介紹了 python 中的一個 decorator 的語法，這只是個語法糖 (Syntactic sugar)，我們可以把以下函式
```python
def trace(fn):
   def wrapped(x):
       print('-> ', fn, '(', x, ')')
       return fn(x)
   return wrapped
   
def triple(x):
   return 3 * x

triple = trace(triple)
triple(12)
```
寫成
```python
def trace(fn):
   def wrapped(x):
       print('-> ', fn, '(', x, ')')
       return fn(x)
   return wrapped
  
@trace
def triple(x):
   return 3 * x

triple(12)
```
decorator 的「套用時機」是在 `def` 執行的當下，而不是在呼叫 `triple(12)` 的時候。

---
# Section 1.7 (Recursive)

>[!note] 概念一
>The iterative function constructs the result from the base case of 1 to the final total by successively multiplying in each term. The recursive function, on the other hand, constructs the result directly from the final term, n, and the result of the simpler problem, fact(n-1).

上述是在說明求 $n$ 階層的兩種實踐方法與其對應思路。iterative 思考方式是從底部往上去推最後結果；recursive 思考方式是從頂部往下去推直到 base case，得到最後結果。

>[!note] 概念二
>That is, we should not care about how fact(n-1) is implemented in the body of fact; we should simply trust that it computes the factorial of n-1. Treating a recursive call as a functional abstraction has been called a _recursive leap of faith_.

說明用 recursive 求 $n$ 階層的過程中，我們應該抽象的去看這函式，想成一個封裝好的求值器。

>[!definition] 定義一
>- mutual recursion: When a recursive procedure is divided among two functions that call each other, the functions are said to be _mutually recursive_.
>- tree recursion:  A function calls itself more than once.








