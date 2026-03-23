## 核心想法
雙指針框住的地方是可能解空間。

## 相向雙指針

## 同向雙指針


## 何時雙向、何時同向
 從[167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/) 引發的思考。
**指針移動的本質** : 指針移動過去後，便不再訪問，所以可視為丟棄的順序。丟棄順序依賴於題目本身的條件和指針開始位置的選擇。
對於問題167.原本想用[[Sliding window]]中的模板去思考，模板如下
```c
int right = 1;
for(int i=1; i<n; ++i){
	for(int j=right; j<=n; ++j){
		if(test(right)){ // test the input is valid or not
			// update max or min number
			// ++right;
		}
	}
}
```
但卡在兩件事上
1. right可以往回縮，因為left(模板的i)增大，right可以縮小。
2. 如果兩數加起來的值小於目標，似乎左右指標增加都行。
先回答第一個問題。原本以為的循環順序如下
- 右衝→左縮→右縮→左縮→...
但其實可以把右衝這個過程直接放在尾端，畢竟只是多探索一些地方，不影響結果，整個過程變為
- 左縮→右縮→左縮→右縮→...
第二個問題，其實只要想清楚為何右指標向左移的原因即可。右指標左移表達右指標右邊的值都不行，究其原因是左指標的值只會遞增。




---
## 常見問題延伸
### 雙指針加二分搜
雙指針框住的地方是可能解空間，再加上用到指針的問題本身具有單調性，使得二分搜有施展空間，所以可以在雙指針的範圍內配合二分搜快速定位需要的位置。
### 性質是在兩元素之間
- [2765. 最长交替子数组](https://leetcode.cn/problems/longest-alternating-subarray/)
- [978. 最长湍流子数组](https://leetcode.cn/problems/longest-turbulent-subarray/)
因為性質是在兩元素之間，所以左右指針要框的範圍似乎是在元素之間，但一般迭代的東西是元素，這使得左右指針似乎難以施展或是要想多個邊界測試。其實可以只用元素的迭代實現性質在兩元素之間的雙指針操做，跟之前的模板一樣，進迴圈後，讓右指針一路判到不符合的元素再更新所求的值，之後進迴圈到下一輪，只是在算區間時要讓該環圈的start減一，已達成兩元素之間的要求。
1. 一般模板
```c++
int maxTurbulenceSize(vector<int>& arr) {
       int answer = 0;
       int index = 0;
       while(index < (int)arr.size()){
	       //紀錄開始位置
            int start = index++;
            
            // 右指針向右看性質，直到不符合
            while(index < (int)arr.size() &&  /*arr.at(index)-arr.at(index-1) < 0*/){
                ++index;
            }
			
			// 跑完右指針後更新答案
            answer = max(answer, index-start);
       }

       return answer;

    }
```
1. 性質是在兩元素之間模板
```c++
int maxTurbulenceSize(vector<int>& arr) {
       int answer = 0;
       
       //求的是元素之間的性質所以從1開始
       int index = 1;
       while(index < (int)arr.size()){
	       //紀錄開始位置，但求的是元素之間的性質，所以開始位置減1
            int start = index-1;
            ++index;
            // 右指針向右看性質，直到不符合
            while(index < (int)arr.size() && /*(long long)(arr.at(index)-arr.at(index-1))*(arr.at(index-1)-arr.at(index-2)) < 0*/){
                ++index;
            }
			
			// 跑完右指針後更新答案
            answer = max(answer, index-start);
       }

       return answer;

    }
```




## 延伸
[[區間邊界策略分析：左閉右閉與左閉右開之比較]]