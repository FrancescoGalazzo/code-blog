---
title: "121 · Best Time to Buy and Sell Stock"
date: 2026-04-29 09:00:00 +0000
categories: [Algorithms, LeetCode]
tags: [greedy, array, java, interview, leetcode, easy]
description: A deep dive into the Greedy + Array pattern — intuition, brute force vs optimal, Java solution, and interview strategies to spot and solve this class of problem fast.
---

## What Is This Problem Really About?

At first glance you're dealing with stock prices, but that's just the costume.
What's actually being tested is whether you can look at an O(n²) brute force
and ask: *"am I doing unnecessary work here?"* — and then eliminate it.

The array is unsorted, prices go up and down, and you need the best
buy/sell pair. The naive move is to check every combination. The smart move
is realizing you never need to look backward more than once: one well-chosen
variable, updated as you walk forward, carries everything you need.

That shift — from re-scanning the past to carrying it forward — is the heart
of the **Greedy + Array** pattern, and it shows up constantly in interviews.

***

## Problem Classification

| Property | Detail |
|---|---|
| **Pattern** | Greedy + Array |
| **Core goal** | Maximize a value across a sequence in a single pass |
| **Input** | Unsorted integer array, `n ≤ 10^5` |
| **Expected complexity** | O(n) time · O(1) space |

> **Constraint rule of thumb:** `n ≤ 10^5` means O(n²) will time-out.
> Always target O(n) or O(n log n) for inputs of this size.

***

## The Brute Force Intuition (O(n²))

When facing an **unordered array**, the first natural instinct is to try every
combination — for each possible buy day, scan all later days to find the best
sell day. This gives a correct answer but runs in **O(n²)**:

```java
// Correct but too slow for n = 10^5
for (int i = 0; i < prices.length; i++) {           // buy day
    for (int j = i + 1; j < prices.length; j++) {   // sell day
        maxProfit = Math.max(maxProfit, prices[j] - prices[i]);
    }
}
```

This is a valid starting point to state out loud in an interview — it shows
structured thinking. The next step is to ask: **where is the redundancy?**

***

## The Greedy Insight: Cut the Inner Loop

The key question to ask yourself:

> *"For any sell day `j`, do I really need to look back at all previous days
> to find the best buy day?"*

The answer is **no**. The best buy day is always the **minimum price seen so
far** as you scan left to right. You never need to reconsider — once you have
seen a lower price, it is always at least as good as any earlier price for a
buy day.

This is the **greedy choice property**: making the locally optimal decision at
each step (always track the cheapest price seen so far) leads to the globally
optimal solution[1].

### Step-by-step trace

```
prices = [7, 1, 5, 3, 6, 4]

Day 0: price=7 → minPrice=7, maxProfit=0
Day 1: price=1 → minPrice=1, maxProfit=0   ← new minimum found
Day 2: price=5 → minPrice=1, maxProfit=4   ← profit 5-1=4
Day 3: price=3 → minPrice=1, maxProfit=4   ← profit 3-1=2, no update
Day 4: price=6 → minPrice=1, maxProfit=5   ← profit 6-1=5 ✅
Day 5: price=4 → minPrice=1, maxProfit=5   ← profit 4-1=3, no update

Answer: 5
```

***

## Optimal Java Solution (O(n) · O(1))

```java
class Solution {
    public int maxProfit(int[] prices) {
        int minPrice = Integer.MAX_VALUE;
        int maxProfit = 0;

        for (int price : prices) {
            if (price < minPrice) {
                minPrice = price;                               // better buy day
            } else {
                maxProfit = Math.max(maxProfit, price - minPrice); // better profit?
            }
        }

        return maxProfit;
    }
}
```

Two variables. One loop. The inner loop is gone entirely — replaced by a single
running variable that carries the best past value forward[2].

***

## Interview Mental Framework

This four-step process applies to virtually any array optimization problem:

1. **Read constraints first** — `n ≤ 10^5` means immediately discard O(n²)
2. **State the brute force out loud** — shows structured thinking before optimizing
3. **Find the redundancy** — ask *"do I really need to re-scan past elements?"*
4. **Carry state forward** — replace the inner loop with a running variable

### How to spot a Greedy problem

Look for these signals in the problem statement:

- The problem asks to **optimize a single value** (max or min)
- At each step, there is **one obvious local choice** — no need to explore alternatives
- The best past value is **trivially updatable in O(1)** (running min, max, sum)
- Verify: **would reconsidering a past choice ever help?** If no → greedy. If yes → likely DP

***

## The "Running State" Trick

This is the reusable core of the entire pattern. Any time you see an O(n²)
solution with a nested loop where the inner loop just finds the best past
element, that inner loop can be replaced by a single variable updated
incrementally:

| Running variable | What the inner loop was doing | Problem |
|---|---|---|
| `minPrice` so far | Finding the cheapest buy day | LC 121 – Stock Buy & Sell |
| `maxRunningSum` so far | Finding the best subarray ending here | LC 53 – Maximum Subarray |
| `maxReach` so far | Finding the farthest reachable index | LC 55 – Jump Game |
| `runningGas` so far | Finding the net gas balance so far | LC 134 – Gas Station |

***

## Related Problems (Same DNA)

Practice these in order to internalize the pattern:

| Problem | Key Concept | Difficulty |
|---|---|---|
| LC 53 – Maximum Subarray | Kadane's algorithm — running sum | Easy |
| LC 122 – Stock Buy & Sell II | Multiple transactions — running profit | Medium |
| LC 55 – Jump Game | Greedy on max reachable index | Medium |
| LC 134 – Gas Station | Greedy on running gas balance | Medium |
| LC 123 – Stock Buy & Sell III | At most 2 transactions — two-pass approach | Hard |
