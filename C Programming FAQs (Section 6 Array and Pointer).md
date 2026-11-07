# 參考自 [6. Arrays and Pointers](https://c-faq.com/aryptr/index.html)

>[!question] 問題一
>`char a[]` 跟 `char *a` 一樣嗎? 一樣的時機跟不一樣的時機分別在哪?

這兩個的型態是不同的，`char a[]` 的型別是 array of character 而 `char *a` 是 pointer to character。

>[!question] 問題二
>`char a1[]` 跟 `char *a2` 在寫 `a[3]` 時有何異同?




>[!question]
>`int *b1[10]` 跟 `int (*b2)[10]` ~~差在哪~~?









>[!question]
> 二為數組跟 pointer to pointer to type T 一樣嗎?




>[!question]
>將一維函式做為 argument 的一部分傳遞到函式時，該亦為函式的型別從 array-of-type-T 變為 pointer-of-type-T，試問 `int a1[10];` 跟 `int *a2 = malloc(10 * sizeof(int))` 在做為 argument 的一部分時差距在哪，使用上的差別又在哪? 

`int a1[10]` 跟 `int *a2 = malloc(10 * sizeof(int))` 做為 argument 傳遞到函式，其型別確實都為 pointer-of-type-T，所以在做為 argument 時沒有差別。兩者的差別主要在於使用上，該差別來自於儲存的位置不同，如果 `a1` 為區域變數其會存放在 stack 上（static 跟 global 的資料會放在靜態儲存區），而 `a2` 指向存放在 heap 上的資料。`a1` 會增大該函式對於 stack 的需求同時在函式回傳後 `a1` 資料會失效；`a2` 不會增大函式的 stack 用量也不會在函式回傳後被清除，但在用完 `a2` 後要記得手動 `free()` 掉。 