## Section 1A ($\mathbb{R}^n$ and $\mathbb{C}^n$)

>[!Note] 概念一
>用乘法反元素跟加法反元素來處理減法跟除法

與其多去定義減法或是除法，不如在現有的加法和乘法體系中利用 $a-a=0, a*\frac{1}{a} = 1$ 的想法來定義減法和除法，如此一來我們可以少想兩個運算子。其實只在加法和乘法中做運算有一好處是，這兩運算有交換律和結合律，我們可以不用在乎運算順序，使得平行化運算可以走起來，但是如果引進減法或是除法就喪失這好處。

>[!Note] 概念二
>list,tuple 跟 set 的差別

list 在乎順序和重複，而 set 不在乎，例如:
- $\{1,1\}$ 跟 $\{1\}$ 在 list 中不一樣，而 set 中相同
- $\{1,2\}$ 跟 $\{2,1\}$ 在 list 中不一樣，而 set 中相同
另外 list 跟 tuple 常常在說同一件事。假設 list 中有 $n$ 個元素，我們可以說這是一個 **length** 為 $n$ 的 list 或是 $n$-tuple


>[!note] 概念三
>如何從 $\mathbb{F}$  推廣到 $\mathbb{F}^n$

有了 list 的想法後，我們可以直接用它配合 $\mathbb{F}$ 直接組合出 $\mathbb{F}^n$，這十分契合我們對於 $n$ 維空間的想像，先定義好對於基底的描述順序再塞值，因此順序是重要的，而值可以重複出現，因為可以在不同軸上有相同大小。

>[!question] 問題一
>為何在 $\mathbb{F}^n$ 只定義 scalar multiplication 不定義如下的 vector multiplication
>$$(a_1, a_2, \cdots, a_n) \times (b_1, b_2, \cdots, b_n) = (a_1 b_1, a_2 b_2, \cdots, a_n b_n)$$

首先對於這個給予不同軸不同權重的操作可以由矩陣來提供，而如果想直接跳過矩陣用上面定義的向量乘法來處理，會面臨兩主要問題
1. 向量在這向量乘法處理後，其結果會隨著座標軸的改變而改變
2. 存在零因子
先說第一點，考慮在 $\mathbb{R}^2$ 並用基底 $B_1 = \{(1,0), (0,1)\}$ 中的一向量 $(1,1)$ 我們把它乘上 $(2,0)$ 會得到
$$(1,1) \times (2,0) = (2,0)_{B_1}$$
另一方面，如果我們現在換另一基底 $B_2 = \{(\frac{1}{\sqrt{2}},\frac{1}{\sqrt{2}}), (\frac{-1}{\sqrt{2}},\frac{1}{\sqrt{2}})\}$ 向量變為 $(\sqrt{2},0)$ 我們把它「乘上 $(2,0)$ 」變成 「乘上 $(\sqrt{2},-\sqrt{2})$ 」會得到
$$(\sqrt{2},0) \times (\sqrt{2},-\sqrt{2}) = (2,0)_{B_2}$$
把 $(2,0)_{B_2}$ 換到 $B_1$ 會變成 $(\sqrt{2}, \sqrt{2})_{B_1}$，一樣的向量相乘，只是換個座標，結果竟然改變， BAD!
- 兩相同向量只是換個基底，乘起來的結果竟然不同，這使的這乘法是 dependent on the basis。
這結果可以用以下角度直覺思考，考慮一個可逆矩陣 $P$ 表達座標的改變，那如果要這邊的向量乘法不隨基底改變而變化，以下等式要成立$$P(u⊙v)=(Pu)⊙(Pv)$$可以看到右邊 $P$ 作用了兩遍，而左邊 $P$ 只作用了一遍，兩者很難相等，除非 $P$ 是 permutation matrices。而前面的例子差45度也是因為有一遍沒調回來。對於第二點蠻好理解，在 $\mathbb{R}^2$ 中對於 $(1,0)$ 只要乘上 $(0,k), k \in \mathbb{R}$ 都為零，因此在這操作下無法保證 $$a \times b = 0 \Rightarrow a = 0 \text{ or } b = 0$$其實這乘法有人在用叫 Hadamard product ，常用於機器學習或是資料處理方面，因為每一維度可以給予不同權重。

>[!question] 問題二
>為啥作者說 "I prefer avoiding arbitrary fields at this level because they introduce extra abstraction without leading to any new linear algebra"

直接操作這在 Digression on Fields 這一小節的說明: "Throughout much of this book (except for Chapters 6 and 7, which deal with inner product spaces) you can think of $\mathbb{F}$ as denoting an arbitrary field instead of $\mathbb{R}$ or $\mathbb{C}$."
## Section 1B (Definition of Vector Space)

>[!definition] 定義一 (vector space)
>在 vector space 是指一個集合 $V$ 且定義 addition 跟 scalar multiplication 兩操做，而這兩操作要符合以下性質。對於 addition 來說
>- commutativity: $u + v = v +u$ for all $u,v \in V$.
>- associativity: $(u+v) + w = u + (v+w)$ and $(ab)v = a(bv)$ for all $u,v,w \in V$ and for $a,b \in \mathbb{F}$.
>- additive identity: There exists an element $0 \in V$ such that $v + 0 = v$ for all $v \in V$.
>- additive inverse: For everey $v \in V$, there exists $w \in V$ such that $v + w = 0$.
>- multiplicative identity: $1v = v$ for all $v \in V$.
>- distributive properties: $a(u + v) = au + av$ and $(a+b)v = av + bv$ for all $a,b \in \mathbb{F}$ and all $u,v \in V$

思考一下為啥 associativity 不討論 $ab = ba$ ? 另外為啥沒有 multiplicative inverse ?
第一個問題是因為 $a,b \in \mathbb{F}$ 所以它本身就有 associativity，不必再提；第二了問題除非定義 vector multiplication 不然做不出來。

>[!important] $\mathbb{F}$ 是 field
>$\mathbb{F}$ 是 field 所以對於 field 內非零的元素有乘法反元素，所以 $\mathbb{Z}$ 不能當 vector space 中乘法所用的 field。
>- 純量取自 **field** → vector space
>- 純量取自一般的 **ring** → module

>[!important] addition, scalar multiplication 的封閉性
>addition 跟 scalar multiplication 這兩運算是要保證封閉性的，不能算一算跑出 $V$ 的集合外。

>[!question] 問題一
>vector space 的集合 $V$ 可以是空集合嗎?

答案是不行。在 additive identity 中明確要求要存在 $0 \in V$ 而空集合不具備。須注意 additive inverse 是滿足的，因為 $V$ 是空的。

>[!note] 概念一
>因為 scalar multiplication 取決於 $\mathbb{F}$，所以在精確地描述向量空間為: "vector space over $\mathbb{F}$"。

>[!definition] 定義二
>定義記號 $\mathbb{F}^{S}$ 其中 $S$ 是集合，$\mathbb{F}^{S}$ 代表從 $S$ 到 $\mathbb{F}$ 的函式集合。定義 addition 跟 scalar multiplication 如下
>- addition: For $f,g \in \mathbb{F}^{S}$, the sum $f + g \in \mathbb{F}^{S}$ is the function defined by $$(f+g)(x) = f(x) + g(x)$$ for all $x \in S$
>- scalar multiplication: For $\lambda \in \mathbb{F}$ and $f \in \mathbb{F}^{S}$, the product $\lambda f \in \mathbb{F}^{S}$ is the function defined by $$(\lambda f)(x) = \lambda f(x)$$ for all $x \in S$

>[!question] 問題二
>如果 $S$ 是非空集合，驗證 $\mathbb{F}^{S}$ 是 vector space over $\mathbb{F}$ 。

在之前已經定義過 addiction 跟 scalar multiplication
- addition: For any $f, g \in \mathbb{F}^S$ , $(f + g)(x) = f(x) + g(x)$ for every $x \in S$
- scalar multiplication: For any $f \in \mathbb{F}^S, \lambda \in \mathbb{F}$, $(\lambda f)(x) = \lambda f(x)$ for every $x \in S$
驗性質吧
- Commutativity
Given $f, g \in \mathbb{F}^S, x \in S$ we have $$(f + g)(x) = f(x) + g(x) = g(x) + f(x) = (g+f)(x)$$ Note that in the second equation holds since the commutativity of field $\mathbb{F}$
- Associativity
Given $f, g, h \in \mathbb{F}^S, x \in S$ we have $$(f + (g + h))(x) = f(x) + (g(x) + h(x)) = f(x) + g(x) + h(x) = (f(x) + g(x)) + h(x) = ((f + g) + h)(x)$$ Note that in the second and third equation holds since the associativity of field $\mathbb{F}$
 Given $f \in \mathbb{F}^S, x \in S, \lambda_1, \lambda_2 \in \mathbb{F}$ we have $$(\lambda_1 (\lambda_2 f))(x) = \lambda_1 ((\lambda_2 f)(x)) = \lambda_1 \lambda_2 (f)(x) = \lambda_1 \lambda_2 f(x) = (\lambda_1 \lambda_2) f(x) = (\lambda_1 \lambda_2 f) (x)$$ Note that the 4th equation holds since the associativity of field $\mathbb{F}$
 - Addiction identity
 We can take $0 \in \mathbb{F}^S$ defined by $$0(x) = 0$$for all $x \in S$. We can check this is addition idendity by $0$ is addition identity in $\mathbb{F}$
 - Addition inverse
 We can take $-f \in \mathbb{F}^S$ defined by $$(-f)(x) = -(f(x))$$for all $x \in S$. We can check this is addition inverse by $-v$ is addition inverse of $v$ in $\mathbb{F}$
 - Multiplcation identity
 We can take $1 \in \mathbb{F}$ then$$(1f)(x) = f(x)), \text{ for all $x \in S$.}$$since $1$ is multiplication identity in $\mathbb{F}$.
 - Distribute property
Given $f \in \mathbb{F}^S, a,b \in \mathbb{F}$ we have $$(a + b) f(x) = af(x) + bf(x)$$ since $f(x) \in \mathbb{F}$ and field $\mathbb{F}$ has distribute property.
Given $f,g \in \mathbb{F}^S, a \in \mathbb{F}$ we have $$(a (f + g))(x) = a(f+g)(x) = a(f(x) + g(x)) = af(x) + ag(x) = (af + ag)(x)$$Since field $\mathbb{F}$ has distribute property, the third equation holds automatically.

>[!note] 概念二
>我們要先說明加法反元素唯一，才可以訂定記號: $-v$ 為 $v$ 的加法反元素同時 $w - v = w + (-v)$


>[!question] 問題三
>用如下方式證明: "Every element in a vector space has a unique additive inverse" 錯在哪裡? 
>給定一向量 $v$，假設其有兩反元素 $w,w^{\prime}$ 則 $$v + w = 0, v + w^{\prime} = 0 \Rightarrow (v+w) - (v+w^{\prime}) = 0 \Rightarrow w - w^{\prime} = 0 \Rightarrow w = w^{\prime}$$

上面的推導直接把減法用上，但我們需要先證明加法反元素唯一，才可以定義 $-v$ 跟減法。因此該證明只能用加法處理，其證法如下$$w = w + 0 = w + (v + w^{\prime}) = (w + v) + w^{\prime} = 0 + w^{\prime} = w^{\prime} + 0 = w^{\prime}$$ 
>[!note] 概念三
>"The only part of the definition of a vector space that connects vector addition and scalar multiplication is the distributive property." 怎樣去理解這句話

這句話的一個例子是: 用 $v + v = (1+1) v$ 這件事，可以用在證明 $$0v = 0 \text{ for all } v \in V$$證明過程為 $0v = (0+0)v = 0v + 0v \Rightarrow 0v = 0$
這句話想表達的是:「分配律是從乘法那一側通往加法那一側的唯一通道，過去之後就能用加法的工具收尾」

>[!question] 問題四
>Show that in the definition of a vector space (1.20), the additive inverse condition can be replaced with the condition that 0𝑣 = 0for all 𝑣 ∈ 𝑉. Here the 0 on the left side is the number 0, and the 0 on the right side is the additive identity of 𝑉." 
>我的證明如下: 
>Since $0v = 0$ for all 𝑣 ∈ 𝑉. For any given v, we have $0 = 0v = (1 - 1)v = v - v$. Hence the additive inverse can be seen as -v
>這證明有可以挑刺的地方嗎?

你把 `-v` 跟 `(-1) v` 混為一談 (在 theorem 1.32 其實有證 $-v = (-1) v$)，這兩者意思不同
- `-v`: 為 $v$ 在 $V$ 的加法反元素
- `(-1)v`: 為 $V$ 中的一元素乘上 $-1$ 這個 scalar 為一個 scalar multiplcation 操作
所以上面證明應該如下:
Since $0v = 0$ for all 𝑣 ∈ 𝑉. For any given v, we have $0 = 0v = (1 - 1)v = v + (-1)v$. Hence the additive inverse can be seen as $(-1)v$

>[!important] 一些想法
>1. 加法反元素的存在讓我們在證明時可以左右兩邊一起刪東西，同時他的唯一性也讓我們可以構築出減法。
>2. 在 vector space 中「加法反元素」跟「加法單位原」都是唯一的。
>3. Distributive rule 可以把加法跟乘法的混和運算，拆成單純的加法運算。
## Section 1C (Subspaces)

>[!definition] 定義一 (subspace)
>A subset 𝑈 of 𝑉 is called a subspace of 𝑉 if 𝑈 is also a vector space with the same additive identity, addition, and scalar multiplication as on 𝑉.

對於 subspace 的想像是，先有一個大的 vector space ($V$)，我再去其中找一個小的(可能一樣大)的子集，而該子集一樣要符合 vector space 的定義，同時因為是在 $V$ 裡面找，所以加法和乘法的定義要相同。"the same additive identity" 是多餘的 (證明在問題一)，這存在的意義只是強調
- 我們可以用 $0$ 不在 $U$ 內快速證明 $U$ 不是 subspace。
- subspace 不是獨立造出一個 vector space 在某一程度上繼承自母空間。

>[!question] 問題一
>定義一的 the same additive identity 是否多餘?

~~首先 $U$ 是 vector space 所以一定存在 $0_U \in U$。在前面已經說過:「在 $V$ 中的 additive identity 唯一」，所以 $0_U = 0_V$。問題一的答案是對的， the same additive identity 是多餘。~~
>上面這證明是錯的，因為 $0_U \in U$ 只在 $U$ 中是 identity，對於 $V \setminus U$ 的部分毫無說明，不能直接調用「在 $V$ 中的 additive identity 唯一」這結論來說明 $0_U = 0_V$ 除非你可以將 $0_U$ 擴展到整個 $V$ 上面。

正確證明如下:
Since $U$ is a vector space, there exist $0_U \in U$ such that for every $x \in U$ we have $x + 0_U = x$. Hence$$0_U = 0_U + 0_U$$On the other hand, by the additive inverse, there exist $w \in V$ such that $w + 0_U = 0_V$. Therefore$$0_U = 0_U + 0_U  \Rightarrow w + 0_U = w+  0_U + 0_U \Rightarrow 0_V = 0_U$$
>[!note] 概念一
>subspace 的快速判斷法如下:
>A subset $U$ of $V$ is a subspace of $V$ if and only if $U$ satisfies the following three conditions. 
>- additive identity: $0 \in U$. 
>- closed under addition: $u,w \in U$ implies $u + w \in U$. 
>- closed under scalar multiplication: $a \in \mathbb{F}$ and $u \in U$ implies $au \in U$.

這個想法是先用 additive identity 確認此集合非空 (其實只要 $U$ 有一向量就行，additive identity 可以用 closed under scalar multiplication 去推)，這邊特定描述 $0$ 是想給一個快速判斷非子空間的方法。closed under addition 跟 closed under scalar multiplication 跟 vector space 的 addition 和 scaalar multiplication 一樣都要保證封閉性 (closure)。而 communitativity 、assocaitivity 、distributive law、multiplcation 可以靠母空間的 $V$ 來提供，至於 addition inverse 可以靠 $-1(v)$ 和 closed under scalar multiplication 處理。

>[!note] 概念二
>"The union of subspaces is rarely a sub space, which is why we usually work with sums rather than unions."

這邊一簡單例子是在 $\mathbb{R}^2$ 上的 $x$ 軸和 $y$ 軸取 union 下 addition 顯然不封閉。所以我們通常是討論 sum of subspaces

>[!definition] 定義二 (sum of subspaces)
>Suppose $V_1, \cdots, V_m$ are subspaces of $𝑉$. The sum of $V_1, \cdots, V_m$, denoted by $V_1, \cdots, V_m$, is the set of all possible sums of elements of $V_1, \cdots, V_m$. More precisely, $$V_1 + \cdots + V_𝑚 = \{v_1 + \cdots + v_𝑚 ∶ v_1 \in V_1, \cdots , v_𝑚 \in V_𝑚 \}.$$

>[!note] 概念三
>"Sums of subspaces in the theory of vector spaces are analogous to unions of subsets in set theory. Given two subspaces of a vector space, the smallest subspace containing them is their sum. Analogously, given two subsets of a set, the smallest subset containing them is their union."

在集合中包含兩子集合的最小子集合為其 union，另一方面，vector space 中包含兩 subspace 的最小 subspace 為其 sum。
下一個要關注的點是:「再由多個 subspace ($V_1, \cdots, V_m$) sum 出來的 subspace 中，其所有 vector 表達是否可以用 $v_i \in V_i$ 唯一表達出來」，舉個不唯一的例子:
考慮 $V_1 = \{(x,0) \in \mathbb{F}^2 : x \in \mathbb{F}\}, V_2 = \{(0,x) \in \mathbb{F}^2 : x \in \mathbb{F}\}, V_3 = \{(x,x) \in \mathbb{F}^2 : x \in \mathbb{F}\}$，對於 $(1,1) = (1,0) + (0, 1)$ 可以有兩種方法湊。

>[!definition] 定義二 (direct sum, $\oplus$)
>Suppose $V_1, \cdots, V_m$ are subspaces of $V$. 
>- The sum $V_1 + \cdots + V_m$ is called a _direct sum_ if each element of $V_1 + \cdots + V_m$ can be written in only one way as a sum $v_1 + \cdots + v_m$, where each $v_k \in V_k$. 
>- If $V_1 + \cdots + V_m$ is a direct sum, then $V_1 \oplus \cdots \oplus V_m$ denotes $V_1 + \cdots + V_m$, with the $\oplus$ notation serving as an indication that this is a direct sum.

給一個快速測是否為 direct sum 的方法
>[!note] 概念四
>Suppose $V_1, \cdots, V_m$ are subspaces of $V$. Then $V_1 + \cdots + V_m$ is a direct sum if and only if the only way to write 0 as a sum $v_1 + \cdots + v_m$, where each $v_k \in V_k$, is by taking each $v_k$ equal to 0.

這邊的想法是看下面這式子$$v + 0 = v$$如果 $0$ 有非全零的表達方式，那我一定可以依此法造出第二個為 $v$ 的表達方式。
> $0$ 表法唯一 $\Longleftrightarrow$ 每個 $v$ 表法唯一 $\Longleftrightarrow$ 是直和.

>[!question] 習題一
>Prove or give a counterexample: If $U$ is a nonempty subset of $\mathbb{R}^2$ such that $U$ is closed under addition and under taking additive inverses (meaning $-u \in U$ whenever $u \in U$),then $U$ is a subspace of $\mathbb{R}^2$.
>證明它與為何不只問 closed under addition?

這邊的思路是 scalar multiplication 對一向量做縮放，關鍵在縮放時，有可能跑到不屬於他的地方。我們可以取$$U = \{(x, x) \mid x \in \mathbb{Z} \}$$顯然 $u_1=(x_1, x_1), u_2=(x_2, x_2) \in U$ 時 $u_1 + u_2 = (x_1 + x_2, x_1 + x_2) \in U$ 且 $u = (x_1, x_1) \in U$ 時 $-u = (-x_1, -x_1) \in U$。但取 $(1,1) \in U$ 和 $\sqrt{2} \in \mathbb{R}$ 時 $\sqrt{2}(1,1,) \notin U$。
至於為何不只問 closed under addition，因為 closed under addition 跟 additive inverse 可以構成子群，沒那麼無聊。在 subspace 的反元素是由 scalar multiplication 提供。

>[!question] 習題二
>Give an example of a nonempty subset $U$ of $\mathbb{R}^2$ such that $U$ is closed under scalar multiplication, but $U$ is not a subspace of $\mathbb{R}^2$.

想法是 scalar multiplication 是針對一個向量做縮放，當我掏出兩個向量時，閣下如何應對? 取$$U = \{(x,0) \mid x \in \mathbb{R} \} \cup \{(0,y) \mid y \in \mathbb{R} \}$$顯然任取 $u \in U$ 有 $ku \in U, \forall k \in \mathbb{R}$ 因為 $\mathbb{R}$ 本身的乘法封閉性，但取 $u_1 = (1,0), u_2 = (0,1) \in U$ 時，$u_1 + u_2 = (1,1) \neq U$。

>[!important] 習題一跟二的感想
>scalar multiplication 會對單一個向量作拉伸，可以表達出某一個方向的所有可能；addition 對兩個向量做相加，只要兩向量 linear independent，我可以生出有別於這兩向量所指方向的新方向，如果兩向量 linear dependent，我可以生出有別於這兩向量長度的新長度（兩向量非空）。只要這兩個向量無法透過拉伸得到對方，及兩向量 linear independent 那我透過相加和 scalar multiplication 可以得到一個二維的空間而拉伸只有一維的空間


>[!question] 習題三
>A function $f: \mathbb{R} \to \mathbb{R}$ is called periodic if there exists a positive number $p$ such that $f(x) = f(x+p)$ for all $x \in \mathbb{R}$. Is the set of periodic functions from  $\mathbb{R}$ to $\mathbb{R}$ a subspace of $\mathbb{R}^\mathbb{R}$? Explain.

這題單純的問題是，如果 $f_1$ 的週期為 $p_1$，$f_2$ 的週期為 $p_2$，那 $f_1 + f_2$ 是否具有週期性？
如果 $p_1 = \frac{a_1}{b_1}, p_2 = \frac{a_2}{b_2} \in \mathbb{Q}$ 那 $f_1 + f_2$ 至少有一週期 $lcm(b_1 p_1, b_2 p_2)$。~~反例在於 $\cancel{p_1 \in \mathbb{Q}, p_2 \in \mathbb{R} \setminus \mathbb{Q}}$ ，證明是利用反證法，假設週期為 $P$ 那$$\cancel{\frac{P}{p_1} \in \mathbb{N} \Rightarrow P \in \mathbb{N}}$$ 但在另一方面$$\cancel{\frac{P}{p_2} \in \mathbb{N} \Rightarrow P \in \mathbb{R} \setminus \mathbb{Q}}$$矛盾，所以不存在週期。~~
上面**推矛盾的方法是錯的**，因為 $P$ 不一定是 $p_1, p_2$ 的整數倍，舉例 $f_1$ 的週期為 1 取 $f_2 = -f_1$ 那$$f_1 + f_2 = 0$$為一常數函數，所以她週期可以為 $\sqrt{2}$，顯然  $\sqrt{2}$ 不為 1 的整數倍。這題的反例是取$$f_1 = cos(\pi x), f_2 = cos(x)$$那 $f_1$ 的週期為 2 而 $f_2$ 的週期為 $2 \pi$，至於 $f_1 + f_2$ 的週期，我們可以取一個特別的值 $f_1 + f_2 = 2$ 時，顯然 $x=0$ 是一個解，那其他解呢？我們可以看到對於 $f_1$ 來說，他為 1 僅在 $x = 2n, n \in \mathbb{Z}$；對於 $f_2$ 來說，他為 1 僅在 $x = 2 \pi n, n \in \mathbb{Z}$，所以兩者僅在 $x=0$ 時有 $f_1 + f_2 = 2$ 因此沒有週期。

>[!important] 週期函數的一些想法
>一般在想週期函數時，腦袋常出現的函數是 $\sin, \cos$ 這類重複出像相同起伏的函數，但其實週期函數不一定會有起伏、人腦想的到的規律樣態、亦或是一個規律週期，例如: 常數函數和 Dirichlet function。所以如果要看是否為週期函數，要仔細地逐點去看，造反例時對著一個點施加壓力。
 
>[!question] 習題四
>Prove that the union of three subspaces of $V$ is a subspace of $V$ if and only if one of the subspaces contains the other two.

首先這敘述不一定是對的。在 $\mathbb{F}^2_2$ 中令$$V_1 = \{(0,0), (1,0)\}, V_2 = \{(0,0), (0,1)\}, V_3 = \{(0,0), (1,1)\}$$顯然這三個 subspace 不互相包含，但取 union 是一個 subspace。
這題的證明對於 $\Leftarrow$ 是顯然成立的，因為最大的那個 subspace 本身就是個 subspace。另一方面對於 $\Rightarrow$ 來說，先假設 $\vert F \vert \geq 3$，思路是由這一證明"Prove that the union of two subspaces of 𝑉 is a subspace of 𝑉 if and only if one of the subspaces is contained in the other."知任一 subspace 不能是其他 subspace 的 union 不然就結束了。這邊需要分兩部分討論，第一部分是 $V_1, V_2, V_3$ 有任一個 subspace 包含於其餘 subspace 聯集的集合中，那直接證"Prove that the union of two subspaces of 𝑉 is a subspace of 𝑉 if and only if one of the subspaces is contained in the other."即可；第二部分是存在 $u,v$ 滿足性質$$u \in V_1 \setminus (V_2 \cup V_3) ; v \in V_2 \setminus (V_1 \cup V_3)$$接著考慮$$S = \{u + tv \mid t \in \mathbb{F}, t \neq 0\}$$對於所有 $s \in S$ 都不可能屬於 $V_1, V_2$，因為這會推出 $u \in V_2$ 或是 $v \in V_1$ 的矛盾，剩下的唯一可能是 $S \subseteq V_3$，因為 $\vert F \vert \geq 3$，存在 $t_1 \neq t_2$ 且 $u + t_1 v , u + t_2 v \in S$ 那$$w = (u + t_1 v) - (u + t_2 v) = (t_1 - t_2)v \in V_3$$同時 $h = u + (t_2 - t_1) v \in V_3$ 所以$$h + w = u + (t_2 - t_1)v + (t_1 - t_2)v = u$$因為 $h,w \in V_3$ 所以 $u \in V_3$ 矛盾。

>[!question] 問題四改
>Prove that in the infinite field, the union of $k, k \geq 3$ subspaces of $V$  is a subspace of $V$ if and only if one of the subspaces contains the other subspaces.

這證明是用數學歸納法來做，先假設到 $k-1$ 都成立 (base case 已經驗證過)，接著證有 $k$ 個 subspaces。跟原題一樣分兩部分討論，第一部分是 $V_1, V_2, \cdots, V_k$ 有任一一個 subspace 全在其餘 subspace 的聯集中，那直接靠假設就秒了；第二部分是$V_1, V_2, \cdots, V_k$ 沒有任一一個 subspace 全在其餘 subspace 的聯集中，所以存在 $u,w$ 使得$$u \in V_1 \setminus (V_2 \cup \cdots \cup V_k), w \in V_2 \setminus (V_1 \cup V_3 \cup V_4 \cup \cdots \cup V_k)$$依舊考慮$$S = \{u + tw \mid t \in \mathbb{F}, t \neq 0\}$$由前一題知，$S$ 不屬於 $V_1, V_2$。因為 infinite field 所以存在 $t_1, t_2, \cdots t_{k-1} \in \mathbb{F}$ 而且 $t_1, t_2, \cdots t_{k-1}$皆不同且非零，由鴿籠原理知一定有 $u + t_{j_1}w,u + t_{j-2}w \in V_{j}, j \in \{3,4, \cdots, k\}$ 而這推出$$\underbrace{(u + t_{j_1} w)}_h - \underbrace{(u + t_{j_2} w)}_p = (t_{j_1} - t_{j_2}) w$$因為 $h,p \in V_j$ 所以 $(t_{j_1} - t_{j_2}) w \in V_j$，由 closed under multiplication 且 $t_{j_1} - t_{j_2} \neq 0$ 知 $w$ 在 $V_j$ (在 field 中乘法反完素存在，所以 $(t_{j_1} - t_{j_2})^{-1}$ 存在)，矛盾。**注意 zero vector 包含在所有 vector space 中**。

>[!important] 對於整族造法的想法
>先簡化所有的 $V$ 皆為一維，不妨假設 $V_1$ 是 x 軸而$V_2$ 是 y 軸，而現在 $V_1, V_2, V_3$ 聯集要是一個 vector space，包含 $V_1$ 跟 $V_2$ 的 vector space 一定有 $\mathbb{R}^2$，所以 $V_3$ 至少有 $S = \mathbb{R}^2 \setminus (\{(x,0) \mid x \in \mathbb{R}\} \cup \{(0,y) \mid y \in \mathbb{R}\})$，而反證法的框架限制不能取 x 軸跟 y 軸，正如 $S$ 所描述的樣子。現在問題是 $S$ 是 vector space 嗎？一顯然反例是取固定 x 軸或 y 軸大小的兩向量，相減即可 (這邊在高維是用鴿籠配合相減)，這正是你在整族造法在做的事情。一個細節是我們取 $S^{\prime} = \{u + tw \mid t \in \mathbb{F}, t \neq 0\}$ 而不是取 $S^{\prime} = \{t_1 u + t_2w \mid t_i \in \mathbb{F}, t_i \neq 0\}$ 是因為固定 $u$ 那條軸的分量(係數，所以 $u$ 不帶變數。



>[!question] 習題五
>Suppose $U = \{(x,y,x+y,x-y,2x) \in \mathbb{F}^5: x,y \in \mathbb{F}\}$. Find a subspace $W$ of $\mathbb{F}^5$ such that $\mathbb{F}^5 = U \oplus W$.

可以用擴充基底的方式求解。$U$ 的基底為$$u_1 = (1,0,1,1,2), u_2 = (0,1,1,-1,0)$$另一方面 $\mathbb{F}^5$ 的基底為$$e_1 = (1,0,0,0,0), e_2 = (0,1,0,0,0), e_3 = (0,0,1,0,0), e_4 = (0,0,0,1,0), e_5 = (0,0,0,0,1)$$接下來依照以下順序 $u_1, u_2, e_1, e_2, e_3, e_4, e_5$ 看哪些向量**不可以**由前面的向量組合出來，那些向量可以用 direct sum 構出 $\mathbb{F}^5$。找這些向量一個簡單的技巧是用 leading term，如果 leading term 跟之前的向量不同則該向量無法由前面的向量組合出來。用這例題的例子，考慮這一矩陣$$\begin{pmatrix} 
1 &0 &1 &1 &2 \\
0 &1 &1 &-1 &0 \\
1 &0 &0 &0 &0 \\
0 &1 &0 &0 &0 \\
0 &0 &1 &0 &0 \\
0 &0 &0 &1 &0 \\
0 &0 &0 &0 &1 \\
\end{pmatrix} \rightarrow \begin{pmatrix} 
1 &0 &1 &1 &2 \\
0 &1 &1 &-1 &0 \\
0 &0 &-1 &-1 &-2 \\
0 &1 &0 &0 &0 \\
0 &0 &1 &0 &0 \\
0 &0 &0 &1 &0 \\
0 &0 &0 &0 &1 \\
\end{pmatrix} \rightarrow \begin{pmatrix} 
1 &0 &1 &1 &2 \\
0 &1 &1 &-1 &0 \\
0 &0 &-1 &-1 &-2 \\
0 &0 &0 &2 &2 \\
0 &0 &1 &0 &0 \\
0 &0 &0 &1 &0 \\
0 &0 &0 &0 &1 \\
\end{pmatrix} \rightarrow \begin{pmatrix} 
1 &0 &1 &1 &2 \\
0 &1 &1 &-1 &0 \\
0 &0 &-1 &-1 &-2 \\
0 &0 &0 &2 &2 \\
0 &0 &0 &0 &-1 \\
0 &0 &0 &1 &0 \\
0 &0 &0 &0 &1 \\
\end{pmatrix}$$第一步: $R_3 - R_1$，$R_3$ 變為 $\begin{pmatrix}0 &0 &-1 &-1 &-2\end{pmatrix}$
第二步: $R_4 - R_2 - R_3$，$R_4$ 變為 $\begin{pmatrix}0 &0 &0 &2 &2\end{pmatrix}$
第三步: $R_5 + R_3 + \frac{1}{2}R_4$，$R_5$ 變為 $\begin{pmatrix}0 &0 &0 &0 &-1\end{pmatrix}$
所以可以用 direct sum 構出 $\mathbb{F}^5$ 的向量為 $u_1, u_2, e_1, e_2, e_3$，所以 $W$ 可以選擇$$\mathrm{span}(e_1, e_2, e_3) = \{(x,y,z, 0, 0) \mid x,y,z \in \mathbb{F} \}$$須注意在用 direct sum 時針對的是 subspace 所以不能寫 $e_1 \oplus e_2 \oplus e_3$，要寫 $\mathrm{span}(e_1) \oplus \mathrm{span}(e_2) \oplus \mathrm{span}(e_3)$

>[!question] 問題六
>Prove or give a counterexample: If $V_1, V_2, U$ are subspaces of $V$ such that $V = V_1 \oplus U$ and $V = V_2 \oplus U$, then $V_1 = V_2$.

這邊想問的是對於 subspace 的 direct sum 有無四則運算中加減法的想法，即$$x = a_1 + b, x = a_2 + b \Rightarrow a_1 = a_2 = x - b$$
~~假設 $\cancel{V_1 \neq V_2}$ 那 WLOG 我們可以假設存在 $\cancel{v \in V_1, v \notin V_2}$ 同時 $\cancel{v \in V}$ 所以 $v$ 一定在 $U$ 中或是 $\cancel{v = w + u}$ for some $\cancel{w \in V_2}$ , $\cancel{u \in U}$。如果 $v$ 在 $U$ 中那 $\cancel{V = V_1 \oplus U}$ 不成立；另一方面 $\cancel{w \in V}$ 那一定有 $\cancel{w = p + t}$ for some $\cancel{p \in V_1}$ , $\cancel{t \in U}$ 那$$\cancel{v = w + u = p + t + u \Rightarrow v-p = t + u}$$所以一個在 $V_1$ 的元素等於一個在 $U$ 的元素，矛盾。~~
**上述證明有一重要盲點是:「以為 $V_1 \oplus V_2$ 則 $V_1$ 和 $V_2$ 沒有共享元素」這是錯的因為 zero vector 是每個 space 的標配。**
我們造個一槌定音的反例，考慮 $V = \mathbb{R}^2$，取 $U  = \{(x,0) \mid x \in \mathbb{R}\}$ 則 $V_1$ 跟 $V_2$ 可以分別取$$V_1 = \{(0,y) \mid y \in \mathbb{R}\}, V_2 = \{(y,y) \mid y \in \mathbb{R}\}$$這兩者皆符合條件$$V = V_1 \oplus U, V = V_2 \oplus U$$但不符合 $V_1 = V_2$。
帶回原本錯誤的證明來看，我們可以取 $(0, 1) \in V_1$ 同時 $(1,1) + (-1, 0) = (0,1)$進一步猜解得$$(0,1) + (1,0) + (-1, 0) = (0,1) \Rightarrow (0,1) + (0,-1) = (-1,0) + (1,0) \Rightarrow (0,0) = (0,0)$$
>[!important] 對於 sum 跟 direct sum 的一些想法
>對於 sum 和 direct sum 在 subspace 中來說都沒有四則運算中減法的想法，因為在一個 $V$ 中對於 subspace 的補空間不一定唯一。Direct sum 對於 vector 層面才有好的分解性，正如它的定義 "The definition of direct sum requires every vector in the sum to have a unique representation as an appropriate sum."


>[!question] 問題七
>A function $f ∶ \mathbb{R} \to \mathbb{R}$ is called even if$$f(x) = f(-x)$$for all $x \in \mathbb{R}$. A function $f ∶ \mathbb{R} \to \mathbb{R}$ is called odd if$$f(x) = -f(-x)$$for all $x \in \mathbb{R}$. Let $V_e$ denote the set of real-valued even functions on $\mathbb{R}$ and let $V_O$ denote the set of real-valued odd functions on $\mathbb{R}$. Show that $\mathbb{R}^\mathbb{R} = V_e \oplus V_d$.

這題的解法其實挺直覺的，原定義的描述是針對一組 $(x, -x)$ 所以在構造新的 $f_o, f_e$ 的時候也是針對一組 $(x, -x)$ 來處理。現在隨意給定一個 $f \in \mathbb{R}^\mathbb{R}$ 並令其 $f(x) = a, f(-x) = b$ 我們可以令$$f_o(x) = a_o, f_o(-x) = -a_o, f_e(x) = a_e, f_e(-x) = a_e$$ 接著解方程$$a_o + a_e = a, -a_o + a_e = b \Rightarrow a_o = \frac{a-b}{2}, a_e = \frac{a+b}{2}$$所以任意一個 $f \in \mathbb{R}^\mathbb{R}$ 的確可以分解成 $f = f_o + f_e, f_o \in V_o, f_e \in V_e$。最後驗證 direct sum 假設 $f_0 \in V_o, f_e \in V_e$ 考慮 $$f_o + f_e = 0$$假設$$f_o(x) = a_o, f_o(-x) = -a_o, f_e(x) = a_e, f_o(-x) = a_e$$那$$a_o + a_e = 0, -a_o + a_e = 0 \Rightarrow a_o =0, a_e = 0$$所以 $f_e, f_o$ 都為 0，這表示 direct sum 成立。
 


