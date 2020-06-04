---
title:  "1013 Battle Over Cities (25point(s)) (dfs, 无向图连通分量)"
date:   2020-06-04 12:00:00 +0800
categories: [PAT, Advanced]
tags: [dfs]
---

## 问题描述

 
It is vitally important to have all the cities connected by highways in a war. If a city is occupied by the enemy, all the highways from/toward that city are closed. We must know immediately if we need to repair any other highways to keep the rest of the cities connected. Given the map of cities which have all the remaining highways marked, you are supposed to tell the number of highways need to be repaired, quickly.  

For example, if we have 3 cities and 2 highways connecting `city1 - city2` and `city1 - city3`. Then if city1 is occupied by the enemy, we must have 1 highway repaired, that is the highway `city2 - city3`.


* Input Specification:  
Each input file contains one test case. Each case starts with a line containing 3 numbers `N (<1000)`, `M` and `K`, which are the total number of cities, the number of remaining highways, and the number of cities to be checked, respectively. Then `M` lines follow, each describes a highway by 2 integers, which are the numbers of the cities the highway connects. The cities are numbered from 1 to `N`. Finally there is a line containing `K` numbers, which represent the cities we concern.


* Output Specification:  
For each of the `K` cities, output in a line the number of highways need to be repaired if that city is lost.

* Sample Input:  
```
3 2 3
1 2
1 3
1 2 3
```
* Sample Output:  
```
1
0
0
```

## 问题分析


&emsp;所有城市及其间的道路构成无向图`G(V, E)`，本题要求解的是某个城市顶点被移除时，至少要维修几条公路才能使得城市间相互连通。实则是求解无向图中的连通分支数量，若移除某个城市后，当前图中有`n`个连通分支数，那么只需`n-1`条边即可使无向图重新连通。  
&emsp;对于无向图连通分支数量的求解，采用深度优先搜索遍历全图。如果从任意顶点开始的深度优先搜索不能一次遍历到图的所有顶点，那么该无向图肯定存在至少一个连通分量。  

## 知识点补充：连通分量 
&emsp;关于图的连通性，先从简单的无向图来说，有如下几种定义：  
* `连通图 (Connected Graph)` ：无向图`G`中从任意一个顶点到任意其他顶点都存在一条路径。  
* `非连通图 (Disconnected Graph)` ：无向图不是连通图。  
* `连通分量 (Connected Component, 连通分支)` ：对非连通图`G`，其极大连通子图。

 
&emsp;有向图由于边具有方向性，根据连通性的强弱可分为强连通、单连通和弱连通。  
* `强连通 (Strongly Connected)` : 若`G`是有向图，如果对图`G`中任意两个顶点`u`和`v`，既存在从`u`到`v`的路径，也存在从`v`到`u`的路径，则称该有向图为强连通有向图。对于非强连通图，其极大强连通子图称为其强连通分量。  
* `单连通 (Simply Connected)` ：若`G`是有向图，如果对图`G`中任意两个顶点`u`和`v`，存在从`u`到`v`的路径或从`v`到`u`的路径，则称该有向图为单连通有向图。  
* `弱连通 (Weak Connected)` ：若`G`是有向图，如果忽略图`G`中每条有向边的方向，得到的无向图（即有向图的基图）连通，则称该有向图为弱连通有向图。  

&emsp;当无向图为非连通图时，从图中某一顶点出发，利用深度优先搜索或广度优先搜索算法不可能遍历到图中的所有顶点，只能访问到该顶点所在的极大连通子图（即连通分量）中的所有顶点。若从无向图的每一个连通分量中的一个顶点出发进行遍历，就可以访问到所有顶点。  

## 程序求解

```
#include <iostream>
#include <vector>

using namespace std;

// Note: visited must use reference value as parameter, as value will be modified during dfs.
void dfs(const vector<vector<int> > &city_graph, int city_num, vector<int> &visited)
{
    visited[city_num] = 1;
    for (unsigned int ii = 0; ii < city_graph[city_num].size(); ii++) {
        int city_adj = city_graph[city_num][ii];
        if (0 == visited[city_adj]) {
            dfs(city_graph, city_adj, visited);
        }
    }
}

int main(void)
{
    int NN = 0, MM = 0, KK = 0;

    while (cin >> NN >> MM >> KK) {
        // Cities are numbered from 1 to N.
        // Use vector works as adjacent table to record connected city info.
        vector<vector<int> > city(NN+1);

        int ct1 = 0, ct2 = 0;
        int rc = 0;
        for (int ii = 0; ii < MM; ii++) {
            // scanf()/printf() are more efficient than cin/cout.
            rc = scanf("%d %d", &ct1, &ct2);
            city[ct1].push_back(ct2);
            city[ct2].push_back(ct1);
        }

        int target = 0;
        for (int ii = 0; ii < KK; ii++) {
            rc = scanf("%d", &target);
            int branch = 0;
            vector<int> visited(NN+1, 0);

            // Mark concerned city as visited, then will not be touched during dfs.
            visited[target] = 1;
            // Start from city 1 to NN, traverse each city and do dfs.
            // If still have cities un-visited, means has more than one Connected Component.
            for (int jj = 1; jj < NN + 1; jj++) {
                if (0 == visited[jj]) {
                    dfs(city, jj, visited);
                    branch++;
                }
            }
            printf("%d\n", branch - 1);

        }
    }
    return 0;
}
```
