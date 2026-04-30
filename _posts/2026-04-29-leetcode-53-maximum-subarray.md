---
title: "53 · Maximum Subarray"
date: 2026-04-29 11:00:00 +0000
categories: [Algorithms, LeetCode]
tags: [greedy, array, java, interview, leetcode, kadane]
description: A complete breakdown of LeetCode 53 — intuition, brute force vs Kadane's Algorithm.
---

## What Is This Problem Really About?

At first glance you are summing subarrays. What is actually being tested is
whether you can look at a sequence and make a greedy local decision at every
step — extend a running sum or cut it and start fresh — without ever needing
to look backward.

This is **Kadane's Algorithm**, and it is one of the most famous greedy
techniques in computer science. It belongs to the same family as LC 121:
instead of re-scanning past elements, you carry exactly one number forward
and update it in constant time.

---

## Problem Classification

| Property | Detail |
|---|---|
| **Pattern** | Greedy + Array (Kadane's Algorithm) |
| **Core goal** | Find the contiguous subarray with the largest sum |
| **Input** | Integer array with possible negatives, `n ≤ 10^5` |
| **Expected complexity** | O(n) time · O(1) space |

> **Key constraint:** The subarray must be **contiguous** — you cannot skip
> elements. You must also pick at least one element, which matters when
> the entire array is negative.

---

## The Brute Force (O(n²))

The natural starting point is to enumerate every possible contiguous subarray
and compute its sum. For an array `[a, b, c]`, those are:

```
Starting at index 0: [a]   [a,b]   [a,b,c]
Starting at index 1:        [b]    [b,c]
Starting at index 2:                [c]
```

For an array of length `n`, the total number of subarrays is \(\frac{n(n+1)}{2}\).
At `n = 10^5` that is roughly 5 billion — far too slow.

The trick to keep it at O(n²) rather than O(n³) is to **accumulate as you
extend**, so each step costs O(1):

```java
for (int i = 0; i < n; i++) {
    int currentSum = 0;
    for (int j = i; j < n; j++) {
        currentSum += nums[j];                    // extend by one element
        maxSum = Math.max(maxSum, currentSum);
    }
}
```

Now ask: for any end index `j`, do you really need to try every possible start
index `i`? Or can you figure out the best start in O(1)?

---

## The Greedy Insight

At every index `i`, you face exactly one binary decision:

> *"Is my current running sum helping or hurting future elements?"*

- If `currentSum > 0` → **extend**: carrying it forward adds value to whatever comes next
- If `currentSum ≤ 0` → **start fresh** from `nums[i]`: a negative prefix can only drag future sums down

This is the **greedy choice property**: cutting a negative running sum immediately
is always at least as good as keeping it. You never need to reconsider.

### Step-by-step trace

```
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

i=0: currentSum=-2            maxSum=-2
i=1: -2≤0 → fresh → 1        maxSum=1
i=2: 1>0  → extend → 1-3=-2  maxSum=1
i=3: -2≤0 → fresh → 4        maxSum=4
i=4: 4>0  → extend → 4-1=3   maxSum=4
i=5: 3>0  → extend → 3+2=5   maxSum=5
i=6: 5>0  → extend → 5+1=6   maxSum=6  ✅
i=7: 6>0  → extend → 6-5=1   maxSum=6
i=8: 1>0  → extend → 1+4=5   maxSum=6

Answer: 6  →  subarray [4, -1, 2, 1]
```

---

## Java Solution (O(n) · O(1))

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int currentSum = nums[0];
        int maxSum = nums[0];

        for (int i = 1; i < nums.length; i++) {
            if (currentSum > 0) {
                currentSum += nums[i];                  // extend
            } else {
                currentSum = nums[i];                   // start fresh
            }
            maxSum = Math.max(maxSum, currentSum);      // track global best
        }

        return maxSum;
    }
}
```

> ⚠️ **Initialize to `nums[0]`, not `0`.** If the array is all negative
> (e.g. `[-3, -1, -2]`), the answer is `-1` — you are forced to pick at
> least one element. Initializing to `0` would incorrectly return `0`.

---

## The Two State Variables

Just like LC 121, two variables carry all the information you need:

| Variable | Meaning |
|---|---|
| `currentSum` | Best subarray sum **ending at the current index** |
| `maxSum` | Best subarray sum seen **anywhere so far** |

The key insight is that `currentSum` resets when it goes negative, but
`maxSum` never goes backward — it keeps the best historical value across
all resets.

---

## Common Bugs to Avoid

These two mistakes appear frequently:

**Bug 1 — Wrong reset condition.** Comparing `currentSum + nums[i]` against
the wrong value (e.g. `nums[i-1]`) instead of against `nums[i]` itself.
The correct question is always: *"is currentSum positive?"*

```java
// ❌ Wrong
if (currentSum + nums[i] > nums[i-1])

// ✅ Correct
if (currentSum > 0)
```

**Bug 2 — Overwriting instead of competing.** Assigning `maxSum = currentSum`
at the end of every iteration throws away the historical best. Use `Math.max`:

```java
// ❌ Wrong — loses the best seen in the middle of the array
maxSum = currentSum;

// ✅ Correct
maxSum = Math.max(maxSum, currentSum);
```

---

## How LC 53 Relates to LC 121

These two problems share the same skeleton. The only meaningful difference
is that in LC 121 the running variable (`minPrice`) only ever decreases and
never resets, while in LC 53 the running variable (`currentSum`) can reset
mid-array when it turns negative. That reset condition is the one new idea
Kadane's adds on top of the LC 121 pattern.

| | LC 121 – Stock | LC 53 – Max Subarray |
|---|---|---|
| Running variable | `minPrice` (only decreases) | `currentSum` (can reset) |
| Reset condition | Never | When `currentSum ≤ 0` |
| Second variable | `maxProfit` | `maxSum` |
| Local decision | Update min or update profit | Extend or start fresh |

---

## Interview Mental Framework

Apply the same four steps as any running state problem:

1. **Read constraints** — `n ≤ 10^5` → O(n) required
2. **State brute force** — enumerate all subarrays in O(n²)
3. **Find redundancy** — do I need to re-scan all starts for every end? No
4. **Carry state forward** — `currentSum` replaces the inner loop entirely

---

## Related Problems (Same Family)

| Problem | Key Concept | Difficulty |
|---|---|---|
| LC 121 – Stock Buy & Sell | Running minimum, no reset | Easy |
| LC 152 – Maximum Product Subarray | Track both max and min (negatives flip sign) | Medium |
| LC 918 – Max Sum Circular Subarray | Kadane's + circular wrap-around case | Medium |
| LC 123 – Stock Buy & Sell III | Two-pass state tracking | Hard |

Mastering Kadane's is not just about LC 53 — it is about building the
instinct that a running variable with a reset condition can replace a full
nested scan wherever a negative prefix can only make things worse.
