# C/C++
順序為:
1. 後綴 (suffix/postfix): `++`、`--` ；`()`、`[]`、`.`、`->`
2. 一元 (unary): `++a`、`--a`、`!`、`~`、`*`、`&`、`sizeof`；型別轉換
3. 乘除
4. 加減
5. 位元位移: `>>`、`<<`
6. 關係運算子 (relational operators): `<`、`<=`、`>`、`>=`
7. 相等運算子 (equality operator): `==`、`!=`
8.  `&`
9. `^`
10. `|`
11. `&&`
12. `||`
13. `?:`

先說一些顯然的順序。第三跟四符合數學直覺，第八到第十符合數學直覺 (把 `&` 想成乘，把 `|` 想成加)。後綴跟一元可以被視為對一個元素的先行處理，等處理後再做運算，所以他優先級高。

>[!question] 問題一
>為什麼一樣是位元運算 `>>`、`<<` 的優先級大於 `&`、`^`、`|` ?

一個原因是繼承 B/BCPL 語言的規則。一個間接理由是，位移運算類似於乘除法運算，所以我們把他排在四則運算後面。還有一個常見用法是在做「先算出位元位置、再做遮罩」時可以直接寫 `flags & (1 << n)`。

>[!question] 問題二
>為什麼 `&`、`^`、`|` 設計在 relational operators 跟 equality operator 之後 ?

可以參考 Dennis Ritchie 的這篇文章 [Operator precedence](https://www.lysator.liu.se/c/dmr-on-or.html)。早期的 C 語言繼承了 B/BCPL 語言的想法，在需要 boolean 值的地方 (`if`、`while` ...) 把 `&`、`|` 解釋為現代的 `&&`、`||`，因此有一個經典的寫法
```C
if(a == b & c == d)   // if(a == b & c == d)
```
在這寫法下必須讓  `&`、`|` 優先值低於相等運算子 (equality operator): `==`、`!=`。這是個歷史遺留問題，如果改了向後兼容性 (downward compatibility) 會失去，要改大量的 code。在現代不要寫 `if(a == b & c == d)` 該用`&&`、`||` 就用。

>[!question] 問題三
>這程式 `if(a < b < c)` 的判斷邏輯是啥 ? 

他會先判斷完 `a < b` 並把它的結果 (`x`) 跟 `c` 判斷，及判斷 `x < c`。整個過程是 `((a < b) < c)`，而會有這樣的過程是 relational operator 是左結合 (left associative)。

---
# Python
順序為:
1.  `(...)`、`[...]`、`{...}`                       # 括號、list/dict/set 顯示
2.  `x[i]`、`x[i:j]`、`x(...)`、`x.attr`             # 下標、切片、呼叫、屬性
3.  `await x`
4.  `**`                                       # 次方
5.  `+x`、`-x`、`~x`                               # 一元正負、位元 NOT
6.  `*`、`@`、`/`、`//`、`%`
7.  `+`、`-`
8.  `<<`、`>>`
9.  `&`
10. `^`
11. `|`
12. `in`、`not in`、`is`、`is not`、`<`、`<=`、`>`、`>=`、`!=`、`==`   # 比較(全部同一層)
13. `not x`
14. `and`
15. `or`
16. `x if c else y`                            # 條件運算式
17. `lambda`
18. `:=`                                        # walrus

先說一些顯然的順序。第六跟七符合數學直覺，第九到第十一符合數學直覺 (把 `&` 想成乘，把 `|` 想成加)。下標、呼叫、屬性存取與一元運算可視為對運算元的先行處理，等處理後再做運算，所以他優先級高。

>[!question] 問題一
>為什麼 `&`、`^`、`|` 優先級設計在 relational operators 跟 equality operator 之上，跟 C/C++ 不同 ?

因為 python 設計在 C/C++ 之後並採用 Ritchie 心中所認為理想的規則。

>[!question] 問題二
>這程式 `if(a < b < c)` 的判斷邏輯是啥 ? 

他等價於「先算 `a < b`,為真才接著算 `b < c`,且 `b` 只算一次」` ，因為它支援連鎖比較 (chained comparision)，所以 `if(a < b < c)` 可以理解成 `if(a < b and b < c)`。在規格書 6.10 (Comparisons) 中寫道 "Comparisons can be chained arbitrarily, e.g., `x < y <= z` is equivalent to `x < y and y <= z`, except that `y` is evaluated only once (but in both cases `z` is not evaluated at all when `x < y` is found to be false)."