---
title: "209 - Minimum Size Subarray Sum (Why Sliding Window Is O(n))"
date: 2026-05-03 17:41:00 +0000
categories: [Algorithms, LeetCode]
tags: [sliding-window, array, java, interview, leetcode]
description: "A clear walkthrough of LeetCode 209 with the sliding window pattern, and a detailed explanation of why the algorithm is truly O(n) time, not O(n^2)."
---

## What Is This Problem Really About?

The formal task: given an array `nums` of **positive integers** and an integer `target`, find the **minimal length** of a contiguous subarray whose sum is **at least `target`**. If no such subarray exists, return 0.

The deeper idea is: this is a textbook **variable-size sliding window** problem. The positivity of the numbers lets you expand and shrink a window in linear time and still be sure you find the minimum length.

***

## Problem Classification

| Property | Detail |
|---|---|
| **Pattern** | Variable-size Sliding Window |
| **Core goal** | Shortest subarray with sum ≥ target |
| **Key property** | All `nums[i] > 0` (strictly positive) |
| **Expected complexity** | O(n) time · O(1) space |

That positivity condition is **why** the sliding window works and why the time complexity is O(n).

***

## Naive Intuition (Why Not O(n^2)?)

The first natural idea for “shortest subarray with sum ≥ target” is:

- For every start index `i`, extend an end index `j ≥ i` until the sum ≥ target, track the length, then move to the next `i`.

This means a nested loop:

```java
for (int i = 0; i < n; i++) {
    int sum = 0;
    for (int j = i; j < n; j++) {
        sum += nums[j];
        if (sum >= target) {
            minLen = Math.min(minLen, j - i + 1);
            break;
        }
    }
}
```

Time complexity: in the worst case, the inner loop runs O(n) for each `i`, giving **O(n^2)**. For `n = 10^5`, that’s too slow.

The sliding window approach keeps the idea of expanding and shrinking but does it in **one pass**.

***

## Sliding Window Solution (Java)

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int left = 0;
        int sum = 0;
        int minLen = Integer.MAX_VALUE;

        for (int right = 0; right < nums.length; right++) {
            sum += nums[right];

            // Shrink from the left while the window is still valid
            while (sum >= target) {
                minLen = Math.min(minLen, right - left + 1);
                sum -= nums[left++];
            }
        }

        return minLen == Integer.MAX_VALUE ? 0 : minLen;
    }
}
```

Now the key question: this has a `for` loop and a `while` loop. Why **isn’t** this O(n^2)?  

The answer lies in how often `left` and `right` move.

***

## Why the Time Complexity Is O(n)

At first glance, the nested `for` + `while` makes it look like O(n^2). But the critical observation is:

> **Each index is visited at most twice: once when `right` moves forward, once when `left` moves forward.**

More precisely:

- `right` starts at 0 and **only moves forward**, up to `n - 1`.
- `left` also starts at 0 and **only moves forward**, up to `n - 1`.
- Neither pointer ever moves backward.

### Pointer Movement Accounting

Let’s count the total number of operations over the whole run:

1. The `for (right = 0; right < n; right++)` loop:
   - `right` increases from 0 to `n - 1`.
   - This contributes **n** iterations.

2. Inside the `while (sum >= target)` loop:
   - Every time the body executes, you do `left++`.
   - `left` starts at 0 and can **never exceed `n`**.
   - Therefore, across the entire algorithm, `left` is incremented at most **n** times.

So even though the `while` loop is nested inside the `for`, the total number of iterations of that `while` loop across the whole run is **at most n**.

Combine both:

- `right` loop: at most n iterations.
- `left` increments: at most n iterations.

Total “pointer moves” ≲ 2n → **O(n)**.

There is no situation where for each `right` you scan the whole array with `left`. Once `left` has moved past an index, it never goes back.

### Why positivity matters

This works because all `nums[i]` are **positive**:

- When you move `right` forward (`sum += nums[right]`), the sum **never decreases**.
- Once `sum >= target`, the only way to reduce the sum is to move `left` forward and subtract positive numbers.
- That means:
  - As `right` moves, `sum` monotonically increases (until you start shrinking).
  - As `left` moves, `sum` monotonically decreases (until you start expanding again).

This monotonic behavior guarantees you never need to move `left` backwards to “try again” for the same `right`. Each index enters the window once and leaves once.

If the array could contain negative numbers, shrinking from the left could *increase* the sum, and this neat O(n) argument would break. Then sliding window becomes much trickier or impossible for the same problem.

***

## Example Walkthrough

To make the sliding window idea concrete, use this input:

```text
target = 7
nums   =[1][2][3][4]
```

The goal is not to check every possible subarray from scratch. The goal is to maintain one **contiguous window** and adjust it intelligently.

We keep:

- `left`  = start of the current window
- `right` = end of the current window
- `sum`   = sum of the current window
- `minLen` = best answer found so far

### Step-by-step trace

```text
Start:
left = 0, sum = 0, minLen = ∞
```

Expand the window by moving `right`:

```text
right = 0  → add 2
window =[1]
sum = 2
```

Still too small, so expand again:

```text
right = 1  → add 3
window =[2][1]
sum = 5
```

Still too small:

```text
right = 2  → add 1
window =[3][2][1]
sum = 6
```

Still too small:

```text
right = 3  → add 2
window =[2][3][1]
sum = 8
```

Now the window is valid because `8 ≥ 7`.  
So instead of expanding more, try to **shrink from the left** to see if a shorter valid window exists.

```text
length = 4  → minLen = 4
remove 2 from the left
left = 1
window =[3][1][2]
sum = 6
```

The window is no longer valid, so go back to expanding:

```text
right = 4  → add 4
window =[4][1][2][3]
sum = 10
```

Valid again, so shrink:

```text
length = 4  → minLen = 4
remove 3 from the left
left = 2
window =[4][1][3]
sum = 7
```

Still valid, so shrink again:

```text
length = 3  → minLen = 3
remove 1 from the left
left = 3
window =[1][4]
sum = 6
```

No longer valid, so expand again:

```text
right = 5  → add 3
window =[2][4][1]
sum = 9
```

Valid, so shrink:

```text
length = 3  → minLen = 3
remove 2 from the left
left = 4
window =[4][2]
sum = 7
```

Still valid, so shrink one more time:

```text
length = 2  → minLen = 2
remove 4 from the left
left = 5
window =[2]
sum = 3
```

Now the window is no longer valid, and we have reached the end of the array.

### Final result

```text
Answer = 2
Shortest valid subarray =[4][2]
```

### What this example shows

This example highlights the exact rhythm of the variable-size sliding window:

- **Expand** while the window is invalid.
- **Shrink** while the window remains valid.
- Update the answer **during shrinking**, because that is when you test the smallest valid version of the current window.

That is the whole pattern in LC 209.

***

## How to Recognize the Sliding Window Pattern

Sliding window is one of the most important interview patterns, but it only works when the problem has the right structure. The key is to recognize **when a contiguous segment can be adjusted incrementally instead of recomputed from scratch**.

### The core idea

Sliding window is usually the right pattern when:

- The problem talks about a **subarray** or **substring**
- The segment must be **contiguous**
- You are asked to find a **minimum**, **maximum**, **longest**, **shortest**, or **count**
- The condition can be checked and updated as the window grows or shrinks

In other words, if you can maintain information about a window `[left, right]` and update it in O(1) when one element enters or leaves, sliding window is a strong candidate.

### Signals that usually mean Sliding Window

Look for these clues in the statement:

- “contiguous subarray”
- “substring”
- “window”
- “at most”
- “at least”
- “exactly k”
- “smallest / shortest”
- “longest / maximum length”
- “distinct / unique”
- “sum ≥ target”

These are all common signals that the answer may come from moving two pointers instead of using nested loops. [web:275][web:245]

### The two main types of Sliding Window

#### 1. Fixed-size window

Use this when the window length is fixed, for example:

- subarray of size `k`
- substring of length `k`

Typical logic:

- Expand right
- If window size exceeds `k`, move left
- Check whether the current window is valid

Examples:
- LC 2461
- LC 2841 [web:275]

#### 2. Variable-size window

Use this when the window size is not fixed and must adapt to satisfy a condition.

Typical logic:

- Expand right until the window becomes valid
- Shrink left while the window stays valid
- Track the best answer during this process

Examples:
- LC 209
- LC 3 [web:245][web:276]

### Why LC 209 is a classic Sliding Window problem

LC 209 is a textbook **variable-size sliding window** problem because:

- the subarray must be **contiguous**
- you want the **minimum length**
- the condition is **sum ≥ target**
- all numbers are **positive** [web:236][web:276]

That last point is crucial. Because all numbers are positive:

- moving `right` forward always increases the sum
- moving `left` forward always decreases the sum

This monotonic behavior is what makes the sliding window work so cleanly. [web:245][web:248]

### When Sliding Window does NOT work

Sliding window is not automatic for every subarray problem.

Be careful when:

- the array contains **negative numbers**
- the condition depends on something you cannot update efficiently
- the segment is not contiguous
- you need to consider many unrelated past states [web:275]

For example, in LC 209, sliding window works because the sum behaves predictably with positive numbers. If negative numbers were allowed, shrinking the window could increase the sum, and the standard pattern would break. [web:236]

### The interview mental model

When you see a subarray or substring problem, ask yourself:

1. Is the segment **contiguous**?
2. Am I looking for a **best window** (min, max, longest, shortest)?
3. Can I maintain the condition while moving two pointers?
4. Can I update the state in O(1) when one element enters or leaves? [web:275]

If the answer is yes, think **Sliding Window** before thinking brute force.