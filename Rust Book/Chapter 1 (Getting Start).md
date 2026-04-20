# Installation
安裝
```shell
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```
- `--proto` (protocol): `--proto '=https'` 限制 `curl` **只能**使用 HTTPS 協定，參數中的等號 `=` 非常重要，代表「完全限定」。
- `--tlsv1.2`: 強制使用 TLS 1.2 或更高版本的加密標準進行連線。
- `-sSf`: 
	- `-s` (`--silent`): 隱藏下載進度條和常規訊息，讓終端機畫面保持乾淨。
	- `-S` (`--show-error`): **顯示錯誤**。搭配 `-s` 使用，雖然隱藏了進度條，但如果發生錯誤（例如網路斷線），依然會印出錯誤訊息讓你除錯。
	- `-f` (`--fail`): 失敗時停止輸出。

檢查
```shell
$ rustc --version
```

更新和刪除
```shell
$ rustup update             # 更新
$ rustup self uninstall     # 刪除
```

# Hello, World!
第一個程式
```rust
fn main() {
    println!("Hello, world!");
}
```

編譯和執行
```rust
$ rustc main.rs
$ ./main
```

>[!question] 為何在 rust 中函式要寫一個 `fn` 來表示? 在 C/C++ 中不用這樣
>先思考一下 C/C++ 中不這樣做會帶來怎樣的壞處。在 C/C++ 中要理解一段程式碼的意思，需要上下文的搭配，也就是語法解析文是上下文無關 (Context-free) 的。舉幾個例子
>```c
>A * B
>```
>可能有以下幾種翻譯方式
>- `A` 跟 `B` 都是之前宣告的變數: 這是一個乘法運算。
>- `A` 是一個型別的別名，而 `B` 在前文未出現: 這是宣告一個指針變數。
>```c
>A (B)
>```
>可能有以下幾種翻譯方式
>- `A` 是一個函式而 `B` 是之前宣告的變數: 這是一個函式呼叫。
>- `A` 是一個型別的別名，而 `B` 在前文未出現: 這是宣告一個變數。
>所以 C/C++ 在做編譯時要維護一個符號表 (Symbol Table) 來紀錄之前遇到的名稱與其對應的型態，並在做語法解析 (parsing, 看懂句子得結構並劃出抽象語法樹 (AST) ) 時跑去符號表看，這使得編譯過程加長，同時變得複雜難以最佳化。
>
>Rust 想要一個上下文無關的語法解析過程，所以引入明確的關鍵字
>- 宣告變數：`let` / `mut` / `const`
>- 宣告結構體：`struct`
>- 宣告列舉：`enum`
>- 宣告特徵：`trait`
>- 宣告函式：`fn`
>
>這樣前述的問題可以獲得改善，對於 `A*B` 來看，在 RUST 他一定代表運算式，如果是宣告的話會寫成
>```rust
>let B: *const A;
>```
>對於 `A (B)` 來看， RUST 依舊需要保持符號表來對應，對在語法解析時不用去符號表看，因為它一定是表達式，不可能是宣告式。語法解析完，才輪到語意檢查 (Semantic Analysis) 去符號表檢查語意。


>[!question] 為何 rust 的 `prinitln!` 要用巨集 (macro) 而不是函式?
>我們想在編譯期作格式和型別安全檢測 (Compile-time Safety)，而 Rust 的 `println!` 巨集會在編譯期解析字串格式做
>- **數量檢查：** 它會檢查大括號 `{}` 的數量是否和後面的變數數量完全一致。
>- **型別檢查：** 它會檢查傳入的變數是否有實作 `Display`（用 `{}` 印出）或 `Debug`（用 `{:?}` 印出）特徵（Trait）
>
>而函式只會在執行期才會知道傳入的參數為何，此時才知道格式和型別是否正確。另一方面 rust 的函式不支援可變數量的參數 (variadic arguments)。


# Hello, Cargo!
Cargo is Rust’s build system and package manager. 
- Building the code
- downloading the library
- building those library

檢查 `cargo` 版本
```shell
$ cargo --version
```

用 `cargo` 建造一個新的專案
```shell
$ cargo new hello_cargo
$ cd hello_cargo
```

建的這個專案預設有
- `.gitignore` file
- `Cargo.toml` file
- `src` directory: 內部有 `main.rs` file

>[!note] Cargo.toml file
>該檔案內容大致為
>```toml
>[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"
[dependencies]
>```
>`package` 用來表示該專案所需要的配置
>`dependencies` 用來羅列這專案所需的依賴庫，在 rust 中該依賴庫又稱為 crates 。這邊沒寫東西表示用 rust 內建的「標準函式庫 (Standard Library)」。
>>TOML 的原文是 Tom's Obvious, Minimal Language 。他是一個配置文件格式，目標是提供一個簡單文法，使得配置項目可以唯一的轉換成一個哈希表，並被個語言解析。
>

### 手動建一個測試用的執行檔並執行
建一個測試用的執行檔
```shell
$ cargo build
```
建維後檔案會在 `target/debug` 中。以這專案為例檔案名是 `hello_cargo` ，所以執行命令是
```Shell
./target/debug/hello_cargo
```
在 `cargo build` 後在這專案的根目錄會出現一個新檔案 `Cargo.lock` ，這檔案用來記錄依賴庫的版本號，以供別人執行時能順利安裝所需的依賴庫。

>[!question] 別人 `git clone` 後如何執行該專案的程式?
>重新編譯一次
>```shell
>$ git clone example.org/someproject
$ cd someproject
$ cargo build
>```

### 自動建一個測試用的執行檔並執行
使用命令
```shell
$ cargo run
```
他的過程如同「手動建一個測試用的執行檔並執行」，只是這命令把建檔跟執行綁在一起。

### 測試能否編譯
用命令
```shell
cargo check
```

>[!question] 為啥我們需要測試能否編譯? 直接編譯不行嗎
>因為編譯要產生執行檔，而測試能否編譯不用，使得測試能否編譯耗時較少。粗略地來看 Rust 編譯分兩步驟
>1. 前端檢查 (front-end): 編譯器會檢查你的語法對不對、型別有沒有寫錯，以及執行 Rust 最著名的**所有權與借用檢查 (Borrow Checking)**。
>2. **後端生成與最佳化 (Back-end & LLVM)**：一旦檢查通過，編譯器會把程式碼交給 LLVM（一個強大的編譯器後端框架）。LLVM 會後將程式碼翻譯成機器碼，並連結（Link）成一個你可以實際執行的執行檔（二進位檔）。
>而 `cargo check` 只執行第一步。

### 建一個發行版的執行檔
使用命令
```rust
$ cargo build --release
```
生成的執行檔會在 `target/release` 。
這編譯時間會比較久，因為他會做最佳化，使得執行檔執行時間下滑。