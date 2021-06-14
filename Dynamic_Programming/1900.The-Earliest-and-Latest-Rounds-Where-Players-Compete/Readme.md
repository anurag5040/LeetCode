### 1900.The-Earliest-and-Latest-Rounds-Where-Players-Compete

#### 解法1：DP
本题的数据规模很小，可以考虑直接DP暴力。我们令dp1[n][a][b]，表示当有n个选手、其中a和b是最强的两位选手时，最少需要多少轮对决能让a与b对决。相应的，令dp2[n][a][b]表示最多需要多少轮次。

为了充分利用对称性，减少需要填充的dp值，我们约定：
1. 让```a<b```。即如果 a>b时，```return dp[b][a]```
2. 让ab整体偏向数组的左边。即如果 ```(a + b)/2.0 > (n + 1)/2.0```（你可以认为ab的中轴线超过了整体的中轴线），```return dp[n+1-b][n+1-a]```. 注意n+1-b是b的对称点，n+1-a是a的对称点。

我们分两种情况讨论。

第一种，a和b分别在中轴线两边，根据之前的约定，a会比b更靠近端点：
```
(L) XXXX a XX b' XXXX M XXXX b XX a' XXXX (R)
```
我们在上面的示意图中标记了a的对称点a'，b的对称点b'，中轴线M（可以是一个元素，也可以是一个虚拟的中轴）。

我们知道a左边的选手可输可赢，假设胜利的人数是x，那么这些人会留在这个区域。另外ab'之间的选手也是可输可赢，假设胜利的人数是y，那么这些人也会留在这个区域。b'因为会与b对决而被淘汰。b'到b之间的区域的选手，会两两对决，留下一半的人，即```z = (b-b'-1+1)/2```. 

我们发现，如果我们约定了x和y之后，剩下的一半人(n+1)/2里面，a排在第x+1位，b排在x+1+y+z+1位，故我们可以递归求解 ```dp((n+1)/2, x+1, x+1+y+z+1)```。遍历所有的x和y之后，我们可以知道dp[n][a][b]的最大值和最小值。

第二种，a和b都在中轴线左边，根据之前的约定，a会比b更靠近端点：
```
(L) XXXX a XX b XXXX M XXXX b' XX a' XXXX (R)
```
这种情况相对更简单。令a左边的区域保留x人，ab中间的区域保留y人，那么就转化为```dp((n+1)/2, x+1, x+1+y+1)```当遍历所有的x和y后，就能知道dp[n][a][b]的最大值和最小值。

最终的答案是dp[n][firstPlayer][secondPlayer].

时间复杂度分析是这样的：状态有三个维度，我们遍历所有的状态，所以是o(N^3)。在计算每一个状态的时候，又是遍历了两层for循环。所以总共是o(N^5)的复杂度。但是更细节地分析的话，对于dp[n][a][b]，第一个维度只有log(N)级别的可能。所以整体的复杂度其实是o(N^4logN).

#### 解法2：贪心
本题是两个月前残酷群的[智慧杯第四场赛题](https://wisdompeak.github.io/lc-score-board/cup.html). 原题对数据规模的要求更加高，n要求1e18. 显然必须要有log(N)的解法。

##### 最早轮次
如果a和b在中轴线的两侧，那么通常情况下，经过设计，只要经过一轮，a和b就能够中轴对称。为什么呢？
```
(L) XXXX a XX b' XXXX M XXXX b XX a' XXXX (R)
   —————  ———
     x     y
```
在这一轮对局之后，我们只需要a左边的个数等于b右边，就能让二者对称。假设(La)之间有x个元素，只要将x的元素均匀二分到(La)和(a'R)这两个区域即可。如果x是奇数没法平分，那也没关系，我们还有(ab')之间的y个元素，只要拿出一个放在(ba')区域，这样就又能使得(La)和(bR)两部分平衡。

但是我们注意到，当x是奇数，但y等于0的时候，上面的补救措施就无法实现，故需要再多加一轮才能实现a与b的对称。

如果a和b在中轴线的同侧，我们的目标只有一个，就是尽快地让整体的中轴线落在a与b之间（也就是变成上面的情况）。所以我们只需无脑地每个回合人数减半、让前半区选手通赢。
```
(L) XXXX a XX b XXXX M XXXX b' XX a' XXXX (R)
```
直至中轴线恰好落在a与b之间的这一轮，我们通过类似前述的细微调整，优化(La)区域和(ab)区域的人数，就可以直接让a与b对称。

但是同样有一个例外，那就是当n为奇数，且a与b相邻的时候，我们是无论如何都无法让a与b关于一个中轴线对称的，比如```abx```。这种情况下，我们只能继续让选手减半（方法任意，因为a与b总是相邻），直至总人数为偶数，才有可能让ab关于中轴线对称。

##### 最晚轮次
和前一个问题的思想相反，我们希望整体中轴线尽量地远离ab。因为中轴线一旦落入ab中间，那么就很容易让ab对称。所以宗旨就是，让ab尽可能地往左偏。

根据这个精神，当a和b在中轴线两侧时，我们会让(La)和(ab')这两个区间的选手都输，这样a与b就被最大程度地往左偏移，远离了整体的中轴线。
```
(L) XXXX a XX b' XXXX M XXXX b XX a' XXXX (R)
   —————  ———
     x     y
```

当a和b在中轴线同侧时，同样，我们让(La)和(ab)这两个区间的选手都输，这样a与b也是最大程度地往左偏移，远离了整体的中轴线。
```
(L) XXXX a XX b XXXX M XXXX b' XX a' XXXX (R)
```

以上只是贪心策略的大体思想，代码中还有很多细节就不再赘述了。