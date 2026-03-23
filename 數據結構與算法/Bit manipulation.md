## Logtrick 
## 參考自[LogTrick 入门教程](https://zhuanlan.zhihu.com/p/1933215367158830792)

從下面問題出發
>[!question] 問題一
>對於一個給定陣列 nums 來說，我想要輸出一個陣列 modified_nums，其陣列大小跟nums一樣都為n ，modified_nums中的元素都由以下方式定義(陣列是 1-index) $$\text{modified\_nums}[i] = \text{nums}[i] \ | \ \text{nums}[i+1] \ | \ \cdots \ | \ \text{nums}[n]$$ 
>
### 思路一 : 暴力
對於每個 $\text{modified\_nums}[i]$ 都從 $i$ 開始作按位或到結尾，這樣要 $o(n^2)$ 的時間。
### 思路一改 : 暴力+表
仔細想一下你有大量重複的計算，例如在計算$\text{modified\_nums}[i+1]$ 時，要做$OR$ 的數字是在做$\text{modified\_nums}[i]$ 時的子集。觀察一下有以下等式$$\text{modified\_nums}[i] = \text{modified\_nums}[i+1] \ | \ nums[i]$$如果你有$OR$ 的逆運算你便可以從 $\text{modified\_nums}[i]$ 和 $nums[i]$ 推 $\text{modified\_nums}[i+1]$，但細想一下並沒有直接的逆運算(XOR只在XOR時有逆運算)，你直接將 $nums[i]$ 為1的位元位置從 $\text{modified\_nums}[n]$ 拿開會傷及無辜。突圍的觀察是
- 加法具有顯然的逆運算--減法。

我們可以統計整個陣列中每個位元上出現 1 的次數，然後在從前往後遍歷時，將當前元素的位元貢獻從計數中移除。這樣就能在不直接對 OR 進行逆運算的情況下，得到每個位置的累積 OR 結果。
### 思路一改改 : 暴力但是顛倒順序
在**思路一改**中要 $OR$ 的逆運算，但看這式子 $\text{modified\_nums}[i] = \text{modified\_nums}[i+1] \ | \ nums[i]$ ，其實從 $\text{modified\_nums}[i+1]$ 和 $nums[i]$ 推 $\text{modified\_nums}[i]$ 很簡單，只要做一次 $OR$ 運算，所以改變方向從結尾往開頭計算就行。
### 思路二 : logTrick
有一個觀念是想到
>$OR$ 運算相當於對各數值二進制表示中為1的位元位置進行聯集操作。

所以令$\text{modified\_nums}[i]$ 為 $A_i$，這個$A_i$ 可以想成
$$
A_i = a_i \cup a_{i+1} \cup \cdots \cup a_{n}, \text{ where $a_i$ is the bit positions that are set to 1 in the binary representations of $nums[i]$}
$$
所以有以下包含鏈$$A_n \subseteq A_{n-1} \subseteq \cdots \subseteq A_i$$這個包含鏈給我們一個提示是結果具有單調性，這**單調性在處理更新和查詢問題時常常有奇效**
- 更新問題 : 你可能不需要更新全部
- 查詢 : 二分搜
講一下這邊更新的邏輯，出發點是利用包含鏈性值(左邊的集和較大)，這讓右側新加入數值，不一定要更新所以有左邊的集合，只要更新到包含 $a_i$ 的集合就行，因為更左邊的集合自然包含 $a_i$ 。~~這操作最多只要走 $\log(U)$ 步即可， $U$ 代表在數列中最大值在二進制中的長度~~。每一個集合最多更新 $\log(U)$ 次，$U$ 代表在數列中最大值在二進制中的長度，所以總時間為 $O(n \log(U))$ 。

接下來考慮**更麻煩的問題**
>[!question] 問題二
>给你一个由非负整数组成的数组 $a$，和一个非负整数 $k$。设 $b$ 是 $a$ 的一个非空连续子数组。计算 $\left| OR(b) - k \right|$ 的最小值，其中 $OR(b)$ 是 $b$ 中所有元素的按位或。

在這問題可以視為問題一的變體，問題一規定子數列結尾必須是數列結尾，而問題二沒規定子數列結尾在哪，可以是數列的任意位置。
### 思路一 : 暴力
跟問題一類似，只是在往結尾更新時在每一個下標停一下比較$\left| OR(b) - k \right|$ 的值。
### 思路一改 : 暴力+表
依舊是使用 $OR$ 單調的思路，假設一子數列框的區域是另一個子數列框的區域的子集，那他在 $OR$ 運算下的值會越大，這性質讓我們想到一個常見操作 : **滑動窗口**。具體操作是這樣
1. 先固定一個左端點，右端點向右移動直到這子數列的 $OR$ 值大於等於 $k$ ，可以想成跑到**合法**的邊界
2. 移動左端點，直到這子數列的 $OR$ 值小 $k$ 可以想成跑到**非法**的邊界，這邊移動左端點對應的 $OR$ 逆運算，可由**問題一思路一改**所提供。
### ~~思路一改改 : 暴力但是顛倒順序~~
**這方法失效，因為結尾不固定**
### 思路二 : logTrick
直接套**問題一思路二**的思路就行，你在右端加入點時，自然而然會更新在加入點左側的所有子數列。
### 延伸問題 : 那對於and、gcd操作呢 ?
這問題是針對問題二做改編，把 $OR$ 換成 $AND$、$GCD$ ，你算法如何設計? 
>[!question] 問題二改
>给你一个由非负整数组成的数组 $a$，和一个非负整数 $k$。设 $b$ 是 $a$ 的一个非空连续子数组。计算 $\left| AND(b) - k \right|$ 、 $\left| GCD(b) - k \right|$ 的最小值，其中 $AND(b)$ 是 $b$ 中所有元素的按位與，$GCD(b)$ 是 $b$ 中所有元素取最大公因數。
#### AND 操作
有一個觀念是想到
>$AND$運算相當於對各數值二進制表示中為1的位元位置進行並集操作。
1. 直接套上面**問題二思路一改**。
2. Logtrick 也可以用。觀察到AND操作下集合關係變為$$A_{i, j} \subseteq A_{i+1, j} \subseteq \cdots \subseteq A_{j,j}$$如果新加入數值所代表的集合 $a_{j+1}$ 內任一元素不在 $A_{k,j}$ 中那它們也不會再 $A_{l,j}, l \leq k$裡面。這讓我們知道只要更新到 $A_{k,j}$ 就可以停手。
#### GCD操作
 只要想到
 >gcd操作依舊有單調性。說得更嚴謹些，令 $S_{l,k}$ 是原數組中下標為 $l$ 到 $k$ 的子數組，$P(gcd(S_{l,k}))$ 表示 $gcd(S_{l,k})$ 質因數分解後的質數集合，那有以下包含關係成立$$P(gcd(S_{i,j})) \subseteq P(gcd(S_{i+1,j})) \subseteq \cdots \subseteq P(gcd(S_{j,j}))$$

1. 直接套上面**問題二思路一改**，紀錄的事情改為現有質因數個數。
2. 一樣可以用Logtrick，從新加入的右端點向左更新直到要加入的質因數不再集合之中，這跟 $AND$ 操作一致。時間複雜度方面跟之前是一樣，因為質因數最小是2，所以每個集合最多更新 $\log U$ 次。

**最後一個問題**
>[!question] 問題三
>在更新方向的選擇上，為何不統一從左到右，即左端點和右端點更新一致從左到右 ? 
>當前方向是左端點由右到左；右端點由左到右。

兩者想做的事是一致的，都是遍歷所有可能，只是在logtrick的使用下左端點由右到左可以有"剪枝"的功能。有一思想
- **從左到右或從右到左都在遍歷，只是在有單調特性的問題中，可以由較可能影響的子問題開始嘗試，直到沒有關聯的子問題，這樣可以減小更新的次數。**

>[!note] 總結
>Logtrick使用原因，子數列在的運算體系下有單調的特性，我們使用單調性配合更新方向的選擇可以減小更新次數。單調特性的題，常用[[Sliding window]] 和 [[Multi-pointers]] 的技巧，這時要好好想一下**左端點移動時對應的逆運算**。

---
# 貢獻法
## Motivation

>[!question] 問題一
>針對一個數列，若要計算其在特定子集組合下的**按位 XOR、AND、OR** 等運算結果，是否存在一種方法，能夠在不窮舉所有可能組合的情況下，推導出目標的統計性質？

首先給一個觀察
>[!observation] 觀察一
>由於 **XOR、AND、OR** 等按位運算具有**位獨立性**，即每個位元的運算結果僅取決於運算元對應的位元，因此原始問題可拆解為一系列獨立的**單一位元子問題**來處理。

所以問題一可以改成
>[!question] 問題一
>針對一個數列的某一個特定位元，若要計算其在特定子集組合下的**按位 XOR、AND、OR** 等運算結果，是否存在一種方法，能夠在不窮舉所有可能組合的情況下，推導出目標的統計性質？

問題一說的太籠統了，有兩個關鍵點沒說
>[!question] 問題一改
>1. 特定子集組合是哪種 ? 
>2. 對於特定子集組合，什麼性質不需要窮舉所有可能組和 ?
## Traverse & Property
>[!important] 假設
>假設數組大小為 $m$
### Traverse (all paring)
相關的題目有
- [477. Total Hamming Distance](https://leetcode.com/problems/total-hamming-distance/)
- [2425. Bitwise XOR of All Pairings](https://leetcode.com/problems/bitwise-xor-of-all-pairings/)
- [1835. Find XOR Sum of All Pairs Bitwise AND](https://leetcode.com/problems/find-xor-sum-of-all-pairs-bitwise-and/)
- [3153. Sum of Digit Differences of All Pairs](https://leetcode.com/problems/sum-of-digit-differences-of-all-pairs/)
對於這對於這組合方式，我們可以將pairing想成自己和其他人只配對一次。在觀察一中我們已將問題簡化在一個位元上做討論，而一個位元只有 $0, 1$ 兩個可能，我們可以分開來討論，所以題目簡化為
>[!question] 問題二
>假設自己這個數在某個位元為 $0, 1$ ，他跟其他人互動對於結果的影響為和 ?

如果( $a$ )和其他人( $b_1, b_2, \cdots, b_n$ ) 互動可以看成 $a$ 和 $b_1$, $a$ 和 $b_2$, ..., $a$ 和 $b_n$ 的互動，**彼此之間的結果互不影響**，那可以直接統計會影響性質的數量就行。
### Traverse (Combination)
相關的題目有
- [2275. Largest Combination With Bitwise AND Greater Than Zero](https://leetcode.com/problems/largest-combination-with-bitwise-and-greater-than-zero/)
- [1863. Sum of All Subset XOR Totals](https://leetcode.com/problems/sum-of-all-subset-xor-totals/)
可以視為上一種遍歷的擴展，一樣可以想成自己和其他人，只是其他人可出現或不出現，總共有 $2^{m-1}$ 種可能。一樣一個位元只有 $0, 1$ 兩個可能，我們可以分開來討論。最後如果**子集之間不互相影響**，我們可以分開討論，統計相關數量就好。


>[!note] 總結
>上述兩總遍歷方式都可以看作自己和其他人的互動，又各個**子集之間不互相影響**，所以可以直接統計跟結果有關的數量就行。

---
# lowbit

>[!question] 問題
>給定一個`int x`，如何快速輸出最低位的 1 ?
>舉個例子 : `int x = 40` ，你要輸出 `1000` ，因為 $40_{10} = 101000_2$。

假設 $x = 110100$ 那$$x= 110100 \Rightarrow \sim x = 001011 \Rightarrow \sim x + 1 = 001100 \Rightarrow x \& (\sim x + 1) = 000100$$ 整體思路是這樣
>在 $\sim x$時，$x$最小位的 $1$ 右邊的 $0$ 會變 $1$ ，之後加一將這些 $1$ 變成 $0$ ，並把因為**Bitwise NOT**操作變 $0$ 的 $x$最小位的 $1$ 還原成 $1$。至於 $x$最小位的 $1$ 左邊的位元會因為跟**Bitwise NOT** 做 **Bitwise AND** 而變為 $0$，注意$\sim x + 1$的進位不會影響到$x$最小位的 $1$ 左邊的位元。

---
# Xor 可以視為模2運算
---

