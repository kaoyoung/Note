```Rust
use std::io;

fn main() {
    println!("Guess the number!");
    println!("Please input your guess.");
    let mut guess = String::new();
    
    io::stdin()
	    .read_line(&mut guess)
	    .expect("Failed to read line");
    
    println!("You guessed: {guess}");
}
```

讓我們逐行解析
```Rust
use std::io
```
- If a type you want to use isn’t in the prelude, you have to bring that type into scope explicitly with a `use` statement.
>**prelude**: Rust has a set of items defined in the standard library that it brings into the scope of every program. This set is called the prelude.

- This code contains a lot of information, so let’s go over it line by line. To obtain user input and then print the result as output, we need to bring the `io` input/output library into scope.

```Rust
let mut guess = String::new();
```
- The equal sign (`=`) tells Rust we want to bind something to the variable now.
- `String::new`: a function that returns a new instance of a `String`
- `::`: indicates that `new` is an associated function of the `String` TYPE
>**Associated function**: It is a function that's implemented on a type, in this case `String`.

```Rust
io::stdin()
	.read_line(&mut guess)
	.expect("Failed to read line");
```
- If we hadn’t imported the `io` module with `use std::io;` at the beginning of the program, we could still use the function by writing this function call as `std::io::stdin`.
- `.read_line(&mut guess)` calls the `read_line` method on the standard input handle to get input from the user.
- We’re also passing `&mut guess` as the argument to `read_line` to tell it what string to store the user input in.
- The `&` indicates that this argument is a _reference_, which gives you a way to let multiple parts of your code access one piece of data without needing to copy that data into memory multiple times.
-  references are immutable by default. Hence, you need to write `&mut guess` rather than `&guess` to make it mutable.
- `read_line` puts whatever the user enters into the string we pass to it, but it also returns a `Result` value. `Result` is an enumeration.
	- `Result`’s variants are `Ok` and `Err`. 
- If this instance of `Result` is an `Err` value, `expect` will cause the program to crash and display the message that you passed as an argument to `expect`.

```Rust
println!("You guessed: {guess}");
```
- The `{}` set of curly brackets is a placeholder: Think of `{}` as little crab pincers that hold a value in place.
- When printing the value of a **variable**, the variable name can go **inside the curly brackets**.

```Rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```
-  When printing the result of evaluating an **expression**, place empty curly brackets in the format string, then follow the format string with a **comma-separated list** of expressions to print in each empty curly bracket placeholder in the same order.

## Generating a Secret Number
Rust doesn’t yet include random number functionality in its standard library. However, the Rust team does provide a `rand` crate with said functionality.
>[!question] 為啥 Rust 不把 `random` 放入標準庫?  C/C++ 都放了
>把一個函式放入標準庫要很嚴謹，因為標準庫通常要保證向後兼容 (backward compability)，而 `random` 目前還沒有大一統的實作方式，同時兼顧速度和隨機，目前該領域還在大步向前。還有一個理由是 Rust 的 `std` 刻意保持精簡，只選擇**「必須由語言/標準庫保證統一抽象的功能」** (例如: `Vec`、`Hashmap`、執行緒、I/O 等)，而亂數涉及**平臺相關的熵源（getrandom）、演算法選擇、效能與安全性的取捨**，這類高度變動且多樣化的需求更適合由生態系統來競爭、演化。Rust 應對 `random` 的使用，提供 C/C++ 沒有的統一第三方套件管理 (Cargo) 來提供 `random` 函數的。第三方函數的提供還可以細分賽道，亂數在不同領域要求不同
>- 遊戲: 追求速度
>- 蒙地卡羅模擬: 均勻分布與可重現性
>- 安全: 不可預測和密碼學安全

>[!question] 為何標準庫不允許 `random`  但允許同樣要隨機的 `Hashmap` ?
>`HashMap` 被留在標準庫中，最主要的原因是**生態系的互通性 (Interoperability)**。因為大部分應用程式都需要鍵值對 (key-value) 資料結構，所以需要一個大家都共同遵守的 API 介面，而 Rust 提供該介面，但基本算法可以用第三方套件。至於`Hashmap` 內原本預設的實作是偷偷靠一個極簡、私有的機制像 OS 索要隨機數。
### Increasing Functionality with a Crate
去 Cargo.toml 改
```toml
[dependencies]
rand = "0.8.5"
```
- 把 `rand` crate 加入 dependency
- When we include an external dependency, Cargo fetches the latest versions of everything that dependency needs from the _registry_, which is a copy of data from Crates.io.
>**Crates.io**: Crates.io is where people in the Rust ecosystem post their open source Rust projects for others to use.

### Updating a Crate to Get a New version
```shell
$ cargo update
```
- Cargo provides the command `update`, which will ignore the _Cargo.lock_ file and figure out all the latest versions that fit your specifications in _Cargo.toml_.
	- Cargo will then write those versions to the _Cargo.lock_ file.
>**_Cargo.lock_ file**: It's the file that generated when you first build the project. This file makes the dependencies of your program consistent before you update the dependency. 

### Generating a Random Number
```Rust
use std::io;

use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```
這邊主要新增兩行
```Rust
use rand::Rng;
```
-  The `Rng` trait defines methods that random number generators implement, and this trait must be in scope for us to use those methods.
```Rust
let secret_number = rand::thread_rng().gen_range(1..=100);
```
- `rand::thread_rng()` function that gives us the particular **random number generator** we’re going to use: one that is local to the current thread of execution and is seeded by the operating system.
- `gen_range()`: takes a range expression as an argument and generates a random number in the range.

### Comparing the Guess to the Secret Number
```Rust
use std::cmp::Ordering;
use std::io;

use rand::Rng;

fn main() {
    // --snip--
    
    let guess: u32 = guess.trim().parse().expect("Please type a number!");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```
理解一下
```Rust
use std::cmp::Ordering
```
-  `Ordering` type is another enum and has the variants `Less`, `Greater`, and `Equal`.
```Rust
match guess.cmp(&secret_number) { 
	Ordering::Less => println!("Too small!"), 
	Ordering::Greater => println!("Too big!"), 
	Ordering::Equal => println!("You win!"), 
}
```
- A `match` expression is made up of _arms_. An arm consists of a _pattern_ to match against, and the code that should be run if the value given to `match` fits that arm’s pattern.
- When the code compares 50 to 38, the `cmp` method will return `Ordering::Greater` because 50 is greater than 38.
```Rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```
- The `trim` method on a `String` instance will eliminate any whitespace at the beginning and end
- The `trim` method eliminates `\n` or `\r\n`
- The `parse` method on strings converts a string to another type.
- The colon (`:`) after `guess` tells Rust we’ll annotate the variable’s type.

>[!question] 為啥這邊的 `secret_number` 要用 reference?
>這邊的 `&` 跟 C++ 一樣都是給記憶體位置，但 Rust 保證該記憶體安全且預設不可變 (可變要用 `&mut`)，這邊安全保證是把 `＆` 當成不可變的借用，而在任一時間內同一筆資料只能有「一個可變借用」或「多個不可變借用」，這確保不會發生 race condition 跟懸空參考 (Dangling Reference) 的問題。

>[!question] 為啥 `match` 的最後一項也要逗號?
>其實你也可以不加，但這是一個叫尾隨逗號 (trailing comma) 的習慣，方便之後在加它後面東西。

### Allowing Multiple Guesses with Looping
```Rust
use std::cmp::Ordering;
use std::io;

use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```
這邊看一下
```Rust
let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
```
- The undersocore of `Err(_)`: It means a catch-all value

>[!question] 這邊 `Ok(num)` 的 `num` 代表啥?
>`Ok` 是 Rust 規定的標籤，用來說明操作成功，而 `num` 是成功完成 `match guess.trim().parse()` 得到的變數，而該變數名不一定要是 `num` 你隨意。

>[!question] 為啥這邊 `match` 的 `Ordering::Equal` 最後不加逗號?
>因為大括號 `{}` 的右括號已明確結束位置，正如
>```Rust
>if x > 3 {
>}
>```
>if 條件句結束也不會加分號。而
>```Rust
>Ordering::Less => println!("Too small!"),
>```
>如果不加逗號，編譯器不知到哪裡結束，例如
>```Rust
>match guess.cmp(&secret_number) {
>    Ordering::Less => "too small".to_string() // 把字串轉成 String 型別
>        .to_uppercase()                       // 轉成大寫
>        .replace("TOO", "VERY"),              // 把 TOO 替換成 VERY
>
>    Ordering::Greater => "too big".to_string(),
>    Ordering::Equal => "you win".to_string(),
>}
>```

