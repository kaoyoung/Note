# Variable and Mutability

```rust
let x = 5; // 預設不可變，安全且靜態 
let mut y = 5; // 明確宣告可變，y 可以被修改
```

>[!question] 為啥 rust 變數預設為不可變 (immutable)?
>在 c/c++ 中，變數如果沒用 `const` 來設定的話，其值可以不停變化，導致我們可能會不小心改到，或是在多線程中有 race condition 的問題。 rust 為了讓你在用可變變數時多想一下，而把變數預設為不可變 (immutable)，不可變變數還可以讓編譯器大膽的最佳化。

```rust
let guess = "42"; // 第一個 guess 是字串
let guess: u32 = guess.parse().expect("Not a number!"); // 第二個 guess 是數字，完美遮蔽了第一個
```

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

>[!question] c/c++ 不能在同一個 scope 用 shadowing ，為啥更安全的 rust 要用這機制?
>

>[!question] 不同型別的變數用不同名稱不是更安全嗎?

>[!question] 這邊 rust 的思想是「一個資料在一個時間點只有一個型別重要」，那如果同時要用兩個以上的型別呢?

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

>[!question] let 的變數跟 const 差在哪? 不是都是不可變

# Data Types