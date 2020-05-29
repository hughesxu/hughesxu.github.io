---
title:  "1003 Emergency (25point(s)) (Dijsktra, Bellman-Ford, SPFA)"
date:   2020-05-29 14:41:22 +0800
categories: [PAT, Advanced]
tags: [dijsktra, bellman-ford, spfa]
---

## 问题描述

 
As an emergency rescue team leader of a city, you are given a special map of your country. The map shows several scattered cities connected by some roads.
Amount of rescue teams in each city and the length of each road between any pair of cities are marked on the map.  
When there is an emergency call to you from some other city, your job is to lead your men to the place as quickly as possible, and at the mean time, call up as many hands on the way as possible.

* Input Specification:  
Each input file contains one test case. For each test case, the first line contains 4 positive integers:  
&emsp;`N (≤500)` - the number of cities (and the cities are numbered from 0 to `N−1`), `M` - the number of roads,  
&emsp;`C1` and `C2` - the cities that you are currently in and that you must save, respectively.  
The next line contains `N` integers, where the `i-th` integer is the number of rescue teams in the `i-th` city. Then `M` lines follow, each describes a road with three integers `c1`, `c2` and `L`,
which are the pair of cities connected by a road and the length of that road, respectively. It is guaranteed that there exists at least one path from `C1` to `C2`.


* Output Specification:  
For each test case, print in one line two numbers: the number of different shortest paths between `C1` and `C2`, and the maximum amount of rescue teams you can possibly gather.
All the numbers in a line must be separated by exactly one space, and there is no extra space allowed at the end of a line.

* Sample Input:  
```
5 6 0 2
1 2 1 5 3
0 1 1
0 2 2
0 3 1
1 2 1
2 4 1
3 4 1
```
* Sample Output:  
```
2 4  
```

## 问题分析


&emsp;这是一个无向赋权图单源最短路径问题。`G(V, E)`, `V`代表所有城市集合，`E`代表相邻城市间道路集合。值得注意的是输出第一项并不是从起始城市到目标城市的最短路径，而是具有最短路径权重的路径数目；第二项是在所有最短路径中能够号召聚集紧急解救团队的最大数目。  
&emsp;分析显然，城市间的距离不可能为一个负值，所以是个正权值图，即没有任何一条边`e`具有负值权重。很容易想到使用`Dijsktra`求解，为了熟悉单源最短路径问题的求解，也尝试了`Bellman-Ford`和`SPFA`方法，尽管这两种算法对无负权值图问题求解效率不高。  
&emsp;使用上述几种算法求解的实现中，均定义了`FATAL(x)`函数处理`FATAL ERROR`, 并定义了`City_Info`结构体来描述每个城市节点信息，具体定义如下：  
```
// Print out FATAL ERROR info, and exit the program  
#define FATAL(x)            \  
    do {                    \  
        printf("%s\n", x);  \  
        exit(1);            \  
    }while(0);
```

```
// City_Info to store related info  
typedef struct {  
    int known;      // current city node known (processed) or not.  
    int dist;       // Distance from source city node to current.  
    int path_num;   // Number of shortest path (from s to current).  
    int resc_num;   // Number of rescue teams in current city.  
    int resc_max;   // Max amount of rescue teams that can be gathered (under shortest paths from s to current).  
}City_Info;  
```


### (1) Dijsktra

```
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

int main(void)
{
    int rc = 0;
    int NN = 0, MM = 0, C1 = 0, C2 = 0;
    int ii = 0, jj = 0;
    City_Info *city = NULL;
    int c1 = 0, c2 = 0, LL = 0;
    int **cdata = NULL;

    int min_dist = 0, min = -1;

    while (EOF != (rc = scanf("%d%d%d%d", &NN, &MM, &C1, &C2))) {
        // Dynamically calloc memory for most data structure.
        city = (City_Info *)calloc(NN, sizeof(City_Info));
        if (NULL == city) {
            FATAL("ERROR: Unable to calloc for data*");
        }

        // cdata to store all the road info, like adjacent table.
        cdata = (int **)calloc(NN, sizeof(int *));
        for (ii = 0; ii < NN; ii++) {
            cdata[ii] = (int *)calloc(NN, sizeof(int));
        }

        // Initialize all the city distance to INT_MAX
        for (ii = 0; ii < NN; ii++) {
            rc = scanf("%d", &(city[ii].resc_num));
            city[ii].dist = INT_MAX;
        }
        city[C1].dist = 0;                        // distance between C1 and C1 is ZERO.
        city[C1].resc_max = city[C1].resc_num;    // Max rescue teams C1 could gather equal to rescue teams it have.
        city[C1].path_num = 1;                    // Number of shortest path to 1 (from C1 to C1).

        // Initialize distances between 2 cities with INT_MAX, then overwrite with correct value below.  
        // Cities have distance of INT_MAX between them mean: no connected road.
        for (ii = 0; ii < NN; ii++) {
            for (jj = 0; jj < NN; jj++) {
                cdata[ii][jj] = INT_MAX;
            }
        }
        for (ii = 0; ii < MM; ii++) {
            rc = scanf("%d%d%d", &c1, &c2, &LL);
            cdata[c1][c2] = LL;
            cdata[c2][c1] = LL;
        }

        // Dijsktra Algorithm
        if (C1 != C2) {
            for ( ; ; ) {
                // Find out unknown city that has shortest path distance 
                for (ii = 0, min_dist = INT_MAX, min = -1; ii < NN; ii++) {
                    if (0 == city[ii].known && city[ii].dist < min_dist) {
                        min = ii;
                        min_dist = city[ii].dist;
                    }
                }
                if (-1 == min)      // min == -1 means: No such unknown city found
                    break;

                // Set unknown city to known state
                city[min].known = 1;

                for (ii = 0; ii < NN; ii++) {
                    if (INT_MAX != cdata[min][ii] && 0 == city[ii].known) {              // Find out adjacent and unknown city: ii.
                        if (city[min].dist + cdata[min][ii] < city[ii].dist) {
                            city[ii].dist = city[min].dist + cdata[min][ii];             // Update with latest path distance.

                            city[ii].resc_max = city[min].resc_max + city[ii].resc_num;  // If find shorter path, update rescue number directly!

                            city[ii].path_num = city[min].path_num;

                        } else if (city[min].dist + cdata[min][ii] == city[ii].dist) {
                            city[ii].path_num += city[min].path_num;
                            if (city[ii].resc_max < city[min].resc_max + city[ii].resc_num)
                                city[ii].resc_max = city[min].resc_max + city[ii].resc_num;
                        }
                    }
                }
            }
        }

        printf("%d %d\n", city[C2].path_num, city[C2].resc_max);

        // Free of memory
        if (city)
            free(city);
        for (ii = 0; ii < NN; ii++) {
            if (cdata[ii])
                free(cdata[ii]);
        }
        free(cdata);
    }
    return 0;
}
```


### (2) SPFA
 
&emsp;在尝试`SPFA`的过程中卡住了一段时间，最开始采用`Dijsktra`方法中同样的原则更新`city[ii].path_num`时，case 2/4不能通过，只能获得19分。  
&emsp;查询网上别人分享的代码，仔细思考后找到关键点：`SPFA`对每个边并不能保证只做一次松弛操作，所以如果一个已经处理过的节点（进入过`Queue`并离开`Queue`）再次进入`Queue`中被处理时，`path_num`的更新会重复包含一部分之前已经计算在内的路径数， 导致结果出错。  
&emsp;采用的改进方法为：**每次更新`path_num`均用其前驱节点的`path_num`求和计算得到，这样避免了重复路径数的产生。**  
```
#include <iostream>
#include <queue>
#include <vector>
#include <set>
#include <climits>
int main(void)
{
    int NN = 0, MM = 0, C1 = 0, C2 = 0;
    int ii = 0, jj = 0;
    int c1 = 0, c2 = 0, LL = 0;
    int index = 0;

    while ( cin >> NN >> MM >> C1 >> C2 ) {

        queue<int> ComingQ;
        vector<int>Inqueue(NN);
        vector<City_Info> city(NN);

        for (ii = 0; ii < NN; ii++) {
            cin >> (city[ii].resc_num);
            city[ii].dist = INT_MAX;
        }
        city[C1].dist = 0;
        city[C1].resc_max = city[C1].resc_num;
        city[C1].path_num = 1;

        // Initialize all the data, similar principle with that used in Dijsktra.
        // cdata to store all the road info, like adjacent table.
        vector<vector<int> > cdata(NN);
        for (ii = 0; ii < NN; ii++)
            cdata[ii].assign(NN, INT_MAX);

        for (ii = 0; ii < MM; ii++) {
            cin >> c1 >> c2 >> LL;
            cdata[c1][c2] = LL;
            cdata[c2][c1] = LL;
        }

        // Start of process: put source city node into Processing Queue.
        ComingQ.push(C1);
        Inqueue[C1] = 1;

        // Save pre-node city
        vector<set<int> > pre(NN);

        while ( !ComingQ.empty() ) {
            index = ComingQ.front();
            Inqueue[index] = 0;
            ComingQ.pop();

            for (ii = 0; ii < NN; ii++) {
                // Find out adjacent cities
                if (INT_MAX != cdata[index][ii]) {
                    if (city[ii].dist > city[index].dist + cdata[index][ii]) {
                        city[ii].dist = city[index].dist + cdata[index][ii];

                        city[ii].resc_max = city[index].resc_max + city[ii].resc_num;

                        city[ii].path_num = city[index].path_num;
                        pre[ii].clear();
                        pre[ii].insert(index);

                        if (0 == Inqueue[ii]) {
                            ComingQ.push(ii);
                            Inqueue[ii] = 1;
                        }
                    } else if (city[ii].dist == city[index].dist + cdata[index][ii]) {
                        /*
                         * If one node adjacent to target node be pushed into queue for second time,
                         * then partial path_num will be counted for the second time.
                         */
                        // city[ii].path_num += city[index].path_num;

                        pre[ii].insert(index);
                        city[ii].path_num = 0;
                        for(set<int>::iterator it = pre[ii].begin(); it != pre[ii].end(); it++)
                            city[ii].path_num += city[*it].path_num;

                        if (city[ii].resc_max < city[index].resc_max + city[ii].resc_num)
                            city[ii].resc_max = city[index].resc_max + city[ii].resc_num;

                        // resc_max update also lead to partial update of other part of graph
                        if (0 == Inqueue[ii]) {
                            ComingQ.push(ii);
                            Inqueue[ii] = 1;
                        }
                    }
                }
            }
        }
    }
    cout << city[C2].path_num << " " << city[C2].resc_max << endl;
}
return 0;
}

```


### (3) Bellman-Ford

&emsp;采用`Bellman-Ford`算法求解时同样使用和`SPFA`相同的策略更新`path_num`。  

```
#include <iostream>
#include <queue>
#include <vector>
#include <set>
#include <climits>

using namespace std;

int main(void)
{
    int NN = 0, MM = 0, C1 = 0, C2 = 0;
    int ii = 0, jj = 0;
    int c1 = 0, c2 = 0, LL = 0;
    int index = 0;

    while ( cin >> NN >> MM >> C1 >> C2 ) {
        vector<City_Info> city(NN);

        for (ii = 0; ii < NN; ii++) {
            cin >> (city[ii].resc_num);
            city[ii].dist = INT_MAX;
        }
        city[C1].dist = 0;
        city[C1].resc_max = city[C1].resc_num;
        city[C1].path_num = 1;

        // Initialize
        // cdata to store all the road info, like adjacent table.
        vector<vector<int> > cdata(NN);
        for (ii = 0; ii < NN; ii++)
            cdata[ii].assign(NN, INT_MAX);

        for (ii = 0; ii < MM; ii++) {
            cin >> c1 >> c2 >> LL;
            cdata[c1][c2] = LL;
            cdata[c2][c1] = LL;
        }

        // Start of process
        vector<set<int> > pre(NN);

        for (jj = 1; jj < NN; jj++) {         // Max relaxation times of each edge is (NN-1).

            // Traverse every edge of city graph
            for (index = 0; index < NN; index++) {
                for (ii = 0; ii < NN; ii++) {
                    // Find out adjacent cities
                    if (INT_MAX != cdata[index][ii]) {
                        /* Skip those unknown/invisible node (have not updated distance), 
                         * because these nodes could not relax any adjacent nodes.
                         */
                        if (INT_MAX == city[index].dist) {

                        } else if (city[ii].dist > city[index].dist + cdata[index][ii]) {
                            city[ii].dist = city[index].dist + cdata[index][ii];

                            city[ii].resc_max = city[index].resc_max + city[ii].resc_num;

                            city[ii].path_num = city[index].path_num;
                            pre[ii].clear();
                            pre[ii].insert(index);

                        } else if (city[ii].dist == city[index].dist + cdata[index][ii]) {
                            /*
                             * If one node adjacent to target node be pushed into queue for second time,
                             * then partial path_num will be counted for second time.
                             */
                            // city[ii].path_num += city[index].path_num;

                            pre[ii].insert(index);
                            city[ii].path_num = 0;
                            for(set<int>::iterator it = pre[ii].begin(); it != pre[ii].end(); it++)
                                city[ii].path_num += city[*it].path_num;

                            if (city[ii].resc_max < city[index].resc_max + city[ii].resc_num)
                                city[ii].resc_max = city[index].resc_max + city[ii].resc_num;

                        }
                    }
                }
            }
        }
        cout << city[C2].path_num << " " << city[C2].resc_max << endl;
    }
    return 0;
}
```
