---
title:  "最长上升子序列(Longest Increasing Subsequence)"
date:   2020-06-29 9:00:00 +0800
categories: [Algorithm, DP_Greedy]
tags: [DP, Greedy]
---

## 最长上升子序列与最长不下降子序列

&emsp;如果某个序列有如下性质：
```
        X = {x1, x2, x3, ..., xn},  满足x1 < x2 < x3 < ... < xn, 即xi < xi+1 (0 <= i <= n-1)
```
这个序列就称之为**上升（递增）序列**。类似地，如果此序列并不是完全升序，即满足`x1 <= x2 <= x3 < ... <= xn`，这样的序列称之为**不下降（非递减）序列**。  

&emsp;**最长上升子序列(`LIS, Longest Increasing Subsequence`)** 或 **最长不下降子序列(`LNDS, Longest Not-decreasing Subsequence`)**问题，就是要在给定的序列中找到上升序列或不下降序列，使得序列长度最长。这两类问题极其相似，针对最长上升子序列问题LIS进行详细求解，并在相应位置给出对于最长不下降子序列LNDS求解所需要的改动。  
&emsp;**最长上升子序列问题基本描述**：给定包含`NN`个元素的原始序列`seq`，求解其最长上升子序列的长度。例子：对于序列`{1, 2, 4, 5, 3, 2, 2}`，其最长上升子序列为`{1, 2, 4, 5}`，长度为4。其最长不下降子序列有2个，长度均为4：`{1, 2, 4, 5}`，`{1, 2, 2, 2}`。  


## 问题分析与求解
### 动态规划
&emsp;最长上升子序列问题，是满足最优子结构性质的（证明步骤作为遗留问题，可以清晰地看到，当分析到`X`中第`ii`个元素时，问题的求解是和其前面的元素`jj (0< jj < ii)`相关的）。考虑采用动态规划的方式进行求解，尝试刻画最优子结构特征来得到状态转移方程。  
&emsp;定义状态`dp`：`dp[ii]表示 以序列X中第ii个元素结尾的LIS的长度`。  
&emsp;初始状态时，以每个元素结尾，初始化LIS的长度为`1`，即`dp[ii] = 1 (0 <= ii < NN)`。从头到尾遍历整个序列，到达序列的任意位置`ii (0 <= ii < NN)`时，遍历该位置前的所有元素`jj (0 <= jj < ii)`。如果`seq[ii] > seq[jj]`，那么可以将`seq[ii]`接在以`seq[jj]`为结尾的LIS后面，形成一个新的LIS；否则，当前以当前位置结束的LIS不变。由此，可以得到状态转移方程：  
```
    dp[ii] = dp[jj] + 1;          // if seq[jj] < seq[ii]
    dp[ii] = 1                    // if seq[jj] >= seq[ii]
```
&emsp;对于最长上升子序列问题，常常只需要给出LIS的长度。更进一步地，尝试能够得到LIS具体的序列。需要注意的时，给定序列的LIS可能会有多个，并且多个LIS存在两种情况：  
* 以同一个元素结尾的LIS可能不止一个。给定序列`{1, 2, 5, 4, 7}`，`{1, 2, 5, 7}`和`{1, 2, 4, 7}`均是LIS，均以`7`为结尾元素。  
* 多个LIS以不同的元素结尾。给定序列`{4, 5, 1, 2}`，`{4, 5}`和`{1, 2}`是其两个以不同元素结尾的LIS。


&emsp;定义嵌套的`vector`容器`vector<vector<vector<int> > > lis`来记录以每个元素结尾的LIS，其中内层的`vector<vector<int> >`用来记录具体的LIS，由于可能有多个，所以也使用了`vector`容器来存储。举例说明：对于给定序列`{1, 2, 5, 4, 7}`的以`7`为结尾元素的LIS，将存储为`lis[4] = { {1, 2, 5, 7}, {1, 2, 4, 7} }`。  
&emsp;具体的求解程序展示如下，由两个循环组成，所以时间复杂度为`n^2`。

 
```
// Time complexity: n^2
int LIS_n_n(vector<int> &seq, int NN)
{
    /*
     * dp[ii] is the sequence with seq[ii] as its last element
     *  if seq[jj] < seq[ii] (jj < ii):           dp[ii] = dp[jj] + 1;
     *  else:                                     dp[ii] = 1
     */
    vector<int> dp(NN, 1);
    int longest = 1;
    // Store index sets with LIS, 0 put as default one
    vector<int> index = {0};
    // lis[ii] store the LIS sequence with seq[ii] as its last element
    vector<vector<vector<int> > > lis;

    // Initialize all lis[ii] (0 <= ii < NN) with its seq[ii] firstly
    for (int ii = 0; ii < NN; ii++) {
        vector<vector<int> > tmp = { {seq[ii]} };
        lis.push_back(tmp);
    }

    for (int ii = 1; ii < NN; ii++) {
        for (int jj = 0; jj < ii; jj++) {
            if (seq[ii] > seq[jj]) {
                if (dp[ii] <= dp[jj] + 1) {
                    if (dp[ii] < dp[jj] + 1)
                        // Clear existing sequence in lis[ii], as longer one appear.
                        lis[ii].clear();        

                    dp[ii] = dp[jj] + 1;

                    vector<vector<int> > local;
                    local.assign(lis[jj].begin(), lis[jj].end());
                    for (unsigned int kk = 0; kk < local.size(); kk++) {
                        local[kk].push_back(seq[ii]);
                        lis[ii].push_back(local[kk]);
                    }
                }
            }
        }
        if (longest <= dp[ii]) {
            if (longest < dp[ii])
                index.clear();
            longest = dp[ii];
            index.push_back(ii);
        }
    }

    printf("Longest Increasing Sequences are:\n");
    for (unsigned int ii = 0; ii < index.size(); ii++) {
        int idx = index[ii];
        for (unsigned int jj = 0; jj < lis[idx].size(); jj++) {
            for (unsigned int kk = 0; kk < lis[idx][jj].size(); kk++) {
                if (kk != 0)
                    printf(" ");
                printf("%d", lis[idx][jj][kk]);
            }
            printf("\n");
        }
    }

    return longest;
}
```
&emsp;对于**最长不下降子序列**问题，只需要第`24`行的`if (seq[ii] > seq[jj])`判断改为`if (seq[ii] >= seq[jj])`即可。  



### 二分法+贪心算法
&emsp;对于最长上升子序列问题，还存在一种时间复杂度为`n*logn`的算法。这里考虑一个贪心的想法：  
```
    对于一个上升子序列，其结尾元素越小，越有利于在后面接其他的元素，上升序列变得更长的可能性就越大。  
```
&emsp;考虑维护一个数组`tail[len]`表示长度为`len`的上升序列中结尾元素的最小值，显然可以知道`tail`数组的值是非递减的。从头至尾遍历整个序列，若：  
* `seq[ii] > tail[len]` (`tail`数组最后的一个元素)，将`seq[ii]`加入到`tail`的最后面，LIS的长度增加1 `(len++)`。  
* `seq[ii] <= tail[len]`，查找`tail`数组中第一个*不小于（大于等于）*`seq[ii]`的数，并用`seq[ii]`替换。由于`tail`数组的非递减性，可以使用二分法查找，将查找时间复杂度降为`log(n)`。  

&emsp;对于采用二分法+贪心算法的求解中对LIS的输出，网上查询未能找到输出所有LIS序列的方法（当LIS不止一个时），仅提供输出其中一个LIS的方法。这种方法中，在从头到尾遍历原序列`seq`的过程中，记录每个元素在LIS中应该的位置，之后再倒序查找原序列元素对应的LIS位置，即可得到一个LIS。


```
// Time complexity: n*log(n)
int LIS_n_log_n(vector<int> &seq, int NN)
{
    /*
     * tail[ii] is the minimum value of last element in sequence which length is ii.
     * cc[ii] is the related position of element seq[ii] in LNDS
     *
     *  if seq[ii] > tail[len] :                                     tail[++len] = seq[ii];
     *  else: find first value of tail which isn't smaller than seq[ii], replace with seq[ii]      tail[jj] = seq[ii]
     *  Could also think:   from the last value of tail which is equal to seq[ii], use it to update the next one.
     */
    vector<int> tail(NN + 1);
    vector<int> cc(NN + 1);
    int len = 1;
    tail[len] = seq[0];
    cc[0] = len;

    for (int ii = 1; ii < NN; ii++) {
        if (seq[ii] > tail[len]) {
            tail[++len] = seq[ii];
            cc[ii] = len;
        } else {
            int idx = lower_bound(tail.begin() + 1, tail.begin() + len + 1, seq[ii]) - tail.begin();
            tail[idx] = seq[ii];
            cc[ii] = idx;
        }
    }

    stack<int> output;
    int jj = len;
    for (int ii = NN - 1; ii >= 0; ii--) {
        if (cc[ii] == jj) {
            output.push(seq[ii]);
            jj--;
        }
        if (0 == jj)
            break;
    }
    while (!output.empty()) {
        int top = output.top();
        printf("%d ", top);
        output.pop();
    }

    return len;
}
```
&emsp;对于最长不下降子序列问题，稍微修改`tail`数组元素的更新原则：  
* `seq[ii] <= tail[len]`，查找`tail`数组中第一个大于`seq[ii]`的数，并用`seq[ii]`替换。  
&emsp;将上面程序中第`23`行的`lower_bound`函数替换为`upper_bound`即可。  

## 实际问题实例
&emsp;PAT练习题中`1045 Favorite Color Stripe`即可转化为一个求解LIS的问题。问题描述如下：


Eva is trying to make her own color stripe out of a given one. She would like to keep only her favorite colors in her favorite order by cutting off those unwanted pieces and sewing the remaining parts together to form her favorite color stripe.  
It is said that a normal human eye can distinguish about less than `200` different colors, so Eva's favorite colors are limited. However the original stripe could be very long, and Eva would like to have the remaining favorite stripe with the maximum length. So she needs your help to find her the best result.  
Note that the solution might not be unique, but you only have to tell her the maximum length. For example, given a stripe of colors `{2 2 4 1 5 5 6 3 1 1 5 6}`. If Eva's favorite colors are given in her favorite order as `{2 3 1 5 6}`, then she has `4` possible best solutions `{2 2 1 1 1 5 6}`, `{2 2 1 5 5 5 6}`, `{2 2 1 5 5 6 6}`, and `{2 2 3 1 1 5 6}`.  

* Input Specification:  
Each input file contains one test case. For each case, the first line contains a positive integer `N (≤200)` which is the total number of colors involved (and hence the colors are numbered from `1` to `N`). Then the next line starts with a positive integer `M (≤200)` followed by `M` Eva's favorite color numbers given in her favorite order. Finally the third line starts with a positive integer `L (≤ 10^4)` which is the length of the given stripe, followed by `L` colors on the stripe. All the numbers in a line a separated by a space.  

* Output Specification:  
For each test case, simply print in a line the maximum length of Eva's favorite stripe.  

* Sample Input:  
```
6
5 2 3 1 5 6
12 2 2 4 1 5 5 6 3 1 1 5 6
```
* Sample Output:  
```
7
```

**问题分析**：可以先将Eva最喜欢的颜色以此进行编号，例中`{2, 3, 1, 5, 6}`的编号即为`{0, 1, 2, 3, 4}`，这实际上转换为了一个次序。将颜色条中不是Eva喜欢的颜色舍去后，将颜色值用前述编号代替，例如例中的颜色条可以转换为`{0, 0, 2, 3, 3, 4, 1, 2, 2, 3, 4}`，这便可以使用求解LNDS的方式进行求解。求解程序如下：  
```
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int LNDS_n_n(vector<int> &given, int NN, vector<int> &colors)
{
    int longest = 1;
    vector<int> dp(NN, 1);

    for (int ii = 1; ii < NN; ii++) {
        for (int jj = 0; jj < ii; jj++) {
            if (given[jj] <= given[ii]) {
                dp[ii] = max(dp[ii], dp[jj] + 1);
            }
        }
        longest = max(longest, dp[ii]);
    }

    return longest;
}
int main(void)
{
    int NN = 0, MM = 0, LL = 0;
    scanf("%d", &NN);
    // Store whether a color is Eva's favorite or not, presence[ii] == 1 means color ii is valid.
    vector<int> presence(NN+1);
    // Store order of favorite color, order[ii] = 0 means color ii is most favorite one.
    vector<int> order(NN+1);
    vector<int> colors;
    scanf("%d", &MM);

    int color = 0;
    for (int ii = 0; ii < MM; ii++) {
        scanf("%d", &color);
        presence[color] = 1;
        order[color] = ii;
        colors.push_back(color);
    }
    scanf("%d", &LL);
    vector<int> given;
    for (int ii = 0; ii < LL; ii++) {
        scanf("%d", &color);
        if (1 == presence[color])
            given.push_back(order[color]);
    }
    printf("%d\n", LNDS_n_n(given, given.size(), colors));

    return 0;
}
```
