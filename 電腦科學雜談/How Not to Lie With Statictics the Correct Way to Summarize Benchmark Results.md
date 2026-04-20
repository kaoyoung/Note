### Ref: [How not To Lie With Statictics The Correct Way to Summarize Benchmark Results](https://dl.acm.org/doi/pdf/10.1145/5666.5673)
# Rule 1: Do Not Use the Arithmetic Mean to Average Nomarlied Numbers
Consider a scenario where machine X is twice as fast as the machine Y on benchmark A , but half as fast on Benchmark B. Intuitively, these two machines possess equal overall performance. However once we normalize the result w.r.t the machine X or Y and calculate the arithmetic mean, the results become inconsistent. This occurs because the arithmetic mean disproportionately weights performance ratios; a $2\times$ speedup increases the sum by $1.0$, whereas being half as fast only decreases it by $0.5$. Consequently, summing normalized numbers yields contradictory conclusions depending on which machine is chosen as the baseline, as demonstrated in the figures below.

![[Normalized problem.png]]

>The sum of nor- malized numbers is also meaningless.
# Rule 2: Use the Geometric Mean to Average Normalized Numbers

If machine X is $t$ time as fast as the machine Y on benchmark A , but $\frac{1}{t}$ time as fast on Benchmark B, the geometic mean will give these two machines a same number. Let's consider the case below

![[Geometric Mean.png]]

> Note that the data in calculating GM is have been rounded.

In table VII
- Machine M is 14% fater than Machine R 
- Machine Z is 16% fater than Machine R 
In table VIII
- Machine M is 14% fater than Machine R 
- Machine Z is 16% fater than Machine R

>1. The geometric mean can be used regardless of how the numbers are **normalized**.
>2. The geometric mean can be used euen if the numbers are **not normalized**; the resulting means can then be normalized.
# Rule 3: Use the Sum (or arithmetic mean) of Raw, Unnormalized Results whenever This "Total" Has Some Meaning
Sometimes, the sum of benchmark results has meaning: for example, **total run time for a set of benchmarks**, However, it is important to calculate,this **raw, unnormalized** sum using data since we have shown that summing (or taking the arithmetic mean) of normalized numbers gives worthless results.

>[!Note]
>We may give different weight to different benchmark to mimic the workload in reality. **Note that these benchmark can't be normalied.**


# A Proof That the Geometric Mean Is the Only Correct Average of Normalized Measurements

First we formulate the problem 
>[!question] The problem
>Let $A = f(a_1, \cdots, a_n)$. Since $A$ is an unweighted expected value or mean, the function $f$ must satisfy the following three properties.
 >1. Reflexive Property: $f(a,a, \cdots, a) = a$
>2. Symmetric Property: $f(a_1, \cdots, a_n) = f(a_{\sigma(1)}, \cdots, a_{\sigma(n)})$ for all permutation $\sigma$ of the number $1, \cdots, n$.
>3. Multiplicative Property: $f(a_1 b_1, \cdots, a_n b_n) = f(a_1, \cdots, a_n) f(b_1, \cdots, b_n)$

>[!Note]
>This problem just illustrates our intuition about evaluting benchmark performance
>1. property 1: If a machine is $a$ times as fast as reference manchine across all benchmarks, its overall average performance is also exactly $a$ times as fast.
>2. property 2: the order of benchmark won't effect the result.
>3. property 3: Changing the baseline or reference machine used for normalization won't distort the relative performance comparisons between machines.

