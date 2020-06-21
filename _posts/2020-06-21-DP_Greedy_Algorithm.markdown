---
title:  "动态规划(Dynamic Programming)和贪心算法(Greedy Algorithm)"
date:   2020-06-21 19:00:00 +0800
categories: [Algorithm]
tags: [DP, Greedy]
---

&emsp;**动态规划(Dynamic Programming)**和**贪心算法(Greedy Algorithm)**都常用来解决最优化问题(Optimization problem)，这类问题有许多可行解，每个解都有一个值，希望寻找具有最优值的解，此解往往是一个最优解，而不是最优解，因为可能有多个解都能使问题达到最优值。


&emsp;动态规划和贪心算法在求解过程上有区别，但也有联系，比如求解的问题都要满足最优子结构(Optimal Substructure)性质。  
&emsp;**最优子结构性质**：*问题的最优解由相关子问题的最优解组合而成，而这些子问题可以独立求解。*

## 动态规划 

&emsp;动态规划和分治方法相似，都是通过组合子问题的解来求解原问题。不同的是，动态规划应用于子问题重叠的情况，即不同的子问题具有公共的子子问题。此时，动态规划对每个子子问题只会求解一次，并在表中保存，可以避免不必要的重复计算。

 
&emsp;***问题要素：最优子结构性质，子问题重叠性质***

### 基本设计步骤  
1. 刻画一个最优解的结构特征，即最优子结构特征。
2. 递归地定义最优解的值，得到状态转移方程（递归表达式）。
3. 计算最优解的值，通常有两种方法：带备忘的自顶向下法(top-down with memoization)和自底向上法(bottom-up method)。
4. 利用计算出来的信息构造一个最优解。
 

### 最优子结构  
&emsp;刻画最优子结构通常遵循如下的通用模式：  
1. 证明问题最优解的第一个组成部分是做出一个选择，这便产生一个或多个待解的子问题。  
2. 对于一个给定问题，在其可能的第一步选择中，假定已经知道哪种选择才会得到最优解。但现在不关心选择是如何得到的，只假定已经知道了这种选择。  
3. 给定可获得最优解的选择后，确定这次选择会产生哪些子问题，以及如何更好地刻画子问题空间。  
4. 利用“剪切-粘贴”(`cut-and-paste`)技术证明：作为构成原问题最优解的组成部分，每个子问题的解就是它本身的最优解。证明这一点常采用反证法。

 

### 子问题重叠  
&emsp;子问题重叠性质要求，利用动态规划方法求解最优解过程中，递归算法会反复求解相同的子问题，而不是不停地生成新的子问题。这也是动态规划和分治方法求解中的一个区别。


### 常见问题  
&emsp;**最优钢条切割问题**，**矩阵链乘法问题**，**最长公共子序列问题(Longest-Common-Subsequence problem)**，**最优二叉查找树**，**背包问题**。

 

## 贪心算法

&emsp;贪心算法在求解的每一步都做出局部的贪心选择，寄希望这样的选择能够产生全局最优解。值得注意的是，贪心算法并不保证得到最优解，但针对很多问题确实能得到最优解。  


&emsp;***问题要素：最优子结构性质，贪心选择性质。***



### 基本设计步骤  
1. 将最优化问题转化为这样的形式：对其做出一次选择后，只剩下一个子问题需要求解。  
2. 证明做出最优选择后，原问题总是存在最优解，即贪心选择总是安全的。  
3. 证明做出贪心选择后，剩余的子问题满足性质：其最优解与贪心选择组合即可得到原问题的最优解，这样就得到了最优子结构。  


### 贪心选择性质  
&emsp;贪心选择性质(greedy-choice property): 可以通过做出局部最优（贪心）选择来构造全局最优解。

### 常见问题与算法 
&emsp;贪心算法典型的用来解决的问题，包括**活动选择问题**，**分数背包问题**。  
&emsp;基于贪心方法设计的算法，包括**`Huffman`编码**，**单源最短路径的`Dijsktra`算法**，**最小生成树(`minimum-spanning-tree`)算法(`Kruskal`算法和`Prim`算法)**。  

## 动态规划与贪心算法的区别和联系 
1. 动态规划与贪心算法求解的问题都必须具备最优子结构性质。
2. 贪心算法本质上是对动态规划的优化，对每个用贪心算法求解的问题，几乎也存在一个动态规划的解法。
3. 动态规划方法中，每个步骤要进行一次选择，但选择通常依赖于子问题的解，因此通常使用自底向上的方法，先求解较小子问题，再根据子问题的解做出选择。而贪心算法在进行第一次选择之前不求解任何子问题，通常采用自顶向下的方法，进行一次又一次选择，将给定问题实例变小。  
 

## 背包问题

&emsp;背包问题是一类经典的动态规划问题，也存在几种变形，不同的变形可以采用的方法也略有差别，比如分数背包问题可以采用贪心算法，而0-1背包问题只能采用动态规划的方法。


&emsp;**背包问题基本描述**：给定一组`N`个物品(`1 \leq i \leq N`)，每种物品都有自己的重量`wi`和价值`vi`，在限定的总重量`W`内，我们如何选择才能使所得物品的总价值最高。


&emsp;经典的背包问题有两种：0-1背包问题和分数背包问题。  
&emsp;**0-1背包问题(0-1 knapsack porblem)**：在选择物品时，我们要么完整地选择它，要么不选择。  
&emsp;**分数背包问题(fractional knapsack problem)**：在选择物品时，可以只拿走物品的一部分，而不必须是全部。

&emsp;两个问题都具有最优子结构性质。对于0-1背包问题，考虑重量不超过`W`而价值最高的装包方案。如果将`j`从方案中删除，则剩余商品必须是重量不超过`W-wj`的价值最高的方案。对于分数背包问题也类似，当做出一个选择后，其剩余商品也必须构成在剩余可承受重量中的价值最高的方案。

&emsp;下面总结各类背包的程序求解，在此之前先定义程序的公共部分，包括一些基本函数的定义与声明。  

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

/************************************************************************************************
 * Common definition for Knapsack Problem
 ************************************************************************************************/
// struct to store information of every item
typedef struct {
    int     numbers;            // Total amount of one item
    int     value;              // Value of one item
    int     weight;             // Weight of one item
    double  avg_v;              // Value per Unit Weight of one item
} Goods;

// Pre-define three types of items
vector<Goods> goods = { {3, 100, 20, 5.0}, 
                        {10, 60, 10, 6.0}, 
                        {2, 120, 25, 4.0} };

// Compare function for sorting, sort by value per weight in non-increasing order
int comp(const Goods &aa, const Goods &bb)
{
    return (aa.avg_v > bb.avg_v);
}

// Return maximum of two values
int max(int aa, int bb)
{
    if (aa >= bb)
        return aa;
    else
        return bb;
}

// Return minimum of two values
int min(int aa, int bb)
{
    if (aa <= bb)
        return aa;
    else
        return bb;
}
```

### 分数背包问题  
&emsp;对于分数背包问题，由于每次选择物品时，可只选择物品的一部分，可以通过贪心算法求解。在每次选择每磅价值`vi/wi`最高的商品，这样能保证最终的价值最高。  
&emsp;为了能够使用贪心算法，一般地，我们要证明问题满足最优子结构性质和贪心选择性质。此问题的最优子结构性质在上文中已有简单的论述。对于贪心选择性质，即要证明每次做出的贪心选择能够构成原问题全局最优解，详细论证可参考博主的文章：  [证明分数背包问题具有贪心选择性质](https://blog.csdn.net/u010339647/article/details/50588499/)

```
int knapsack_greedy(vector<Goods> &goods, int WW)
{
    int max_value = 0;
    int remain_w = WW;
    int index = 0;

    // Sort all items in non-increasing order based on Value per Unit Weight.
    sort(goods.begin(), goods.end(), comp);

    while (remain_w) {
        if (remain_w >= goods[index].weight) {
            remain_w -= goods[index].weight;
            max_value += goods[index].value;
            index++;
        } else {
            max_value += remain_w * goods[index].avg_v;
            remain_w = 0;
        }
    }

    return max_value;
}
```

### 0-1背包问题  
&emsp;易发现，对于0-1背包问题无法套用上述贪心算法求解，以上述的 3 种物品举反例说明（假设背包最大重量为`50`）：如果同样优先选择单元重量价值最高的物品，则会先选择物品2，再选择物品1，此时背包只能容纳重量为`20`的物品，无法继续选择，获得的物品总价值为`160`。这并不是最优的选择，可以通过选择物品1和物品3得到一个总价值为`220`的更优解，这也就证明同样的贪心算法对此类问题无效。

&emsp;此类问题同样具有最优子结构性质，考虑采用动态规划的方法求解，尝试通过刻画最优子结构得到状态转移方程。  
&emsp;定义状态`dp`: `dp[ii][jj]表示 将前`ii`件物品放入限重为`jj`的背包可获得的最大价值，0 <= ii <= N, 0 <= jj <= W`。

&emsp;特别地，对于前`0`件物品放入背包中可获得的最大价值为`0`，对于前`ii`件物品放入限重为`0`的背包可获得的最大价值也为`0`，因此可以初始化`dp[0][0...W] = 0`，`dp[0...N][0] = 0`。当`ii > 0`时对第`ii`件物品考虑两种情况：  
* 不装入第`ii`件物品，此时最大价值从前`ii-1`件物品中选，即`dp[ii-1][jj]`；
* 装入第`ii`件物品，此时仍有重量`jj-w[ii]`可以在前`ii-1`件物品中选择，即`dp[ii-1][jj-w[ii]] + v[ii]`。  
&emsp;即状态转移方程可以表达为：  
```
dp[ii][jj] = max(dp[ii-1][jj], dp[ii-1][ jj-w[ii] ] + v[ii]);   // if jj >= w[ii]
dp[ii][jj] = dp[ii-1][jj];                                      // if jj < w[ii]
```
&emsp;先利用带备忘的自顶向下方法进行求解，求解程序如下：  


```
// Method: top-down with memoization, Recursive
int dp_0_1_recursive(vector<Goods> &goods, vector<vector<int> > &dp, int ii, int jj)
{
    int max_value = INT_MIN;

    printf("ii = %d / jj = %d\n", ii, jj);
    if (dp[ii][jj] >= 0 || 0 == ii || 0 == jj)
        return dp[ii][jj];

    if (goods[ii-1].weight > jj) {              // iith item
        max_value = dp_0_1_recursive(goods, dp, ii-1, jj);
    } else {
        max_value = max( dp_0_1_recursive(goods, dp, ii-1, jj),
                dp_0_1_recursive(goods, dp, ii-1, jj - goods[ii-1].weight) + goods[ii-1].value );
    }
    dp[ii][jj] = max_value;
    printf("[%d, %d] = %d\n", ii, jj, dp[ii][jj]);

    return max_value;
}

int knapsack_0_1_rec(vector<Goods> &goods, const int WW)
{
    int row = (int)goods.size();
    int col = WW;
    vector<vector<int> > dp(row+1);               // Two-dimensional array, dp[ii][ww]
    for (int ii = 0; ii < row+1; ii++)
        dp[ii].assign(col+1, INT_MIN);

    for (int ii = 0; ii < row+1; ii++)
        dp[ii][0] = 0;
    /*
     * Note: If the problem is asking how to make knapsack exactly full, then do not need to initialize dp[0][jj] with ZERO
     */
    for (int jj = 0; jj < col+1; jj++)
        dp[0][jj] = 0;

    return dp_0_1_recursive(goods, dp, row, WW);
}
```

&emsp;再尝试自底向上法，迭代求解。
```
// Method: bottom-up, with Two-dimensional array
int knapsack_0_1_bottom_up(vector<Goods> &goods, const int WW)
{
    int row = (int)goods.size();
    int col = WW;
    vector<vector<int> > dp(row+1);               // Two-dimensional array, dp[ii][ww]
    for (int ii = 0; ii < row+1; ii++)
        dp[ii].assign(col+1, INT_MIN);

    for (int ii = 0; ii < row+1; ii++)
        dp[ii][0] = 0;
    for (int jj = 0; jj < col+1; jj++)
        dp[0][jj] = 0;

    for (int ii = 1; ii < row+1; ii++) {
        for (int jj = 1; jj < col+1; jj++) {
            int max_value = INT_MIN;
            if (goods[ii-1].weight > jj) {
                max_value = dp[ii-1][jj];
            } else {
                max_value = max(dp[ii-1][jj],
                        dp[ii-1][jj - goods[ii-1].weight] + goods[ii-1].value);
            }
            dp[ii][jj] = max_value;
            printf("dp[%d][%d] = %d\n", ii, jj, max_value);
        }
    }
    return dp[row][col];
}
```

&emsp;采用自底向上法求解时发现，`dp[ii][jj]`只依赖于`dp[ii-1][jj]`和`dp[ii-1][ jj-w[ii] ] + v[ii]`，通常情况下我们可以利用滚动数组对其进行空间优化，将二维数组优化为一维数组。由于`dp[ii][jj]`依赖于`dp[ii-1][jj-w[ii]]`，相当于当前行对于`dp`的计算依赖于上一层的前部分计算值，为了避免值得覆盖，只能采用逆向枚举。

```
// Method: bottom-up, with Rotate Array
int knapsack_0_1_bp_rotate_array(vector<Goods> &goods, const int WW)
{
    int row = (int)goods.size();
    int col = WW;
    vector<int> dp(col+1, 0);               // Two-dimensional array, dp[ii][ww]

    for (int ii = 1; ii < row+1; ii++) {
        /* If jj range from 0 to col, previous one and present one will impact each other.
         * Further, the second half of jj will be impacted by the first half of jj when ii is consistent.
         *
         *      dp[ii][jj]  < -- > dp[ii-1][jj - goods[ii-1].weight] + goods[ii-1].value
         * Why: From equation, current row value is related to last row value, after compressed to one row, 
         *      last row value will be overwitten if sequentially enumerate.
         * How: Avoid current row value being overwritten, use reverse enumerate.
         */
        for (int jj = col; jj >= 0; jj--) {
            if (goods[ii-1].weight > jj) {
                dp[jj] = dp[jj];
            } else {
                dp[jj] = max(dp[jj],
                        dp[jj - goods[ii-1].weight] + goods[ii-1].value);
            }
            printf("dp[%d][%d] = %d\n", ii, jj, dp[jj]);
        }
    }

    return dp[col];
}
```
