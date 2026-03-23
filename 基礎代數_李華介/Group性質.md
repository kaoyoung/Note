### 參考自李華介的大學基礎代數
# 動機
在處理數學結構時，我們似乎可以將**數和運算**組合在一起，構成一個數學結構，即便數或運算不一樣，但只要數在該運算下，能有相同的性質，那我們便把他們歸類在一塊。
# 定義
>[!definition] definition of group
>A **group** is a set $G$ equipped with a binary operation $* : G \times G \to G$ satisfying the following axioms
>1. **Closure** : if $a, b \in G$, then $a * b \in G$
>2. **Associativity**: $a*(b*c) = (a*b)*c, \forall a,b,c \in G$
>3. **Identity**: There is an element $e \in G$  such that $a*e = e*a = a, \forall a \in G$
>4. **Inverse**: For each element $a \in G$, there is an element $b \in G$  such that $a*b = b*a = e$
## 細想定義
### Closure
他想要表達：「在該集合內的元素，不管過幾次運算，依舊在該集合內。」，這讓我們在討論運算時不用擔心數值突然跑到一開始給定的範圍 $G$ 外，這讓運算次數不會出問題。
**舉個不為closure的例子 :**
>Take $G = \mathbb{Z}^{-}$ and $* := \times$ then $-1 \in G$ but $$-1 \times -1 = 1 \notin G$$
### Associativity (結合律)
這說明在一串運算中，運算的先後順序不會是重點，如果沒這性質n個數做運算，例如$$a_1 * a_2 * \cdots * a_n$$你必須要定義好$n-1$個運算先後順序(少一是因為最後剩$a_i * a_j$ 這不用定義順序因為只有一種算法)。這性質保證**運算先後順序不重要**。

>[!question] Question 1: Verifying the Associative Property for Addition and Multiplication
>下面等式的左右兩端有何不同$$ 5 - 3 - 2 = 5 + (-3) + (-2) $$
>左式**不具**有結合律$$ 5 - (3 - 2) = 4 \neq 0 = (5 - 3) - 2$$
>右式**具有**結合律$$ 5 + ((-3) + (-2)) = 0 =  (5 + (-3)) + (-2)$$
>從上面這例子可以看到，在整數的運算中用**"加上一個負數"**來取代**"減號"**在結合律上是更好的方法，這使得在做一連串減法時，可將其轉化為一連串負數的加法，從而實現平行運算，想一下如果運算順序重要的話就不能平行去考慮了(也就是減法不能平行去運算)，你可能會想$$10-8-5-2$$可以拆成 $10-8$ 跟 $-5-2$ 但此時你偷偷把減法轉成加上負數了。**乘法配合分數來取代除法也有一樣性質(不考慮0)**。加上一個負數跟乘法配合分數可行，是因為加法在整數中反元素存在，乘法在有理數(不考慮0)中反元素存在。

>[!question] Question 2: Verifying the Associative Property for Cross Product 
>外積不具有結合律，看以下兩等式 $$\begin{align} \vec{a} \times (\vec{b} \times \vec{c}) = \mathbf{\vec{b}}(\vec{a} \cdot \vec{c}) - \mathbf{\vec{c}}(\vec{a} \cdot \vec{b})\\ (\vec{a} \times \vec{b}) \times \vec{c} = \mathbf{\vec{b}}(\vec{a} \cdot \vec{c}) - \mathbf{\vec{a}}(\vec{b} \cdot \vec{c})\end{align}$$從上面兩個式子可以看到除非 $$\mathbf{\vec{c}}(\vec{a} \cdot \vec{b}) = \mathbf{\vec{a}}(\vec{b} \cdot \vec{c})$$ 不然 $\vec{a} \times (\vec{b} \times \vec{c}) \neq (\vec{a} \times \vec{b}) \times \vec{c}$。注 : $\vec{a} \times (\vec{b} \times \vec{c}) , (\vec{a} \times \vec{b}) \times \vec{c}$ 請參考 [[Levi-Civita Symbol]]

### Identity
有單位元素( $e$ )給了我們**維持現狀**的操作，進而可以定義出反元素，除了引出反元素還可以
1. 群同態的核（kernel）
2. 唯一的冪等元（idempotent element）: 即 $e^2 = e$
3. 平凡子群（trivial subgroup）
### Inverse
反元素( inverse element )在 $G$ 中任一元素都存在說明任何操作可以被**復原**，這在解方程時很有用。 

>[!note] 對於 identity 跟 inverse 定義可以簡化
>假設結合律成立，那 Identity 定義只需要 $a*e = a$  同時 Inverse 定義只要有 $a*b = e$ 就行；或是 Identity 定義只需要 $e*a = a$  同時 Inverse 定義只要有 $b*a = e$ 就行。

>[!proof] 簡化定義的證明
>這邊只說明:「Identity 定義只需要 $a*e = a$ 同時 Inverse 定義只要有 $a*b=e$ 就行」。先證明左反元素存在，$$b*a = b*a*e = b*a*(b*c) = b*e*c = b*c = e \Rightarrow b*a = e, \text{ where $c$ is the inverse of $b$}$$ ，最後說明左單位元素存在 $$ e*a = a*b*a = a*e = a \Rightarrow e*a = a$$
>

>[!important] 如果同時只有左反元素和右單位元素，失去群的性質 ! 同理只有右反元素和左單位元素，失去群的性質
> 造一個反例吧，說明只有左反元素和右單位元素的情況，但它不為群。對於所有 $a,b \in G$ 我們定義運算為 $$a*b = a$$即兩數做運算輸出左邊那個，那 closure 一定成立，associativity 呢 ? $$\begin{align} a * (b * c) &= a * b = a \\ (a*b)*c &= a*c = a \end{align}$$下一個考慮左反元素為 $$e*a = e, \forall a \in G$$ 下一個考慮右單位元素 $$c*a = a, \forall a,c \in G$$按照這樣的造法右反元素不一定存在除非 $a = e$，但是按這造法 $\vert G \vert$ 可以大於一，舉取 $G:= \mathbb{N}$



