---
title: "2461 · Maximum Sum of Distinct Subarrays With Length K"
date: 2026-05-02 17:00:00 +0000
categories: [Algorithms, LeetCode]
tags: [sliding-window, array, java, interview, leetcode, hashmap]
description: A complete breakdown of LeetCode 2461 — why a fixed-size sliding window with a frequency HashMap is the right tool, how to track valid windows, common pitfalls, and the full Java solution.
---

## What Is This Problem Really About?

You have an array of integers and a number `k`. You want to find the
**maximum sum** among all subarrays of **exactly length `k`** that contain
**only distinct elements** — no duplicates allowed inside the window.

The core skill being tested is the **fixed-size sliding window** pattern:
rather than recomputing each window from scratch (O(n·k)), you slide a
window of exactly `k` elements across the array, maintaining a running sum
and a frequency map incrementally in O(1) per step.

---

## Problem Classification

| Property | Detail |
|---|---|
| **Pattern** | Fixed-size Sliding Window + Frequency HashMap |
| **Core goal** | Maximum sum subarray of length `k` with all distinct elements |
| **Input** | Integer array with possible duplicates, fixed window size `k` |
| **Expected complexity** | O(n) time · O(k) space |

---

## Why Not Kadane's Algorithm?

If you have solved LC 53 (Maximum Subarray), your first instinct might be
to reach for Kadane's Algorithm. The uniqueness constraint kills that idea
immediately.

Kadane's carries exactly one number forward — `currentSum`. That number
summarizes everything about the past subarray. But the moment you need to
know *"is this element already inside my window?"*, one number is not
enough. You need to remember the **contents** of the window, not just
their sum.

> **The general rule:** if the validity of the window depends on the
> *contents* (distinct elements, frequencies, counts), use a
> **Sliding Window + HashMap**. If it depends only on a numerical
> aggregate (sum positive or negative), use **Kadane's / Greedy**.

---

## The Golden Rule: Counting Elements → Always a HashMap

This is one of the most important data structure rules to internalize for
interviews:

> **Any time you need to count how many times each element appears —
> inside a window, a subarray, or anywhere in a sequence — you need a
> `HashMap<Element, Count>`.**

A HashMap maps each value to its frequency. That is the only structure
that can answer "how many times does X appear?" in O(1).

Here is the idiomatic Java pattern for frequency counting:

```java
// Increment count (or initialize to 1 if first occurrence)
map.put(value, map.getOrDefault(value, 0) + 1);

// Decrement count — only remove the key when count hits 0
map.put(value, map.get(value) - 1);
if (map.get(value) == 0) map.remove(value);
```

The `getOrDefault(key, 0)` call is the key detail: it safely returns `0`
instead of `null` when the key is not yet in the map, avoiding a
`NullPointerException` on the `+ 1`.

### HashMap vs HashSet vs Array — When to Use Each

| Need | Tool | Why |
|---|---|---|
| Does X exist? | `HashSet` | O(1) membership check |
| How many times does X appear? | `HashMap<X, Integer>` | O(1) frequency count |
| How many times does X appear? (values bounded) | `int[]` array | O(1), faster in practice |
| All of the above + iteration order | `LinkedHashMap` | preserves insertion order |

Any time the word **"count"**, **"frequency"**, **"distinct"**, or
**"occurrences"** appears in a problem, that is your cue to reach for a
HashMap immediately.

---

## Why a HashMap and Not a HashSet Here?

A HashSet tracks *whether* an element is present. That is enough when you
expand the window — but it breaks the moment you shrink it.

Consider this window sliding one step to the right:

```
Before: [1, 2, 1]   k=3
Remove left element: 1
HashSet.remove(1) → {2}   ❌ WRONG — 1 still appears at index 2!
```

A HashSet does not know how many times a value appears in the window.
Removing it blindly destroys information about the remaining duplicate.

A **frequency HashMap** handles this correctly:

```
map = {1:2, 2:1}
Decrement count of 1 → map = {1:1, 2:1}   ✅ 1 is still tracked
Only remove the key entirely when the count reaches 0
```

**The rule:** whenever a sliding window can contain duplicate values,
a HashSet is not safe for removal — you need a frequency HashMap.

---

## The Fixed-Size Window Mechanic

This problem has a fixed window size `k`, which makes the sliding mechanic
simpler than a variable-size window:

1. **Always expand right** — add `nums[j]` unconditionally
2. **Always shrink left when oversized** — once `j - i + 1 > k`, remove `nums[i]` and advance `i`
3. **Check validity** — when `j - i + 1 == k` and `map.size() == k`, the window is valid (all distinct)

The left pointer does not move because of duplicates — it moves
**mechanically**, every time the window exceeds size `k`. The HashMap
separately tells you whether the current window of exactly `k` elements
is valid or not.

---

## Step-by-step Trace

```
nums = [1, 5, 4, 2, 9, 9, 9],  k = 3

j=0: add 1 → map={1:1}, sum=1.  size=1 < k, skip.
j=1: add 5 → map={1:1,5:1}, sum=6.  size=2 < k, skip.
j=2: add 4 → map={1:1,5:1,4:1}, sum=10.  size=3==k, map.size=3==k → valid! maxSum=10
j=3: add 2 → map={1:1,5:1,4:1,2:1}, sum=12.  size=4>k → remove 1:
       map={5:1,4:1,2:1}, sum=11.  size=3==k, map.size=3==k → valid! maxSum=11
j=4: add 9 → map={5:1,4:1,2:1,9:1}, sum=20.  size=4>k → remove 5:
       map={4:1,2:1,9:1}, sum=15.  size=3==k, map.size=3==k → valid! maxSum=15
j=5: add 9 → map={4:1,2:1,9:2}, sum=24.  size=4>k → remove 4:
       map={2:1,9:2}, sum=20.  size=3==k, map.size=2≠k → invalid (9 appears twice)
j=6: add 9 → map={2:1,9:3}, sum=29.  size=4>k → remove 2:
       map={9:3}, sum=27.  size=3==k, map.size=1≠k → invalid

Answer: 15  →  subarray [4, 2, 9]  ✅
```

Notice at `j=5`: the map correctly shows `{2:1, 9:2}` — the frequency of
`9` is `2`, which means the window has a duplicate. `map.size() == 2 ≠ k`
catches this immediately. A HashSet would have silently failed here.

---

## Java Solution (O(n) · O(k))

```java
class Solution {
    public long maximumSubarraySum(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<>();
        long currentSum = 0;
        long maxSum = 0;
        int i = 0;

        for (int j = 0; j < nums.length; j++) {
            // Expand: add nums[j] to the window and increment its count
            map.put(nums[j], map.getOrDefault(nums[j], 0) + 1);
            currentSum += nums[j];

            // Shrink: remove leftmost element if window exceeds k
            if (j - i + 1 > k) {
                map.put(nums[i], map.get(nums[i]) - 1);
                if (map.get(nums[i]) == 0) map.remove(nums[i]); // only remove when count=0
                currentSum -= nums[i];
                i++;
            }

            // Valid window: exactly k elements, all distinct (map.size == k)
            if (j - i + 1 == k && map.size() == k) {
                maxSum = Math.max(maxSum, currentSum);
            }
        }

        return maxSum;
    }
}
```

---

## The Three State Variables

| Variable | Type | Meaning |
|---|---|---|
| `map` | `HashMap<Integer, Integer>` | Frequency count of each element in the window |
| `currentSum` | `long` | Running sum of elements in the current window |
| `maxSum` | `long` | Best valid window sum seen so far |

The HashMap is doing two jobs at once: it counts frequencies (enabling
safe removal) and its `size()` tells you how many distinct elements are
currently in the window. That single `map.size() == k` check replaces
what would otherwise be a costly distinctness scan.

---

## Common Pitfalls

**Pitfall 1 — Using a HashSet instead of a HashMap.**
A HashSet cannot safely handle removal when duplicates exist inside the
window. As soon as you need to *count* elements, reach for a HashMap.

**Pitfall 2 — Moving the left pointer on duplicates.**
In a fixed-size window, `i` always moves when the window exceeds `k` —
not when a duplicate is found. The HashMap tells you *if* the window is
valid, but the window always stays exactly size `k`.

**Pitfall 3 — Forgetting to track `currentSum`.**
The HashMap manages uniqueness. The sum must be tracked separately with
`currentSum += nums[j]` on expansion and `currentSum -= nums[i]` on
shrink.

**Pitfall 4 — Returning `int` instead of `long`.**
The problem constraints allow sums that overflow `int`. Always use `long`
for sum variables.

---

## How This Fits the Pattern Family

| Problem | Window Type | Validity Condition | Tool |
|---|---|---|---|
| LC 53 – Maximum Subarray | Variable | Sum must be positive | Kadane's |
| LC 121 – Stock Buy & Sell | N/A (two pointers) | Buy before sell | Greedy running min |
| LC 2461 – This problem | **Fixed (size k)** | All elements distinct | Sliding Window + HashMap |
| LC 3 – Longest Substring No Repeat | Variable | No repeating chars | Sliding Window + HashSet |
| LC 992 – Subarrays K Different Integers | Variable | Exactly k distinct | Sliding Window (advanced) |

The jump from LC 53 to LC 2461 introduces two new ideas at once: a
**fixed window size** that slides mechanically, and a **frequency HashMap**
that tracks window contents rather than a numerical aggregate. Master
these two ideas together and a large family of interview problems opens up.
