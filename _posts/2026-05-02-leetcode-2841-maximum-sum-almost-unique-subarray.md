---
title: "2841 · Maximum Sum of Almost Unique Subarray"
date: 2026-05-02 18:00:00 +0000
categories: [Algorithms, LeetCode]
tags: [sliding-window, array, java, interview, leetcode, hashmap, list]
description: A complete breakdown of LeetCode 2841 — fixed-size sliding window with a frequency HashMap, the almost-unique validity condition, Java List vs Array pitfalls, and the full solution.
---

## What Is This Problem Really About?

You are given a `List<Integer>` and two integers `m` and `k`. You want to
find the **maximum sum** among all subarrays of **exactly length `k`** that
contain **at least `m` distinct elements** — the "almost unique" condition.

This is a direct evolution of LC 2461 (all elements distinct), with one
relaxed condition: instead of requiring all `k` elements to be unique, you
now only require at least `m` of them to be distinct. The pattern and
tooling are identical — fixed-size sliding window + frequency HashMap.

---

## Problem Classification

| Property | Detail |
|---|---|
| **Pattern** | Fixed-size Sliding Window + Frequency HashMap |
| **Core goal** | Maximum sum subarray of length `k` with ≥ `m` distinct elements |
| **Input** | `List<Integer>` (not an array!), plus integers `m` and `k` |
| **Valid window condition** | `window size == k` AND `map.size() >= m` |
| **Expected complexity** | O(n) time · O(k) space |

---

## Java List vs Array — What Changes in Your Code

The function signature takes a `List<Integer>`, not an `int[]`. This is
common in LeetCode problems and in real codebases — you need to know
exactly how to work with it.

### The Three Practical Differences

| Operation | Array (`int[]`) | List (`List<Integer>`) |
|---|---|---|
| **Access element** | `nums[j]` | `nums.get(j)` |
| **Get length/size** | `nums.length` | `nums.size()` |
| **Type of elements** | primitive `int` | boxed `Integer` |

> ⚠️ **No square brackets on a List.** `nums[j]` will not compile — you
> must use `nums.get(j)`. This is the most common mistake when switching
> between array and List problems.

### Avoid Repeated `.get()` Calls

Since `nums.get(j)` is a method call, calling it multiple times on the
same index is wasteful and clutters the code. Always extract to a local
variable:

```java
// ❌ Verbose and calls .get() three times
map.put(nums.get(j), map.getOrDefault(nums.get(j), 0) + 1);
currentSum += nums.get(j);

// ✅ Clean — one call, reuse the variable
int right = nums.get(j);
map.put(right, map.getOrDefault(right, 0) + 1);
currentSum += right;
```

Same applies to the left element when shrinking:

```java
int left = nums.get(i);
map.put(left, map.get(left) - 1);
if (map.get(left) == 0) map.remove(left);
currentSum -= left;
i++;
```

---

## The "Almost Unique" Condition

This is the only real difference from LC 2461. In LC 2461 the validity
condition was:

```java
map.size() == k   // ALL k elements must be distinct
```

Here it is:

```java
map.size() >= m   // AT LEAST m elements must be distinct
```

The HashMap's `size()` always equals the number of distinct elements
currently in the window — because you remove a key entirely when its count
reaches `0`. So `map.size() >= m` is an O(1) check that tells you
instantly whether the window is valid.

---

## The Frequency HashMap — Still the Core Tool

The frequency HashMap is necessary for the same reason as in LC 2461:
the window slides forward and removes elements from the left. A HashSet
would incorrectly remove a key even when duplicates of that value are still
inside the window.

```java
// Safe removal: only evict the key when its count truly hits 0
map.put(left, map.get(left) - 1);
if (map.get(left) == 0) map.remove(left);
```

> **The counting rule:** any time you need to know *how many times* an
> element appears — in a window, a subarray, or anywhere — you need a
> `HashMap<Element, Integer>`. A HashSet only answers "is it there?",
> not "how many times?".

---

## Step-by-step Trace

```
nums = [2, 6, 7, 3, 1, 7],  m = 3,  k = 4

j=0: right=2 → map={2:1}, sum=2.   size=1 < k, skip.
j=1: right=6 → map={2:1,6:1}, sum=8.   size=2 < k, skip.
j=2: right=7 → map={2:1,6:1,7:1}, sum=15.   size=3 < k, skip.
j=3: right=3 → map={2:1,6:1,7:1,3:1}, sum=18.   size=4==k.
     map.size=4 >= m=3 → valid! maxSum=18 ✅
j=4: right=1 → map={2:1,6:1,7:1,3:1,1:1}, sum=19.   size=5>k.
     left=2 → map={6:1,7:1,3:1,1:1}, sum=17.   size=4==k.
     map.size=4 >= m=3 → valid! maxSum=18 (17 < 18)
j=5: right=7 → map={6:1,7:2,3:1,1:1}, sum=24.   size=5>k.
     left=6 → map={7:2,3:1,1:1}, sum=18.   size=4==k.
     map.size=3 >= m=3 → valid! maxSum=18 (tie)

Answer: 18  →  subarray [2, 6, 7, 3]  ✅
```

Notice at `j=5`: the window `[7, 3, 1, 7]` has `map={7:2, 3:1, 1:1}`.
`map.size() = 3 >= m=3` — valid, even though `7` appears twice. This is
exactly the "almost unique" relaxation in action.

---

## Java Solution (O(n) · O(k))

```java
class Solution {
    public long maxSum(List<Integer> nums, int m, int k) {
        Map<Integer, Integer> map = new HashMap<>();
        long currentSum = 0, maxSum = 0;
        int i = 0;

        for (int j = 0; j < nums.size(); j++) {       // .size() not .length
            int right = nums.get(j);                   // .get(j) not [j]
            map.put(right, map.getOrDefault(right, 0) + 1);
            currentSum += right;

            if (j - i + 1 > k) {
                int left = nums.get(i);
                map.put(left, map.get(left) - 1);
                if (map.get(left) == 0) map.remove(left);
                currentSum -= left;
                i++;
            }

            if (j - i + 1 == k && map.size() >= m) {  // ≥ m, not == k
                maxSum = Math.max(maxSum, currentSum);
            }
        }

        return maxSum;
    }
}
```

---

## LC 2461 vs LC 2841 — Side by Side

These two problems share the exact same skeleton. The only two differences
are the input type and the validity condition:

| | LC 2461 | LC 2841 |
|---|---|---|
| **Input type** | `int[]` | `List<Integer>` |
| **Access** | `nums[j]` | `nums.get(j)` |
| **Size** | `nums.length` | `nums.size()` |
| **Valid window** | `map.size() == k` (all distinct) | `map.size() >= m` (at least m distinct) |
| **Return type** | `long` | `long` |

Everything else — expansion, shrinking, frequency map management, sum
tracking — is identical.

---

## Common Pitfalls

**Pitfall 1 — Using `nums[j]` on a List.**
`List<Integer>` does not support bracket access. Always use `nums.get(j)`.
This will cause a compile error, so it is caught early — but it's easy to
forget when you are used to arrays.

**Pitfall 2 — Using `nums.length` on a List.**
`length` is a property of arrays. Lists use `nums.size()` — a method call
with parentheses. Mixing them causes a compile error.

**Pitfall 3 — Using `==` instead of `>=` for the validity check.**
LC 2461 uses `map.size() == k`. Here it must be `map.size() >= m`. Using
`==` would reject valid windows where the count of distinct elements
exceeds `m`.

**Pitfall 4 — Calling `.get()` multiple times on the same index.**
Extract `nums.get(j)` and `nums.get(i)` to local variables `right` and
`left`. It is cleaner, avoids redundant method calls, and makes the intent
of the code immediately obvious.

---

## The Full Pattern Family

| Problem | Window | Validity | Input type |
|---|---|---|---|
| LC 53 – Maximum Subarray | Variable | `currentSum > 0` | `int[]` |
| LC 2461 – Max Sum Distinct (Length K) | Fixed | `map.size() == k` | `int[]` |
| LC 2841 – Max Sum Almost Unique | Fixed | `map.size() >= m` | `List<Integer>` |
| LC 3 – Longest Substring No Repeat | Variable | no repeating chars | `String` |

Moving from LC 2461 to LC 2841 teaches two things at once: how to adapt
the same pattern to a `List` input, and how a single character change in
the validity condition (`==` → `>=`) can meaningfully relax the problem
constraints. Both are small shifts, but recognizing them quickly is exactly
what separates a good interview answer from a great one.
