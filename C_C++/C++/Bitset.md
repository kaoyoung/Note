## 參考自 [【C++ 筆記】bitset 容器，C++ 二進位運算的絕佳利器](https://hackmd.io/@LukeTseng/rJLYXKXRxx#%E5%AD%98%E5%8F%96-bitset-%E7%9A%84%E5%96%AE%E4%B8%80%E4%BD%8D%E5%85%83)
# 動機
在計算機中因為需要記憶體對齊，所以一個 `bool` 變數至少需要 `1 byte` 的空間，這對空間的消耗是所需的8倍。從下面這程式可以看出來
```C
#include <stdio.h>
#include <stdbool.h>

int main(void) {
	// your code goes here
	bool record[1000];
	printf("%zu\n", sizeof(record));
	return 0;
}
```
輸出為
```text
1000
```
那我們如何避免過多的記憶體浪費 ?
一個思路是直接對一個數字做操作，利用數字會用二進制儲存跟位運算(`XOR`、`AND`、`OR`)這兩特性來實現，那這有一問題是: 數字最多為64 bit (假設在64位元的操作系統)，如何操作多於64 `bool` 變數的情況 ? 答: 用陣列存儲數字。

---
# 簡介
`std::bitset<N>` 是 C++ STL（位於標頭檔 `<bitset>`）中，一個用來表示固定長度為
$N$ 位元（bits）的位元序列（bit sequence）的類別模版。bitset 也是 C++ 當中的一種**容器**，這個容器能夠做到單獨、有效的去操作每一個位元，也能解決位元運算、實現二進位表示法的工具，這也是為什麼需要用到它的原因。

---
# 語法
### 初始化
1. `std::bitset <10> a(5);`
輸出 : `0000000101`
2. `std::bitset <10> a("101");`
輸出 : `0000000101`
3. `std::bitset <10> a;`
輸出 : `0000000000`
### 存取單一位元
```C++
#include <iostream> 
#include <bitset> 
int main(){ 
	std::bitset <6> a("101"); 
	std::cout << a[0] << std::endl; 
	std::cout << a[5] << std::endl; 
	return 0; 
}
```
輸出為 : 
```text
1
0
```

>`bitset` 的下標從最右邊開始算。
### 常用函數
-  `set()`：設定位元為 1。
```C++
#include <iostream>
#include <bitset>

int main(){
	std::bitset <6> a(32); // 32 = 100000 
	a.set(); // 設定所有位元為 1 
	a.set(2); // 設定第 2 位為 1 
	a.set(2, 0); // 設定第 2 位為 0 
	std::cout << a; 
	return 0;
}
```
輸出為:
```text
111011
```

- `reset()`：重置位元為 0。
```C++
#include <iostream>
#include <bitset>

int main(){
    std::bitset <6> a(32); // 32 = 100000
    a.reset();      // 所有位元設為 0
    std::cout << "位元重置：" << a << std::endl;
    a.set(); // 所有位元設為 1
    a.reset(3);     // 第 3 位設為 0
    std::cout << "第三位是 0：" << a << std::endl;
    return 0;
}
```
輸出為:
```text
位元重置：000000
第三位是 0：110111
```

- `flip()`：翻轉位元（0 變 1，1 變 0）。
```C++
#include <iostream>
#include <bitset>

int main(){
    std::bitset <6> a(32); // 32 = 100000
    a.flip();       // 翻轉所有位元 -> 011111
    a.flip(1);      // 翻轉第 32 位 -> 011101
    std::cout << a;
    return 0;
}
```
輸出為:
```text
011101
```

- `test()`：測試某位元是否為 1。
```C++
#include <iostream>
#include <bitset>

int main(){
    std::bitset <6> a(32); // 32 = 100000
    std::cout << "第一位是 1 嗎？" << (a.test(0) ? "Yes" : "No");
    return 0;
}
```
輸出為
```text
第一位是 1 嗎？No
```
- `count()`：計算有多少個 1。
- `any()`：是否有任何位元為 1。
- `all()`：是否所有位元都為 1。
- `none()`：是否沒1。
- `size()`：回傳 bitset 的大小。
- `to_string()`：轉成字串。
- `to_ulong()`：轉成 unsigned long。
- `to_ullong()`：轉成 unsigned long long。
```C++
#include <iostream> 
#include <bitset> 

int main(){ 
	std::bitset <6> a(32); // 32 = 100000 
	unsigned long long val = a.to_ullong(); 
	std::cout << val; 
	return 0; 
}
```
輸出為
```text
32
```

### 支援的位元運算子
- `&`：AND
- `|`：OR
- `^`：XOR
- `~`：NOT
- `>>=`：右移賦值運算子。
- `<<=`：左移賦值運算子。
- `&=`：AND 賦值運算子。
- `|=`：OR 賦值運算子。
- `^=`：XOR 賦值運算子。
```C++
#include <iostream>
#include <bitset>

int main(){
    std::bitset<8> b1("10101010");
    std::bitset<8> b2("11110000");
    std::bitset<8> result = b1 & b2;
    std::cout << result;
    return 0;
}
```
輸出為
```text
10100000
```

---
# 實作參考
-  `set()`：設定位元為 1。
`a.set()`: 可以用 `a = a | (~0);`
`a.set(k)`: 可以用 `a = a | (1 << k);`
`a.set(k, 0)`: 可以用 `a = a & (~(1 << k));`

- `reset()`：重置位元為 0。
`a.reset()`: 可以用 `a = a & 0;`
`a.reset(k)`: 可以用 `a = a & (~(1 << k));`

- `flip()`：翻轉位元（0 變 1，1 變 0）。
`a.flip()` :   可以用 `a = a ^ (~0);`
`a.flip(k)` : 可以用 `a = a ^ (1 << k);`

- `test()`：測試某位元是否為 1。
`a.test(k)`: 可以用 `return (a & (1ULL << k)) != 0;`

- `count()`：計算有多少個 1。
`a.count()` : 可以用 `return __builtin_popcount(a);`