### 參考自 [The Levi-Civita Symbol](https://people.uncw.edu/hermanr/qm/Levi_Civita.pdf)
# 動機
在一般作外積運算時，可以用行列式或是把該向量猜成彼此獨立的向量再做乘法，但這稍嫌麻煩，有無更快的運算 ? 
我們先想，在三維空間中用右手定則可以知道
- $x$ 軸和 $y$ 軸外積是 $z$ 軸
- $y$ 軸和 $z$ 軸外積是 $x$ 軸
- $z$ 軸和 $x$ 軸外積是 $y$ 軸
這可以看到 $x$ 、$y$、$z$ 形成一個cyclic group 。同時
- $x$ 軸和 $z$ 軸外積是 $y$ 軸的反向
- $y$ 軸和 $x$ 軸外積是 $z$ 軸的反向
- $z$ 軸和 $y$ 軸外積是 $x$ 軸的反向
$z$ 、$y$、$x$ 形成一個cyclic group。所以我們定義 levi-civita symbol 來說明這件事$$\begin{align} \epsilon_{ijk} &= \epsilon_{jki} = \epsilon_{kij} = 1 \\ \epsilon_{ikj} &= \epsilon_{jik} = \epsilon_{kji} = -1 \end{align}$$其餘由 $\{i,j,k\}$ 這集合中可重複選元素的排列皆為0，這對應到若兩向量相同則其外積為 0 。

這如何簡化表達呢 ? 
1. 表達 $\hat{e_i} \times \hat{e_j}$，假設 $\hat{e_1} = \hat{i}, \hat{e_2} = \hat{j}, \hat{e_3} = \hat{k}$
$\hat{e_i} \times \hat{e_j}$ 可以寫成 $$\hat{e_i} \times \hat{e_j} = \sum^3_{k=1} \epsilon_{ijk} \hat{e_k}$$
2. 表達 $\hat{u} \times \hat{v}$
$\hat{u} \times \hat{v}$ 可以寫成 $$\hat{u} \times \hat{v} = \sum^3_{i=1} \sum^3_{i=1}u_i v_j \hat{e_i} \times \hat{e_j} = \sum^3_{i=1} \sum^3_{i=1}u_i v_j \sum^3_{k=1} \left( \epsilon_{ijk} \hat{e_k} \right) = \sum^3_{i,j,k=1} \epsilon_{ijk} u_i v_j  \hat{e_k} $$
---
# 性質


---
# 例子
>[!example] 例子一
>物理學中的 BAC-CAB rule ，這規則是在說一個等式 $$\hat{a} \times \left( \hat{b} \times \hat{c} \right) = \hat{b} \left( \hat{a} \cdot \hat{c} \right) - \hat{c} \left( \hat{a} \cdot \hat{b} \right)$$

