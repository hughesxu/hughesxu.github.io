---
title:  "背包问题(Knapsack Problem)"
date:   2020-06-22 9:00:00 +0800
categories: [Algorithm, DP_Greedy]
tags: [DP, Greedy]
---

## 背包问题

&emsp;背包问题是一类经典的动态规划问题，也存在几种变形，不同的变形可以采用的方法也略有差别，比如分数背包问题可以采用贪心算法，而0-1背包问题只能采用动态规划的方法。


&emsp;**背包问题基本描述**：给定一组`N`个物品(`1 <= ii <= N`)，每种物品都有自己的重量`w[ii]`和价值`v[ii]`，在限定的总重量`W`内，我们如何选择才能使所得物品的总价值最高。


&emsp;经典的背包问题有两种：0-1背包问题和分数背包问题。  
&emsp;(1) **0-1背包问题(0-1 knapsack porblem)**：在选择物品时，我们要么完整地选择它，要么不选择。  
&emsp;(2) **分数背包问题(fractional knapsack problem)**：在选择物品时，可以只拿走物品的一部分，而不必须是全部。

&emsp;两个问题都具有最优子结构性质。对于0-1背包问题，考虑重量不超过`W`而价值最高的装包方案。如果将`jj`从方案中删除，则剩余商品必须是重量不超过`W-w[jj]`的价值最高的方案。对于分数背包问题也类似，当做出一个选择后，其剩余商品也必须构成在剩余可承受重量中的价值最高的方案。

&emsp;下面总结各类背包的程序求解，在此之前先定义程序的公共部分，包括一些基本函数的定义与声明。  

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

/*********************************************************************************************
 * Common definition for Knapsack Problem
 *********************************************************************************************/
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
&emsp;对于分数背包问题，由于每次选择物品时，可只选择物品的一部分，可以通过贪心算法求解。在每次选择每磅价值`v[ii]/w[ii]`最高的商品，这样能保证最终的价值最高。  
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

### 0-1 背包问题  
&emsp;对于0-1背包问题无法套用上述贪心算法求解，以上述的 3 种物品举反例说明（假设背包最大重量为`50`）：如果同样优先选择单元重量价值最高的物品，则会先选择物品2，再选择物品1，此时背包只能容纳重量为`20`的物品，无法继续选择，获得的物品总价值为`160`。这并不是最优的选择，可以通过选择物品1和物品3得到一个总价值为`220`的更优解，这也就证明同样的贪心算法对此类问题无效。

&emsp;此类问题同样具有最优子结构性质，考虑采用动态规划的方法求解，尝试通过刻画最优子结构得到状态转移方程。  
&emsp;定义状态`dp`: `dp[ii][jj]表示 将前`ii`件物品放入限重为`jj`的背包可获得的最大价值，0 <= ii <= N, 0 <= jj <= W`。

&emsp;特别地，对于前`0`件物品放入背包中可获得的最大价值为`0`，对于前`ii`件物品放入限重为`0`的背包可获得的最大价值也为`0`，因此可以初始化`dp[0][0...W] = 0`，`dp[0...N][0] = 0`。当`ii > 0`时对第`ii`件物品考虑两种情况：  
* 不装入第`ii`件物品，此时最大价值从前`ii-1`件物品中选，即`dp[ii-1][jj]`；
* 装入第`ii`件物品(`jj >= w[ii]`)，此时仍有重量`jj-w[ii]`可以在前`ii-1`件物品中选择，即`dp[ii-1][jj-w[ii]] + v[ii]`。  
&emsp;即状态转移方程可以表达为：  
```
dp[ii][jj] = max(dp[ii-1][jj], dp[ii-1][ jj-w[ii] ] + v[ii]);   // if jj >= w[ii]
dp[ii][jj] = dp[ii-1][jj];                                      // if jj < w[ii]
```


1. 先利用**带备忘的自顶向下方法**进行求解，求解程序展示如下。  


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
                dp_0_1_recursive(goods, dp, ii-1, jj-goods[ii-1].weight) + goods[ii-1].value);
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
     * Note: If the problem is asking how to make knapsack exactly full, 
     *      then do not need to initialize dp[0][jj] with ZERO
     */
    for (int jj = 0; jj < col+1; jj++)
        dp[0][jj] = 0;

    return dp_0_1_recursive(goods, dp, row, WW);
}
```

2. 再尝试**自底向上法**，从最小的子问题开始向大的子问题逐步迭代求解。
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

&emsp;采用自底向上法求解时发现，`dp[ii][jj]`只依赖于`dp[ii-1][jj]`和`dp[ii-1][ jj-w[ii] ] + v[ii]`，通常情况下我们可以利用滚动数组对其进行空间优化，将二维数组优化为一维数组。

&emsp;由于`dp[ii][jj]`依赖于`dp[ii-1][jj-w[ii]]`，相当于当前层`ii`对于`dp`的计算依赖于上一层`ii-1`的前部分计算值，为了避免值的覆盖，只能采用**逆向枚举**。举例说明，将问题简化为：`dp[ii][jj]`依赖于`dp[ii-1][jj-1]`，如果采用正向枚举，`ii`从`0`逐步枚举到`ii-1`时，`dp[ii-1]`的值被更新覆盖，而`dp[ii]`依赖的是被覆盖前的`dp[ii-1]`值，将导致`dp[ii]`计算值有误。为了避免前半部分依赖值被覆盖，采用逆向枚举，这样`dp[ii-1]`总在`dp[ii]`之后才被更新，解决了值覆盖问题。  

```
// Method: bottom-up, with Rotate Array
int knapsack_0_1_bp_rotate_array(vector<Goods> &goods, const int WW)
{
    int row = (int)goods.size();
    int col = WW;
    vector<int> dp(col+1, 0);               // Two-dimensional array, dp[ii][ww]

    for (int ii = 1; ii < row+1; ii++) {
        /* If jj range from 0 to col, previous one and present one will impact each other.
         * Further, the second half of jj will be impacted by the first half of jj.
         *
         *      dp[ii][jj]  < -- > dp[ii-1][jj - goods[ii-1].weight] + goods[ii-1].value
         * Why: From equation, current row value is related to last row value, after 
         *      compressed to one row, last row value will be overwitten if sequentially enumerate.
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

### 完全背包问题  
&emsp;完全背包问题(Unbounded Knapsack Problem)是0-1背包问题的一个变形，其区别在于：每种物品有无限多个。最优子结构（状态转移方程）的刻画和0-1背包问题极为类似。采用同样的状态`dp`来描述，初始状态也完全一致。  
&emsp;不同之处在于，当`ii > 0`时对第`ii`件物品考虑两种情况：  
* 不装入第`ii`件物品，此时最大价值从前`ii-1`件物品中选，即`dp[ii-1][jj]`；
* 装入第`ii`件物品(jj >= w[ii])，此时仍有重量`jj-w[ii]`仍可以在前`ii`件物品中选择，即`dp[ii][jj-w[ii]] + v[ii]`。  
&emsp;即状态转移方程可以表达为：  
```
dp[ii][jj] = max(dp[ii-1][jj], dp[ii][ jj-w[ii] ] + v[ii]);   // if jj >= w[ii]
dp[ii][jj] = dp[ii-1][jj];                                    // if jj < w[ii]
```

&emsp;采用**自底向上法**求解的程序展示如下，同样提供了空间优化版本。值得注意的是，对于此类问题，`dp[ii][jj]`依赖的是上层同位置的值`dp[ii-1][jj]`和本层前部分的值`dp[ii][jj-w[ii]]`，因此只能采用**正向枚举**的方式进行空间优化，这样才能将前部分的更新值及时带入到当前计算值中。 
 
```
// Method: bottom-up, with Two-dimensional array
int knapsack_unbounded_bottom_up(vector<Goods> &goods, const int WW)
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
                        dp[ii][jj - goods[ii-1].weight] + goods[ii-1].value);
            }
            dp[ii][jj] = max_value;
            printf("dp[%d][%d] = %d\n", ii, jj, max_value);
        }
    }
    return dp[row][col];
}

// Method: bottom-up, with Rotate Array
int knapsack_unbounded_bp_rotate_array(vector<Goods> &goods, const int WW)
{
    int row = (int)goods.size();
    int col = WW;
    vector<int> dp(col+1, 0);               // Two-dimensional array, dp[ii][ww]

    for (int ii = 1; ii < row+1; ii++) {
        for (int jj = 0; jj <= col; jj++) {
            /*
             *      dp[ii][jj]  < -- > dp[ii][jj - goods[ii-1].weight] + goods[ii-1].value
             * Why: dp[ii][jj] <--> dp[ii][jj-W], latter value related to previous value of this row
             * How: Sequential enumeration, front to end.
             */
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


### 多重背包问题  
&emsp;多重背包问题(Bounded Knapsack Problem)是0-1背包问题的另一个变形，其区别在于：每种物品是有限多个n[ii]。最优子结构（状态转移方程）的刻画和0-1背包问题类似。采用同样的状态`dp`来描述，初始状态也完全一致。状态转移方程的区别在于：对于每件物品要在有限多个可选项中选择一个最优的数量。  
&emsp;即状态转移方程可以表达为：  

```
// kk为装入第ii种物品的件数, kk <= min(n[ii], jj/w[ii])
dp[ii][jj] = max{(dp[ii-1][jj − kk*w[ii]] + kk * v[i]) for every kk}
```

&emsp;同样地，给出**自底向上法**求解的一般版本和空间优化版本, 和0-1背包类似，只能采用**逆向枚举**。  
```
// Method: bottom-up, with Two-dimensional array
int knapsack_bounded_bottom_up(vector<Goods> &goods, const int WW)
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
            for (int kk = 0; kk <= min(goods[ii-1].numbers, jj / goods[ii-1].weight); kk++) {
                max_value = max( max_value,
                        dp[ii-1][jj - kk * goods[ii-1].weight] + kk * goods[ii-1].value);
            }

            dp[ii][jj] = max_value;
            printf("dp[%d][%d] = %d\n", ii, jj, max_value);
        }
    }
    return dp[row][col];
}

// Method: bottom-up, with Rotate Array
int knapsack_bounded_bp_rotate_array(vector<Goods> &goods, const int WW)
{
    int row = (int)goods.size();
    int col = WW;
    vector<int> dp(col+1, 0);               // Two-dimensional array, dp[ii][ww]

    for (int ii = 1; ii < row+1; ii++) {
        /* If jj range from 0 to col, previous one and present one will impact each other.
         * Further, the second half of jj will be impacted by the first half of jj.
         *
         *      dp[ii][jj]  < -- > dp[ii-1][jj - goods[ii-1].weight] + goods[ii-1].value
         * Why: From equation, current row value is related to last row value, after 
         *  compressed to one row, last row value will be overwitten if sequentially enumerate.
         * How: Avoid current row value being overwritten, use reverse enumerate.
         */
        for (int jj = col; jj >= 0; jj--) {
            for (int kk = 0; kk <= min(goods[ii-1].numbers, jj / goods[ii-1].weight); kk++) {
                dp[jj] = max(dp[jj],
                        dp[jj - kk * goods[ii-1].weight] + kk * goods[ii-1].value);
            }
            printf("dp[%d][%d] = %d\n", ii, jj, dp[jj]);
        }
    }

    return dp[col];
}
```
