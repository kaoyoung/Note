### 參考應用層p23、p24
# C-S model
對於C-S model分析相對簡單，服務可以獨立的分為
- server的上載
- client的下載
時間分析 :
$$
\text{total time = } \max\left(\frac{NF}{u_s}, \frac{F}{d_{min}} \right)
$$
- 有N個client要下載大小為F bit的文件
- $u_s$ 為server上傳帶寬
- $d_{min}$為client最小的下載速率
## 極端情況分析
### 情況一 : 上傳端比下載端快 : $\frac{u_s}{N} \geq d_{min}$
顯然結論是total time被client端的下載速率制約，那此時資料傳遞方式為何 : 
>[!Thinking Logic]
>server上載速率比client最小下載速率還大，用client最小下載速率當server對每個client上傳速率，如果把server對每個client上傳速率設的比client最小下載速率大，你也要等最慢那個人下載完，所以以最小那個client當標竿。常見問題點是把上載和下載分成兩階段，但server上載和client下載其實是同一件事的一體兩面，不要把這過程想成兩步驟(在開始和結束端有邊界要討論，但只要傳的過程夠久，這點時間可以忽略)。可以用水管的進出來思考，一邊進水另一邊噴水，除了在剛開始只有進水沒有出水，另外還有進水結束時只有出水直到水管內水流光。

物理條件分析 : 
1. 對於server端 : 上載帶寬不能大於$u_s$
 上載帶寬 =  全部client下載速率相加  $= N \times d_{min} \leq u_s$ 
2. 對於client端 : 下載速率不能超過自己的下載速率
顯然符合。如果server端上傳速率超過client端下載速率，有buffer來擋一下，但這時要考慮buffer滿的問題，所以設計時要求對於每個client，server都給和client下載速率一樣大的上載速率。
### 情況二 : 下載端比上載端快 : $\frac{u_s}{N} \leq d_{min}$
顯然結論是total time被servert端的上載速率制約，那此時資料傳遞方式為何 : 
>[!Thinking Logic]
>server上載速率比client最小下載速率還小，所以我們不能向上面一樣最慢的人當基準，而是要榨乾server的上傳帶寬，及server對每個client上載帶寬設為$\frac{u_s}{N}$，另一方面因為client最小下載速率比server上載速率快所以不需要考慮client buffer的問題。

物理條件分析 : 
1. 對於server端 : 上載帶寬不能大於$u_s$
 上載帶寬 =  全部client下載速率相加  $= N \times \frac{u_s}{N} = u_s$
 2. 對於client端 : 下載速率不能超過自己的下載速率
顯然符合。我們取client最小下載速率當server上載速率，所以沒buffer問題。
# P2P model
**我們這邊不考慮客戶端下載時間，及假設客戶端下載帶寬很大**
時間分析 : 
$$
\text{total time = } \max \left( \frac{F}{u_s}, \frac{F}{d_{min}}, \frac{NF}{u_s + u_1 + \cdots + u_N} \right)
$$
- 有N個client要下載大小為F bit的文件
- $u_s$ 為server上傳帶寬，$u_i$ 為client上載帶寬
- $d_{min}$為client最小的下載速率
>[!IMPORTANT]
假設客戶端下載帶寬很大，所以不考慮$\frac{F}{d_{min}}$

這邊分析較為複雜，因為上傳的部分無法清晰的劃分server和client，對於下面這個數學式server和client上傳的帶寬彼此交織在一起
$$
\frac{NF}{u_s + u_1 + \cdots + u_N}
$$
在討論極端情況前我們先分析$\frac{F}{u_s}$、$\frac{NF}{u_s + u_1 + \cdots + u_N}$這兩個數學式所代表含意 : 
- $\frac{F}{u_s}$ : 從伺服器傳一個檔案的時間。
- $\frac{NF}{u_s + u_1 + \cdots + u_N}$ : 全部節點上傳N份檔案的時間。
>[!QUESTION] : 為何上傳檔案要區分這兩時間
>請求檔案存在伺服器中，客戶端連之前不知道這檔案，所以需要從伺服器拿到該檔案，所以至少要時間$\frac{F}{u_s}$，把該檔案從伺服器傳到網路上。另一方面客戶端可以再拿到檔案的一部份後，藉由其上載能力，把該部分傳到其他客戶端(檔案可以拆成多份，然後彼此傳)。
## 極端情況分析
### 情況一:伺服器傳一個檔案的時間大於等於全部節點上傳N份檔案的時間($\frac{F}{u_s} \geq \frac{NF}{u_s + u_1 + \cdots + u_N}$)
>[!Thinking Logic]
>伺服器傳一個檔案的時間為瓶頸，所以他應該專注於將該檔案傳到客戶端中，而客戶端負責把其餘$N-1$個檔案丟到網路中。每個client上載速率不同，伺服器應該給上載速率更大的客戶端更多資料，它才能分享更多檔案到網路中(要有該資料才能分享)，寫個更數學一點，對於client i來說它應該拿到$\frac{F}{U}u_i, U = u_1+\cdots+u_N$ 份檔案。對於這分法為何最佳，可以用反證法加以說明，如果一個client j來的比它對應的$\frac{F}{U}u_j$少必有client k拿得比$\frac{F}{U}u_k$多，那時間必然會增加，更精確的時間分析請看後面。

物理條件分析 : 
1. 對於server端 : 上載帶寬不能大於$u_s$
$$
上載帶寬 =  \sum^N_{i=1}\frac{u_i}{U}u_s = u_s
$$
	 從上式看出把server上載性能榨乾
 2. 對於client端上載 : 上載速率不能超過自己的上載速率
 因為從服務器拿到$\frac{u_i}{U}u_s$帶寬，所以如果能像其他client總共提供$(N-1)\frac{u_i}{U}u_s$帶寬即能把得到的資料都同時給出去，盡完他分配的責任
 以知不等式
$$
 N u_s \leq u_s + U \Rightarrow (N-1)u_s \leq U \Rightarrow \frac{U}{N-1} \geq u_s
$$
所以$(N-1)\frac{u_i}{U}u_s$上界為
$$
(N-1)\frac{u_i}{U}u_s \leq (N-1)\frac{u_i}{U}\frac{U}{N-1} = u_i
$$
符合client端上載速率的要求。
 3. 對於client端下載 : 假設$d_{min}$很大所以不考慮。
 >[!Time Analysis]
 >1. 對於server來說(他只傳遞原檔案)
>$$
\text{time\_{server}} = \frac{F}{u_s}
>$$
>
>1. 對於client i來說
>$$
\text{time\_{client}} = \left((N-1)F\frac{u_i}{U}\right) \times \frac{1}{u_i} = \frac{(N-1)F}{U} \leq \frac{F}{u_s}
>$$
>從上面式子來看client得上傳時間是independent on client為一個固定值，所以只要有人比上面分配的多必然使client的用時增加。

總時長為$\frac{F}{u_s}$
### 情況二:伺服器傳一個檔案的時間小於等於全部節點上傳N份檔案的時間($\frac{F}{u_s} \leq \frac{NF}{u_s + u_1 + \cdots + u_N}$)
>[!Thinking logic]
>現在的瓶頸是全部節點上傳N份檔案的時間，所以這裡的伺服器要分擔部分上傳N份檔案的責任。這情況的想法是利用滿各客戶端的上傳帶寬加上伺服器的帶寬，對於每個客戶端要傳它有的檔案到其他N-1個客戶端，所以伺服器到它的帶寬是$\frac{u_i}{N-1}$，即用多少拿多少，而伺服器處理剩下的部分

客戶端總拿到的帶寬 : 
$$
\frac{u_1 + u_2 + \cdots + u_N}{N-1}
$$
伺服器除去客戶端剩下的帶寬 : 
$$
u_s - \frac{u_1 + u_2 + \cdots + u_N}{N-1} = \frac{Nu_s - u_s - U}{N-1} \geq 0
$$
這邊的大於零來自於限制
$$
\frac{1}{u_s} \leq \frac{N}{u_s+U} \Rightarrow (N-1)u_s \geq U
$$
這0非常關鍵，如果為負代表分配的帶寬超過伺服器的上傳帶寬。
>[!IMPORTANT] 怎樣去思考$(N-1)u_s \geq U$
>其實換成以下式子本質就清晰了
>$$
u_s \geq \frac{U}{N-1} = \frac{u_1}{N-1} + \cdots + \frac{u_N}{N-1}
>$$
>伺服器上傳的速率大於客戶端上傳能力總和的上限，所以伺服器可以跳下來幫助傳播。這邊要稍微思考
>$$
\frac{u_j}{N-1} \text{ 的本質}
>$$
>直觀來看是把client j上傳的頻寬平分給剩餘N-1客戶，而這背後隱藏一件事是，我終究要傳遞我得到的數據，而傳遞數據時平均的傳遞給客戶是好的。
物理條件分析 : 
1. 對於server端 : 上載帶寬不能大於$u_s$
顯然符合
 2. 對於client端上載 : 上載速率不能超過自己的上載速率
 因為從服務器拿到$\frac{u_i}{N-1}$帶寬小於$u_I$所以符合。
 3. 對於client端下載 : 假設$d_{min}$很大所以不考慮。

>[!Time Anaylsis(WRONG VERSION)] 
>1. 對於server來說 : 
>$$
\text{time\_{server}} = \left(F + (N-1)F\frac{u_s - \frac{U}{N-1}}{u_s} \right) \times \frac{1}{u_s} = \frac{F}{u_s} \left( \frac{Nu_s - U}{u_s} \right)
>$$
>
>2. 對於client i來說
>$$
\text{time\_{client}} = (N-1)F\frac{\frac{u_i}{N-1}}{u_s} \times \frac{1}{u_i} = \frac{F}{u_s}
>$$

上面的分析低估了客戶端的上傳能力，他不只接收了
$$
(N-1)F\frac{\frac{u_i}{N-1}}{u_s}
$$
可以想成他在$\frac{F}{u_s}$後還繼續發揮餘熱畢竟時間還沒到。server要把檔案傳到客戶端才有他們發揮餘熱的空間，而情況一因為server把檔案傳到客戶端是瓶頸所以沒客戶端發揮餘熱的空間。
>[!Time Anaylsis(CORRECT VERSION)] 
>直接思考peer j拿到一個檔案要多久，有三個上載帶寬
>1. server傳$\frac{u_j}{N-1}$
>2. server剩餘上載帶寬$\frac{u_s - \frac{U}{N-1}}{N}$
>3. 從其他peer來的上載帶寬$\sum^N_{i=1, i \neq j}\frac{u_i}{N-1}$
>總共得到上載帶寬
>$$
\frac{u_j}{N-1} + \frac{u_s - \frac{U}{N-1}}{N} + \sum^N_{i=1, i \neq j}\frac{u_i}{N-1} = \frac{u_s}{N} + \frac{U}{N-1}(1-\frac{1}{N}) = \frac{u_s + U}{N}
>$$
>總時間為
>$$
F \div \frac{u_s + U}{N} = \frac{FN}{u_s + U}
>$$
