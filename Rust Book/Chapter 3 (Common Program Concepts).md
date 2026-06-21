# Variable and Mutability

```rust
let x = 5; // 預設不可變，安全且靜態 
let mut y = 5; // 明確宣告可變，y 可以被修改
```

>[!question] 為啥 rust 變數預設為不可變 (immutable)?
>在 c/c++ 中，變數如果沒用 `const` 來設定的話，其值可以不停變化，導致我們可能會不小心改到，或是在多線程中有 race condition 的問題。 rust 為了讓你在用可變變數時多想一下，而把變數預設為不可變 (immutable)，不可變變數還可以讓編譯器大膽的最佳化。

>[!question] 為啥在 RUST 中變數不用指定型別，這樣記憶體怎麼知道分配多大? 程序員如何知道該值的邊界在哪?
>Rust 是 statically and strongly type 的語言，所以在編譯期變數的型別必須決定且禁止隱性型別轉換 (Implicit Type Conversion)，在沒給定的變數編譯器會自動幫你做型別推斷 (Type Inference)。在預設情況下
>- 沒特別寫的整數: `u32`
>- 沒特別寫的浮點數: `f64`

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

>[!question] `let` 的變數在不加 `mut` 下跟 `const` 差在哪? 不是都是不可變
>兩者的想法不太一樣。對於 `let` 來說是在編譯期確定型別正不正確，而值執行期才會填進去；對於 `const` 來說是在編譯期確定型別正不正確且型別需明白指定，同時值也要確定。這讓 `let`  可以接受函數的回傳值，而 `mut` 不行。

```Rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");
    }

    println!("The value of x is: {x}");
}
```
- In effect, the second variable overshadows the first, taking any uses of the variable name to itself until either it itself is shadowed or the scope ends.
- Shadowing is different from marking a variable as `mut` because we’ll get a compile-time error if we accidentally try to reassign to this variable without using the `let` keyword.

```rust
let guess = "42"; // 第一個 guess 是字串
let guess: u32 = guess.parse().expect("Not a number!"); // 第二個 guess 是數字，完美遮蔽了第一個
```
- The other difference between `mut` and shadowing is that because we’re effectively creating a new variable when we use the `let` keyword again, we can change the type of the value but reuse the same name

>[!question] 為啥 rust 要用 shadowing?
>核心思想是「一個資料在一個時間點只有一個型別重要」，例如可能在年齡在表單上時是 `"42"` ，但在做判斷時需要轉成整數，所以需要一個可以隨時轉換型別的方法，而在 rust 中就用 shadowing 來處理這件事。另外你可以用 shadowing 在快速初始化或修改值後永遠不要再被改動，例如
>```rust
>// 1. 宣告為可變，因為我們需要改變它
let mut spaces = "   ".to_string(); 
spaces.push_str("!"); 
// 2. 初始化完成了，我們不希望後面不小心改到它
// 透過 Shadowing，重新宣告一個「不可變」的 spaces
let spaces = spaces; 
// spaces.push_str("?"); // 這裡編譯器會報錯，因為現在的 spaces 是不可變的！
>```

>[!question] 在 C/C++ 中，不允許在同一個作用域（Scope）內重複定義同名變數（Redefinition）；但 Rust 卻允許在同一個 Scope 內透過 `let` 不斷遮蔽（Shadowing）同名變數。為什麼標榜記憶體安全的 Rust 反而放寬這種限制？
>可以從一個較為抽象的角度切入，即「我們要傳遞一個抽象的概念，其可以有多個型別來表達，但在某一個時刻它只需要一個型別」，舉個例子:「我需要知道一個人的年齡，可以在文書上用字串表達，在報表上用整數表達，但在具體使用場景只需要一個型別即可」，所以我們用 shadowing 可以讓這抽象概念按需求決定型別。如此我們不需要命名多個變數只為表達一概念，更重要的是，它強制讓過時的表現形式失效，避免了誤用舊資料的風險。

>[!question] 不同型別的變數用不同名稱不是更安全嗎?
>回想一下為什麼不同型別的變數用不同名稱會讓你覺得更安全，會這樣想是因為「我不把型別寫在名字上，我不知道現在是啥型別」，但 Rust 是靜態強情形別語言，編譯器會幫你把關，同時 IDE 會用 Inlay Hints 顯示推倒型別，所以 Rust 不太擔心這問題。 RUST 更要防範「不小心用了不該用的舊狀態」，所以用 shadowing。

>[!question] 這邊 rust 的思想是「一個資料在一個時間點只有一個型別重要」，那如果同時要用兩個以上的型別呢?
>如果同時需要多個型別，代表資料的**生命週期**或**存取權**發生了重疊。此時若強行 Shadowing 會導致舊資料提早被 Drop（如果是擁有所有權的型別），所以使用獨立變數或 Struct 封裝，本質上是在**明確定義這組資料的共同生命週期與所有權邊界**。

---
# Data Types
We’ll look at two data type subsets: **scalar** and **compound**.

> Rust is a **statically typed language**, which means that it must know the types of all variables at **compile time**.

Hence if there are many types are possible, we must add a  type annotation, like this:
```rust
let guess: u32 = "42".parse().expect("Not a number!");
```
## Scalar 
Rust has four primary scalar types: 
- integers, 
- floating-point numbers, 
- Booleans, and 
- characters.
### Integer

![[integer bit length.png]]
-  the `isize` and `usize` types depend on the architecture of the computer your program is running on: 64 bits if you’re on a 64-bit architecture and 32 bits if you’re on a 32-bit architecture.

![[Data Type.png]]
- Note that number literals that can be multiple numeric types allow a type suffix, such as `57u8`, to designate the type.

>Integer Overflow:
>Integer overflow has tow different behaviours
>- **debug mode**: Rust **includes checks** for integer overflow that cause your program to **panic at runtime** if this behavior occurs. Rust uses the term **panicking** when a program exits with an error.
>- **release mode**: Rust does **not include checks** for integer overflow that cause panics. Instead, if overflow occurs, Rust performs **two’s complement wrapping**.
>
>To explicitly handle the possibility of overflow, you can use these families of methods provided by the standard library for primitive numeric types:
>- Wrap in all modes with the `wrapping_*` methods, such as `wrapping_add`.
>- Return the `None` value if there is overflow with the `checked_*` methods.
>- Return the value and a Boolean indicating whether there was overflow with the `overflowing_*` methods.
>- Saturate at the value’s minimum or maximum values with the `saturating_*` methods.
>

>[!question] 這邊說的程式 panic 是啥意思?
>程式遇到了無法處理的嚴重錯誤，因此強制中斷執行並退出。
### Floating-Point Types
Rust also has two primitive types
- `f32`
- `f64`
The default type is `f64` because on modern CPUs, it’s roughly the same speed as `f32` but is capable of more precision. All floating-point types are **signed**.
```c
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```
- Floating-point numbers are represented according to the **IEEE-754 standard**.
### Numeric Operations
```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // Results in -1

    // remainder
    let remainder = 43 % 5;
}
```
### Boolean Type
Booleans are **one byte** in size. The Boolean type in Rust is specified using `bool`
```rust
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

>[!question] 為啥 boolean 是 1 byte 不是 1 bit? 如果要 1 bit 操作? 在 C/C++ 中也這樣嗎?
>Rust 跟 C 的 boolean 都是 1 byte，而 C++ 中 boolean 通常是 1 byte，會這樣設計主要是 CPU 最小的尋址大小通常是 1 byte，要找 1 bit 還要配合位移或掩碼處理，會增加指令數，還有一原因是 Alignment。在 Rust 要 1 bit 可以用 `u8` 或 `u64` 配合位運算處理，用 `bitflags`，或是 `bitvec`。

### Character Type
```Rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```
- Rust’s `char` type is 4 bytes in size and represents a Unicode scalar value, which means it can represent a lot more than just ASCII.
	- Accented letters
	- Chinese
	- Japanese
	- Korean
	- emogi
	- zero-width space

>[!question] 為啥 Rust 的 character type 要用 4 byte 而不是 1 byte?
>在 C/C++ 時電腦主要是 ASCII 編碼， 1 byte 綽綽有餘，但為了融合進 Unicode 所以變 4 byte。在 Rust 強制區分了 **「位元組 (`u8`)」** 和 **「人類認知的字元 (`char`)」**。
>- 如果你需要像 C 語言一樣操作 1 Byte 的資料（例如處理網路封包、二進位檔案），請使用 **`u8`**。
>- 如果你需要處理「文字的邏輯單元」（例如判斷某個字元是不是大寫、是不是數字），請使用 **`char`**（4 Byte 讓它能安全涵蓋全世界所有字元）。
## Compound
### Tuple
A _tuple_ is a general way of grouping together a number of values with a variety of types into one compound type. Tuples have a fixed length: Once declared, they cannot grow or shrink in size.

```Rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```
- The variable `tup` binds to the entire tuple because a tuple is considered a single compound element.

```Rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {y}");
}
```
- To get the individual values out of a tuple, we can use pattern matching to destructure a tuple value

```Rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```
- We can also access a tuple element directly by using a period (`.`) followed by the index of the value we want to access.
- The tuple without any values has a special name, `unit`.  This value and its corresponding type are both written `()` and represent an **empty value** or an **empty return type**.
- Expressions implicitly return the **unit** value if they don’t return any other value.

>[!question] 為啥是一個空的東西當 unit? unit 應該是 1 吧。
>這邊想說的是可能狀態唯一。
### Array
Unlike a tuple, every element of an array must have the **same type**. Unlike arrays in some other languages, arrays in Rust have a **fixed length**.

```Rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

```Rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];

```
- Arrays are more useful when you know the number of elements will not need to change.
```Rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```
- You write an array’s type using square brackets with the type of each element, a semicolon, and then the number of elements in the array
```Rust
let a = [3; 5];   // this is the same as let a = [3, 3, 3, 3, 3];
```
- initialize an array to contain the same value for each element by specifying the initial value, followed by a semicolon, and then the length of the array in square brackets
```Rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```
- An array is a **single chunk of memory** of a known, **fixed size** that can be allocated on the **stack**.
```Rust
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!("Please enter an array index.");

    let mut index = String::new();

    io::stdin()
        .read_line(&mut index)
        .expect("Failed to read line");

    let index: usize = index
        .trim()
        .parse()
        .expect("Index entered was not a number");

    let element = a[index];

    println!("The value of the element at index {index} is: {element}");
}
```
If you instead enter a number past the end of the array, such as `10`, you’ll see output like this:
```text
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
- **runtime error** at the point of using an invalid value in the indexing operation.
- When you attempt to access an element using indexing, Rust will check that the index you’ve specified is less than the array length.
- Rust protects you against this kind of error by immediately exiting instead of allowing the memory access and continuing.

---
# Functions
### Parameters
```Rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {x}");
}
```
- **Parameter**: A variable listed in a function's definition. It acts as a placeholder for the value that will be passed into the function.
- **Argument**: The actual concrete value that is passed into the function when it is called.
- In function signatures, you _must_ declare the type of each parameter.

> This is a deliberate decision in Rust’s design: Requiring type annotations in function definitions means the compiler almost never needs you to use them elsewhere in the code to figure out what type you mean.

### Statements and Expressions
Function bodies are made up of a series of statements optionally ending in an expression.
- **Statements**: are instructions that perform some action and do not return a value.
- **Expressions**: evaluate to a resultant value.

```Rust
fn main() {
    let y = 6;
}
```
- Function definitions are also statements; the entire preceding example is a statement in itself.
```Rust
fn main() {
    let x = (let y = 6);
}
```
- Statements do not return values. Therefore, you can’t assign a `let` statement to another variable; **you’ll get an error**
-  In those languages, you can write `x = y = 6` and have both `x` and `y` have the value `6`; that is not the case in Rust.

>[!question] 這邊在說的跟 `x=y=6` 在其他語言可以，但在 Rust 中不行在說啥?
>在 C 中允許 `x=y=6` 是因為 `y=6` 執行完後把 6 當右值 (R-value) 交給 `x` 繼續使用；Rust 不允許
>```Rust
>let mut x = 6;
>let mut y = 6;
>x = y = 6;
>```
>他會噴錯誤，因為賦值表達式的類型式 `()` 而 `x` 期望的是 `i32`
>```txt
>error[E0308]: mismatched types
>```
>Rust 會這樣設計是借鏡了函數式編程的想法: 「把表達式跟改變變數的值分開」，`y=6` 專注於改變變數的值，而不去管表達式(產生新的值)。另一方面這可以避免一個 C 的 bug
>```C
>int a = 5;
>if( a = 3){
>	printf("a is %d\n", a);
>}
>printf("a is %d\n", a);
>```
>他會輸出
>```text
>a is 3
>a is 5
>```
>「改變變數的值」跟「回傳值讓你賦值」如果分開，可以避免這個 bug。


```Rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```
-  A new **scope block** created with curly brackets is an expression
	- Calling a function is an expression. 
	- Calling a macro is an expression.
- **Expressions do not include ending semicolons**. If you add a semicolon to the end of an expression, you turn it into a statement, and it will then not return a value.

>[!question] 為啥這邊說 "Expressions do not include ending semicolons"，可是 `let y = 6;` 也要 semicolons 阿?
>因為 `{}` 在 Rust 中是表達式要回傳一個值，所以最後的 `x+1` 不能加分號，如果加的話會變陳述式，類型變為 `()` ，而讓 `y` 無法賦值。這邊也有跟 `let y = 6;` 一樣的分號，在`{}` 後面。

### Functions with Return Values
```Rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```
-  In Rust, the return value of the function is synonymous with **the value of the final expression** in the block of the body of a function.
- a lonely `5` with **no semicolon** because it’s an expression whose value we want to return.
- You can return early from a function by using the `return` keyword and specifying a value

>[!question] 為啥要 expresion 回傳，直接寫 `return` 不是更簡明嗎?
>先說一下為啥這邊的 `5` 不用加分號，Rust 本身是「表達式導向 (Expression-Oriented) 的語言」，而 `{}` 本身是一個表達式，而他的值取決於最後一個表達式，所以這邊的 `5` 不用加分號，如果加上分號會變成陳述式，其值會變成 `()` 。Rust 為了讓函式體和其他區塊 (block) 一樣都是表達式，其值就是最後一個表達式的值，所以這邊不加 `return` 因為沒有要提早返回，直接看最後一個就好。

```Rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```
- If we place a semicolon at the end of the line containing `x + 1`, changing it from an expression to a statement
Compiling this code will produce an error, as follows:
```txt
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error[E0308]: mismatched types
 --> src/main.rs:7:24
  |
7 | fn plus_one(x: i32) -> i32 {
  |    --------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
8 |     x + 1;
  |          - help: remove this semicolon to return this value

For more information about this error, try `rustc --explain E0308`.
error: could not compile `functions` (bin "functions") due to 1 previous error
```

---
# Comments
```rust
fn main() {
    // I'm feeling lucky today
    let lucky_number = 7;
}
```

---
# Control Flow

### if Expressions
```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

>[!question] 為啥 rust 的判斷不用小括號刮起來?
>可以避免和元組 (tuple) 或函式調用產生歧異。須注意 Rust 只是不強制 `if` 條件式外層的括號，內部是可以有括號的
>```Rust
>if (x > 0 || y > 0) && z == 0 {
 >   println!("x 或 y 大於 0，且 z 必須為 0");
>}
>``` 

```rust
fn main() {
    let number = 3;

    if number {
        println!("number was three");
    }
}
```
**Rust will throw an error**
```txt
--> src/main.rs:4:8
  |
4 |     if number {
  |        ^^^^^^ expected `bool`, found integer

For more information about this error, try `rustc --explain E0308`.
error: could not compile `control_flow` (bin "control_flow") due to 1 previous error
```
**The correct code is as follow**
```rust
fn main() {
    let number = 3;

    if number != 0{
        println!("number was three");
    }
}
```

>[!question] 為啥 Rust 不支持隱式轉換? 
>~~因為它是 strong typing language。~~ (這回答不精確，python 也是強型別語言，但它允許 `int` 跟 `float` 相加)。
>Rust 的核心思想是: 安全性 (safety)、明確性 (explicitness) 和可預測性 (predictability)
>如果允許隨便隱式轉型像 `int` 轉 `short` 這種大範圍轉小範圍，或是 C/C++ 經典的 `signed` 變 `unsigned`，例如
>```C
>int a = -1;
>unsigned b = 1;
>if(a > b){
>	printf("it's ridiculous");
>}
>```
>它會輸出
>```txt
>it's ridiculous
>```
>所以 Rust 把所有可能引發精度消失或歧異的隱式轉型都拿走，如果執意為之需要用 `as` 或是 `.into()` 呼叫。Rust 其實存在非常侷限且安全的隱式轉換，最著名的就是 **Deref Coercion**（例如將 `&String` 隱式轉換為 `&str`）。

>[!question] C/C++ 中如何支持隱式轉換
>由編譯器根據預設的層級跟規則來自動安排轉型指令。常見的有
>- Integer Promotion: 比 `int` 小的整數型別 (`char`、`short`、`bool`) 提升至 `int` (或 `unsigned int`)
>- Usual Arithmetic Conversion: 較小 (精度較低) 的型別轉成較大 (精度較高) 的型別
>	- 層級概念: `int` -> `unsigned int`-> `long`-> `float`-> `double`
>- Assignment Conversions: 把一個值塞給不同型別時，值會自動迎合該型別
>- Array-to-Pointer Decay
>- Boolean Conversions

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

>[!question] 在 Rust 中 `==, %, *, +, -, &, |, <<, >>, &&, ||` 的優先級為何?
>優先順序為
>1. `as`
>2. `*`、`/`、`%`
>3. `+`、`-`
>4. `<<` 、`>>`
>5. `&`
>6. `^`
>7. `|`
>8. `==`、`!=`、`<`、`>`、`>=`、`<=`
>9. `&&`
>10. `||`

``` rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```

>[!question] 為啥在 C/C++ 中不能這樣寫，而在 RUST 可以?
>在 C/C++ 中 `if else` 只是陳述式 (statement)，本身沒有返回值只是在做流程控制；而在 Rust 是表達式 (expression)，本身有返回值，所以可以直接用它來負值，類似於 C/C++ 中的三元運算式（Ternary Operator）。在 Rust 中`if`、`loop`、甚至區塊 `{}` 都是表達式。

>[!question] 為啥在 if else 的中括號中不用分號?
>因為在 Rust 裡，分號會把表達式變成陳述式，並使它的回傳值 `()` ，無法跟 `number` 的型別匹配。


```Rust
fn main() {
    let condition = true;

    let number = if condition { 5 } else { "six" };

    println!("The value of number is: {number}");
}
```
- When we try to compile this code, we’ll get an **error**.
- variables must have a single type, and Rust needs to know definitively at **compile time** what type the `number` variable is.

>[!question] 為啥 Rust 要在 compile time 知道變數型別?
>因為它是 static typing language。

### Repetition with Loops
Rust has three kinds of loops: `loop`, `while`, and `for`.
```Rust
fn main() {
    loop {
        println!("again!");
    }
}
```
- The `loop` keyword tells Rust to execute a block of code over and over again either forever or until you explicitly tell it to stop.
- You can place the `break` keyword within the loop to tell the program when to stop executing the loop

```Rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```
- After the loop, we use a semicolon to end the statement that assigns the value to `result`.
- You can also `return` from inside a loop. While `break` only exits the current loop, `return` always exits the **current function**.

>[!question] 為啥把 `break counter * 2;` 改成 `return counter * 2;` 不會跳出內層 loop?
>- `break` 只會退出**最內層的迴圈**，然後繼續執行迴圈之後的程式碼（例如 `println!`）。
>- `return` 則會**立即結束當前函式**，並把值傳回給呼叫者。整個函式結束，迴圈下面的任何程式碼都不會再執行。

>[!question] 為啥 `break counter * 2;` 要加分號，它不是 expression 嗎?
>這邊的 `break` 本身就會帶著值回傳，所以最後有沒有加分號都沒差，他本身就不在乎後續的變化，書中加分號單純是明確語句結尾。

```Rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```
- Loop labels must begin with a **single quote**.
- The first `break` that doesn’t specify a label will exit the inner loop only.
- The `break 'counting_up;` statement will exit the outer loop. 

>[!question] 這設計是為了脫離巢狀結構並避免使用 `goto` 嗎?
>其時在 Rust 中沒有 `goto` ，因為 `goto` 有兩個常見應用，「跳出巢狀迴圈」跟「集中式的錯誤處理跟資源回收」。「集中式的錯誤處理跟資源回收」被 RAII (resource acquistion is initialization) 處理，RAII 保證資源是綁定在變數的生命週期上的，當你在 Rust 中打開一個檔案或分配一塊記憶體，只要程式執行離開了該變數的「作用域 (Scope)」（大括號結束），編譯器就會**自動**幫你呼叫 `Drop` (解構函式) 釋放資源。而「跳出巢狀迴圈」則是靠上文的標籤機制處理。

>[!question] Rust 的 loop label 一定要加 `'` 嗎?
>直觀想法: 如果 loop lable 不加 `'` 那 `break counting_up;` 你要怎麼知道 `counting_up` 是標籤還是要回傳的變數? 所以解法是 loop label 一定要加 `'`。

```Rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```
- This construct eliminates a lot of nesting that would be necessary if you used `loop`, `if`, `else`, and `break`

```Rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }
}
```
- this approach is **error-prone**; we could cause the program to panic if the index value or test condition is incorrect.
- It’s also **slow**, because the compiler adds **runtime code to perform the conditional check** of whether the index is within the bounds of the array on every iteration through the loop.
```Rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```
- eliminated the chance of bugs that might result from going beyond the end of the array or not going far enough and missing some items.
- Machine code generated from `for` loops can be more **efficient** as well because the **index doesn’t need to be compared to the length of the array** at every iteration.

>The **safety** and **conciseness** of `for` loops make them the most commonly used loop construct in Rust.

>[!question] 細說這邊的 machine code 為啥不用做邊界檢查? 性能提升多少? 
>當你用 `for element in a` 時 ，會用迭代器 (iterator) 透過指標來運作，因為迭代器本身不可能超出陣列範圍，所以你可以大膽的操作。迭代器不用 `if i >= a.len()` 去做邊界檢查，這避免了分支預測 (避免猜錯的 pipeline flust)，更容易做迴圈展開 (loop unrolling)，可以做 SIMD (編譯器發現這是一段連續且安全的記憶體，可以用 SIMD 指令集，讓 CPU 可以在一個 cycle 同時把多個數字乘二)。

```Rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}
```
- `Range`: provided by the **standard library**, which generates all numbers in sequence starting from one number and **ending before** another number.

>[!question] 為啥這邊的 `Range` 要用 `..` 而不是 `...`?
>在程式中常見「左閉右開」跟「左閉右閉」兩區間，我們需要兩個表示法來說這兩件事
>- `..`: 表「左閉右開」
>- `..=`: 表「左閉右閉」
>這邊因為「左閉右開」常見 (像一般便利集合、數列等)，所以符號少，而「左閉右閉」用等號視覺張力更強。

