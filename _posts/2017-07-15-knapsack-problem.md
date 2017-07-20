---
layout: posts
title: "knapsack problem"
date: 2017-07-15
---


# 0/1 knapsack

## Problem

We have N items, each item has a weight and value, stores in two arrays `wt[], val[]`. Also we have a knapsack with maximum weight capacity `W`. We can either put or not put an item into knapsack. Given that the total weight of items in knapsack can not exceed `W`,  how can we maximize the item values in knapsack?

## Ideas

The most naive approach would be generating all different combinations of items and calculate ones that do not exceed the `W`. 

One observation is, for each item, we can either include or not include into our optimal solution. So the max value we can get from n items with weight `W` is the max of below two:
- Max value from `n-1` items with weight `W` (not include nth item)
- Value of nth item plus max value from `n-1` items with weight `W` substracted by the weight of nth item. (include nth item)

We can implement this recursive approach. After drawing a recurence tree, we may notice that when the remaining cap and items are the same accross some sub-problem, our function calculate some sub-problems more than once, meaning we have an overlapping problem here, which leads to dynamic programming approach.

We analyzed the optimal structure as above and define our state function:
` dp[i][w] = max(dp[i-1][w], dp[i-1][w - wt[i-1]] + val[i-1])`
 `dp[i][w]` means the max value we can get with the first `i` items and capacity `w` , assuming that we have an id for each item from 0 to n-1. 

## Space Optimization

Notice that, to obtain the max value with n items, we only need the result of with n-1 items, we do not need results before, like n-2, n-3 items. That leads to an optimization of space: we don't have to store the result for each item, only store for the previous item is enough for this problem. 

So instead of 2D array as `dp[i][w]`, we only need an 1D array that stores the max value with capacity w for each iteration of items. The array is `dp[0…W] `. Here is the tricky part: we iterate over n items, for `ith` eration , we take the `ith` item into consideration After previous iteration, the array stores the max value for weight 0 to W with i-1 items. To obtain the max value with `ith` item and weight `w` , we need the result of i-1 item in weight `w` and `w - wt[i-1]` .  NOTE that, we need to iterate all weight value in reverse order, since we want the result with i-1 items. We will see the difference again in `Complete knapsack problem`. The pseudo code is here:

```
for i = 0 to N-1:
	for w = W to 0:
		dp[w] = max(dp[w], dp[w - wt[i-1]] + val[i-1])
```

## Initialization

Some problems ask to exactly fill the knapsack at the capacity `W`, others allow partially fill. This will affect our initialization of the array. If asked exactly fill the knapsack, we will initialize our array with an invalid value like `-1` , and `dp[0] = 0`. If asked partially fill, we initialize the array with 0s. 

With the initialization of our array, it means that the state with 0 item, the max value we can obtain for each weight capacity. If asked exactly filled, so only the capacity 0 can be filled with 0 items. Other capacities can not be filled. 

If we do it by recursion, "exactly filled" will affect the termination condition: when all items are considered, if the remaining capacity is still greater than 0, meaning that we do not exactly fill the knapsack, we need to return 'invalid' therefore. 

## Implementation (python)

```python
def knapsack01(W, wt, val, n):
    """
    input type: int, list, list, int
    output type: int
    """
    dp = [float('-inf')] * (n + 1)  # -inf means invalid 
    dp[0] = 0
    for i in range(n):
        for w in range(W, -1):
            dp[w] = max(dp[w], dp[w - wt[i]] + val[i])
    return dp[W]
```

## Analysis

Time: O(N * W).  Two loops 

Space: O(W).  1D array optimization



# Complete Knapsack problem

## Problem

Unlike 0/1 knapsack problem, here we can use each item unlimited times. 

## Ideas

Quite similar as 0/1 knapsack problem, we also consider for each item. However, we have more options: take 0, 1, 2 … until the knapsack is full. So we define the state functions as below:

`dp[i][w] = max(dp[i-1][w - k * wt[i-1]] + k * val[i-1])`  
where `k * wt[i-1] in [0, w]`

Here we still need to iterate all items, but for each item, instead linear time, we need `∑(w/wt[i])` time, so the total time is worse than O(W * N)

## Convert to 0/1 situation

We can 'add' more items that for each item, duplicate it `W/wt[i]` times, and the whole problem becomes a 0/1 knapsack problem, but this one does not improve the time complexity. 

We can use binary reduction, to duplicate each item with 1, 2, 4, 8… times of it original weight. This approach will improve from the linear duplicate to logirithm duplicate. 

## Optimization to O(W * N)

In 0/1 problem, we iterate the weight value in reverse order, as we need the result of previous item iteration. Now with unlimited times, for each item at each capacity, we need the result of current item iteration. If we want to use an item i times, we need the result of using it i-1 times. SO, we iterate the weight value from 0 to W. Here is the revised state function:

`dp[i][w] = max(dp[i-1][w], dp[i][w - wt[i-1]] + val[i])`

## Implementation (python)

```python
def knapsack01(W, wt, val, n):
    """
    input type: int, list, list, int
    output type: int
    """
    dp = [float('-inf')] * (n + 1)  # -inf means invalid 
    dp[0] = 0
    for i in range(n):
        for w in range(W, -1):
            dp[w] = max(dp[w], dp[w - wt[i]] + val[i])
    return dp[W]
```

